# Caso 12: Race Condition en el Checkout
## Cuando Dos Hilos Debitan la Misma Cuenta

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | checkout, condición de carrera, doble débito, consistencia transaccional |
| **Patrón Técnico** | Optimistic Concurrency Control, Idempotent Operations, Transactional Outbox |
| **Stack Seleccionado** | Flutter + Riverpod/StateNotifier + HTTP idempotente (Idempotency-Key) + SQLite cache |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, al pagar desde dos pestañas, mi cuenta no debe ser debitada dos veces."*

En dispositivos rápidos o con taps repetidos, se envían múltiples requests simultáneas al backend, generando doble débito si el servidor no aplica control de concurrencia.

### Evidencia de Industria

- **Caso Fintech LATAM 2022:** 0.4% de checkouts generaron cobros duplicados por reintentos simultáneos.
- **Stripe/Adyen Guidelines:** Recomiendan Idempotency-Key en operaciones financieras críticas.

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
| **Restricciones Duras (NO permite)** | **Sin apoyo backend:** Idempotencia solo en cliente no evita doble cargo. **Clock skew:** No depende del tiempo, pero OCC backend debe manejar versionado. **Offline prolongado:** Reintentos diferidos pueden expirar la llave o el checkout. |
| **Criterio de Selección** | Riverpod/StateNotifier para estado de checkout aislado y testeable; Idempotency-Key siguiendo guías Stripe/Adyen; Transactional Outbox recomendado en backend para evitar inconsistencia entre pago y actualización de orden. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Idempotency-Key | Identificador único de operación que garantiza que múltiples requests producen un solo efecto. |
| Optimistic Concurrency Control | Validar versiones antes de aplicar cambios; si la versión cambia, se rechaza o reintenta. |
| Transactional Outbox | Patrón para asegurar que eventos y cambios de DB se publican de forma atómica. |
| Backoff exponencial | Estrategia de reintentos con tiempos crecientes para reducir carga y colisiones. |
| Checkout | Proceso de cobro de una orden; crítico en e-commerce/fintech. |

---

## Referencias

- [Stripe Idempotent Requests](https://stripe.com/docs/idempotency)
- [Adyen Idempotency](https://docs.adyen.com/development-resources/api-idempotency/)
- [Fowler - Transactional Outbox](https://martinfowler.com/articles/patterns-of-distributed-systems/transactional-outbox.html)
