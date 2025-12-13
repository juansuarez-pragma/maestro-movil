# Caso 12: Race Condition en el [Checkout](#term-checkout "Proceso de cobro de una orden; crítico en e-commerce/fintech.")
## Cuando Dos Hilos Debitan la Misma Cuenta

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | checkout, condición de carrera, doble débito, consistencia transaccional |
| **Patrón Técnico** | [Optimistic Concurrency Control](#term-optimistic-concurrency-control "Validar versiones antes de aplicar cambios; si la versión cambia, se rechaza o reintenta."), Idempotent Operations, [Transactional Outbox](#term-transactional-outbox "Patrón para asegurar que eventos y cambios de DB se publican de forma atómica.") |
| **Stack Seleccionado** | Flutter + Riverpod/StateNotifier + HTTP idempotente ([Idempotency-Key](#term-idempotency-key "Identificador único de operación que garantiza que múltiples requests producen un solo efecto.")) + SQLite cache |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Sin idempotencia end-to-end, taps rápidos o múltiples pestañas generan requests simultáneas que terminan en doble débito.
- Throttle en UI no evita reintentos automáticos ni race en backend; sin `Idempotency-Key` y OCC, se duplican cargos.
- Falta de correlación impide reconciliar respuestas tardías y auditar incidentes.

### Escenario de Negocio

> *"Como usuario, al pagar desde dos pestañas, mi cuenta no debe ser debitada dos veces."*

### Incidentes reportados
- **Fintech LATAM 2022:** 0.4% de checkouts con cobros duplicados por reintentos simultáneos.
- **Stripe/Adyen:** Guías oficiales exigen `Idempotency-Key` para pagos críticos.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Fintech LATAM 2022 | Checkouts móviles | 0.4% con cobros duplicados por reintentos simultáneos. |
| PSPs (Stripe/Adyen) | Global | Idempotencia es requisito para pagos; duplicados generan disputas. |
| ACFE Fraud 2022 | Global | Cobros indebidos/ATO aumentan costos de reverso y soporte. |

**Resumen global**
- Los duplicados erosionan confianza y disparan chargebacks.
- La idempotencia es práctica estándar para eliminar doble cargo y soportar reintentos seguros.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Reversiones costosas, soporte saturado, pérdida de confianza |
| **Regulatorio** | Reclamos ante defensoría/financiera, multas por cobros indebidos |
| **Técnico** | Datos divergentes entre cliente y backend, auditoría compleja |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Deshabilitar botón tras primer tap | **INADECUADO:** No protege contra múltiples hilos/requests concurrentes ni reintentos automáticos. |
| **ACEPTABLE** | Throttle/Debounce en UI + retries con backoff | **MEJORA UX:** Reduce taps múltiples, pero sin garantía de idempotencia en backend. |
| **ENTERPRISE** | **Idempotency end-to-end:** Idempotency-Key única por checkout, OCC en backend, Transactional Outbox para consistencia; cliente reintenta seguro | **ÓPTIMO:** Evita doble débito, permite reintentos seguros ante fallos de red, trazable. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Generar `Idempotency-Key` por orden y reusar en reintentos. Mostrar estado `processing` mientras se resuelve el checkout. Persistir estado de orden en SQLite para reanudar. Reconciliar respuesta tardía sin duplicar cargos. |
| **Restricciones Duras (NO permite)** | **Sin apoyo backend:** Idempotencia solo en cliente no evita doble cargo. **Clock skew:** OCC backend debe manejar versionado. **Offline prolongado:** Reintentos diferidos pueden expirar la llave/orden. |
| **Criterio de Selección** | Riverpod/StateNotifier para estado de checkout aislado y testeable; `Idempotency-Key` siguiendo guías Stripe/Adyen; Transactional Outbox en backend para consistencia pago/orden. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Estado de checkout transita pending/processing/success sin carreras | Equipo móvil, CI |
| Integration (CI) | Reintentos con misma `Idempotency-Key` generan un solo cargo | Móvil/Backend, CI + staging |
| Seguridad/consistencia | Respuestas tardías/duplicadas se reconcilian sin duplicar cargos | QA/Seguridad |
| Observabilidad | Evento `checkout.*` con `idempotency_key`, `trace_id`; métrica de duplicados | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Estado visible | Mostrar `processing`; botón deshabilitado, pero reintentos controlados | Reduce tap spam |
| Reintentos | Retry con backoff + misma `Idempotency-Key` | Resiliencia sin duplicar |
| Errores | Mensajes claros; opción de reintentar con misma key | Evita cargo duplicado |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Idempotency-Key | Generar por orden y persistir para reintentos | Clave anticobro duplicado |
| OCC backend | Versionado/locks suaves en backend | Evita colisión de updates |
| Outbox | Transactional Outbox para eventos de pago/orden | Consistencia eventual segura |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Cobros duplicados por requests simultáneos/reintentos. |
| Opciones evaluadas | Solo UI debounce; locking pesimista; idempotencia + OCC + outbox. |
| Decisión | Idempotencia end-to-end + OCC + outbox; Riverpod para estado del checkout. |
| Consecuencias | Requiere soporte de backend y PSP; manejo de keys y retries. |
| Riesgos aceptados | Expiración de key en offline prolongado; dependencia de PSP en idempotencia. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Cobros duplicados | 0 incidentes | Crítico si > 0 | Confianza y menos chargebacks |
| Reintentos seguros | 100% con misma `Idempotency-Key` | Alerta si falla | Resiliencia sin duplicar |
| Tiempo confirmación checkout | p95 < 3 s | Warning si se acerca | UX fluida |
| Tickets por doble cargo | ↓ ≥ 80% | Alerta si no baja | Soporte controlado |
| Éxito OCC/outbox | > 99% | Alerta si baja | Consistencia orden/pago |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-idempotency-key"></a>Idempotency-Key | Identificador único de operación que garantiza que múltiples requests producen un solo efecto. |
| <a id="term-optimistic-concurrency-control"></a>Optimistic Concurrency Control | Validar versiones antes de aplicar cambios; si la versión cambia, se rechaza o reintenta. |
| <a id="term-transactional-outbox"></a>Transactional Outbox | Patrón para asegurar que eventos y cambios de DB se publican de forma atómica. |
| <a id="term-backoff-exponencial"></a>Backoff exponencial | Estrategia de reintentos con tiempos crecientes para reducir carga y colisiones. |
| <a id="term-checkout"></a>Checkout | Proceso de cobro de una orden; crítico en e-commerce/fintech. |
| <a id="term-idempotency-key"></a>Idempotency-Key | Identificador único de operación para garantizar un solo efecto. |

---

## Referencias

- [Stripe Idempotent Requests](https://stripe.com/docs/idempotency)
- [Adyen Idempotency](https://docs.adyen.com/development-resources/api-idempotency/)
- [Fowler - Transactional Outbox](https://martinfowler.com/articles/patterns-of-distributed-systems/transactional-outbox.html)
- [Baymard Institute - Cart Abandonment](https://baymard.com/lists/cart-abandonment-rate)
- [ACFE Fraud 2022](https://acfepublic.s3-us-west-2.amazonaws.com/2022-Report-to-the-Nations.pdf)
