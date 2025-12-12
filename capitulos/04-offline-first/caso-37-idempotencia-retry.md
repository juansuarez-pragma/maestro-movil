# Caso 37: El Pedido Duplicado
## Idempotencia en Retry de Operaciones Críticas

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | idempotencia, retries, pedidos duplicados, operaciones críticas |
| **Patrón Técnico** | Idempotent Operations, Retry with Backoff, Idempotency-Key |
| **Stack Seleccionado** | Flutter + Dio interceptors + Idempotency-Key + Riverpod estado de pedidos |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, si reintento una compra, no quiero que se genere doble pedido o cobro."*

Retries por fallas de red pueden duplicar operaciones si no son idempotentes.

### Evidencia de Industria

- **PSP/fintech:** Idempotency-Key es estándar para pagos/órdenes.
- **Incidentes e-commerce:** Doble pedido por reintentos simultáneos o timeouts.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Doble cobro/pedido, reembolsos costosos |
| **Reputacional** | Pérdida de confianza del cliente |
| **Técnico** | Inconsistencia entre cliente y backend |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Retries sin control ni idempotencia | **INADECUADO:** Riesgo alto de duplicados. |
| **ACEPTABLE** | Limitar retries y deshabilitar UI | **MEJORA:** Menos duplicados, pero no garantiza idempotencia. |
| **ENTERPRISE** | **Idempotency end-to-end:** clave única por operación, retries seguros, validación en backend, UI coherente | **ÓPTIMO:** Evita duplicados y mantiene consistencia. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Generar Idempotency-Key por operación y persistirla en cola de acciones. Reusar la clave en reintentos. UI refleja estado pendiente/confirmado. Limpiar clave al concluir. Telemetría de intentos. |
| **Restricciones Duras (NO permite)** | **Sin soporte backend:** Idempotencia real requiere servidor. **Claves reusadas:** Deben ser únicas; evitar colisiones. **Expiración:** Backend puede expirar claves; manejar errores claros. |
| **Criterio de Selección** | Interceptor para agregar clave; Riverpod para estado de pedidos; backoff con límites; observabilidad de duplicados. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Idempotencia | Repetir una operación produce el mismo resultado sin duplicarla. |
| Idempotency-Key | Identificador único por operación para deduplicar en backend. |
| Backoff | Espera creciente entre reintentos. |
| Pedido crítico | Operación que no debe duplicarse (pagos, órdenes). |
| Retry seguro | Reintento que preserva idempotencia y consistencia. |

---

## Referencias

- [Stripe Idempotency](https://stripe.com/docs/idempotency)
- [AWS Architecture - Backoff](https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/)
