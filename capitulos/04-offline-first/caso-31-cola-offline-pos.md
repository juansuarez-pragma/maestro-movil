# Caso 31: Vender en el Metro
## Cola de Transacciones Offline para POS Móvil

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | POS offline, cola de transacciones, pagos sin red, sincronización diferida |
| **Patrón Técnico** | Offline Queue, Store-and-Forward, Idempotent Operations |
| **Stack Seleccionado** | Flutter + SQLite/Isar para cola + Riverpod para orquestar + retry/backoff |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Cobros offline sin cola persistente se pierden al cerrar app o al reconectar; reintentos manuales generan doble cobro.
- Sin `Idempotency-Key` y confirmación backend, la reconciliación es inconsistente.
- Falta de backoff/TTL → tormenta de reintentos y riesgo de caducar transacciones sin trazabilidad.

### Escenario de Negocio

> *"Como vendedor, necesito cobrar en el metro sin conexión y sincronizar cuando vuelva la red."*

### Incidentes reportados
- **POS móviles:** Store-and-forward es estándar para zonas sin cobertura.
- **Incidentes retail:** Pérdida de ventas por fallos de sincronización offline→online; duplicados por reintentos sin idempotencia.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| POS móviles | Retail/field | Store-and-forward reduce pérdidas en zonas sin cobertura. |
| Incidentes retail | EU/LATAM | Doble cobro y pérdida de ventas por reconexión sin idempotencia. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; offline/sync es área crítica. |

**Resumen global**
- Sin cola persistente + idempotencia, los cobros offline se pierden o se duplican al reconectar.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Ventas perdidas o doble cobro al reintentar |
| **Reputacional** | Clientes sin recibo o con cargos duplicados |
| **Técnico** | Divergencia de inventario y conciliación compleja |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Guardar en memoria y reintentar manual | **INADECUADO:** Se pierden datos al cerrar app. |
| **ACEPTABLE** | Cola local simple con retries | **MEJORA:** Persiste, pero sin idempotencia ni control de orden. |
| **ENTERPRISE** | **Queue persistente + idempotencia:** almacenar payloads con Idempotency-Key, retries con backoff, confirmación del backend, reconciliación post-sync | **ÓPTIMO:** Minimiza pérdidas y duplicados, trazable. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Persistir transacciones en SQLite/Isar con estado (pending/sent/confirmed/failed). Reintentar con backoff y límite. Adjuntar Idempotency-Key para evitar dobles cargos. Reconstruir recibos tras sync. |
| **Restricciones Duras (NO permite)** | **Autorización real offline:** Solo pagos diferidos; requiere aceptación de riesgo. **Conflictos de stock:** Resolver en backend. **TTL de cola:** Transacciones expiran según políticas de adquirente. |
| **Criterio de Selección** | Queue persistente para confiabilidad; idempotencia para seguridad; Riverpod para exponer estado de cola y progreso. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Cola persiste y cambia estados (pending→sent→confirmed/failed) | Equipo móvil, CI |
| Integration (CI) | Reintentos con misma `Idempotency-Key` no duplican cobros; reconexión procesa cola | Móvil/Backend, CI |
| Seguridad/consistencia | TTL/backoff aplicados; expirados se descartan con log | QA/Seguridad |
| Observabilidad | Eventos `queue.tx` con estado, reintentos y resultado | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Estado de cola | UI muestra pendientes/sincronizando/confirmadas | Transparencia |
| Recibos | Generar recibo provisional y actualizar tras sync | Experiencia consistente |
| Reintentos | Backoff y límite; CTA para reintentar manual si falla | Control de riesgo |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| TTL | Expirar transacciones según adquirente; marcar y notificar | Cumplimiento |
| Idempotencia | `Idempotency-Key` por transacción; validar en backend | Evita doble cargo |
| Conciliación | Log por transacción para auditoría y soporte | Trazabilidad |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Cobros offline se pierden o duplican al reconectar. |
| Opciones evaluadas | Memoria + reintento manual; cola simple; cola persistente + idempotencia + backoff. |
| Decisión | Cola persistente con `Idempotency-Key`, backoff, TTL y reconciliación. |
| Consecuencias | Mayor complejidad de cola/estados; requiere soporte backend. |
| Riesgos aceptados | Cobros caducan si TTL vence; riesgo inherente a pagos diferidos. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Cobros perdidos | 0 | Crítico si > 0 | Ingresos protegidos |
| Cobros duplicados | 0 | Crítico si > 0 | Evita reclamos |
| Éxito de sync | > 99% tras reconexión | Alerta si baja | Confiabilidad |
| Tickets por cobro offline | ↓ vs baseline | Alerta si no baja | Menos soporte |
| Latencia de sync | p95 dentro de SLA tras reconexión | Warning si sube | UX clara |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Store-and-Forward | Guardar transacción local y enviarla cuando haya red. |
| Idempotency-Key | Identificador único para evitar doble procesamiento. |
| Backoff | Aumentar tiempo entre reintentos para reducir carga. |
| Reconciliación | Ajustar registros tras sincronizar con backend. |
| TTL | Tiempo de vida de una transacción antes de caducar. |

---

## Referencias

- [Stripe Idempotency](https://stripe.com/docs/idempotency)
- [NIST Offline Transactions Guidance](https://csrc.nist.gov/)
- [SQLite Best Practices](https://www.sqlite.org/cli.html)
