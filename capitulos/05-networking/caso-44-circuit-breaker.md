# Caso 44: Circuit Breaker Móvil
## Proteger la App cuando el Backend Agoniza

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | circuit breaker, resiliencia, fallos backend, degradación |
| **Patrón Técnico** | Circuit Breaker, Fallback/Degrade, Health Check |
| **Stack Seleccionado** | Flutter + Dio interceptors + Riverpod estado de salud + backoff |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, prefiero funciones degradadas antes que ver la app rota cuando el backend falla."*

Sin breaker, los clientes bombardean servicios caídos causando cascadas y mala UX.

### Evidencia de Industria

- **Patrón clásico de resiliencia:** Implementado en apps de alto tráfico.
- **Incidentes SRE:** Breakers evitan thundering herd y reducen MTTR.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Errores masivos visibles, tiempo de espera excesivo |
| **Técnico** | Cascadas de fallos, sobrecarga al backend en recuperación |
| **Reputacional/Económico** | Pérdida de confianza, tickets y churn |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Retries ciegos sin límite | **INADECUADO:** Agrava incidentes. |
| **ACEPTABLE** | Timeouts y retries limitados | **MEJORA:** Menos impacto, pero no corta tráfico a servicios caídos. |
| **ENTERPRISE** | **Circuit breaker cliente:** umbrales de fallo, half-open con probes, fallback/degradación, UI clara | **ÓPTIMO:** Protege backend y da UX controlada. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Abrir breaker tras N fallos/latencias altas. Half-open con probe controlado. Degradar UI (modo sólo lectura, cache) mientras está abierto. Telemetría de estado del breaker. |
| **Restricciones Duras (NO permite)** | **Sin idempotencia:** Fallback debe respetar consistencia. **Mobile variability:** Errores locales de red no siempre implican fallo del backend; requiere heurísticas. **Coordinación:** Sin compartir estado entre clientes, cada uno decide localmente. |
| **Criterio de Selección** | Breaker en interceptor para consistencia; umbrales ajustados; fallback claro; telemetría para observabilidad. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Circuit Breaker | Patrón que abre/cierra flujo de requests según salud. |
| Half-open | Estado de prueba para ver si el servicio se recuperó. |
| Fallback | Comportamiento degradado cuando el servicio está caído. |
| Probe | Request de prueba controlada para cerrar el breaker. |
| Threshold | Umbral de fallos/latencia para abrir el breaker. |

---

## Referencias

- [Martin Fowler - Circuit Breaker](https://martinfowler.com/bliki/CircuitBreaker.html)
- [Polly/AWS SDK Retry & CB Patterns](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/)
