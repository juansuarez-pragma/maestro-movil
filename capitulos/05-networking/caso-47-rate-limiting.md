# Caso 47: Rate Limiting Graceful
## Degradar Funcionalidad sin Crashear

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | rate limit, degradación, resiliencia, throttling |
| **Patrón Técnico** | Client-side Rate Limiting, Graceful Degradation, Quota Awareness |
| **Stack Seleccionado** | Flutter + Dio interceptors + Riverpod para estado de cuota + backoff |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, si la API me limita, quiero ver una degradación clara y no errores inesperados."*

Al alcanzar límites, muchos clientes siguen reintentando y empeoran el bloqueo; sin manejo, la UX se rompe.

### Evidencia de Industria

- **APIs públicas/financieras:** Códigos 429 frecuentes; manejo adecuado reduce churn.
- **SRE practices:** Evitar thundering herd ante rate limits.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Errores repetidos, frustración |
| **Técnico** | Bloqueo prolongado por reintentos; consumo de cuota |
| **Reputacional/Económico** | Funciones críticas inutilizables bajo límite |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Ignorar 429 y reintentar normal | **INADECUADO:** Extiende bloqueo, mala UX. |
| **ACEPTABLE** | Mostrar error genérico | **MEJORA:** Evita algunos reintentos, pero no guía al usuario ni respeta Retry-After. |
| **ENTERPRISE** | **Rate limit aware:** leer Retry-After, pausar solicitudes, degradar UI, mostrar tiempos de espera, backoff | **ÓPTIMO:** Reduce bloqueo y mantiene UX controlada. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Leer headers de cuota/Retry-After. Pausar colas de requests. Mostrar countdown en UI. Degradar funciones no críticas. Telemetría de 429 y tiempos de espera. |
| **Restricciones Duras (NO permite)** | **Sin headers:** Algunos backends no envían Retry-After; requiere defaults. **Límites globales:** Cliente no controla cuota de otros clientes. **Operaciones críticas:** Algunas no deben reintentarse automáticamente. |
| **Criterio de Selección** | Interceptor para manejar 429; Riverpod para estado de cuota/pausas; UX clara para espera/degradación. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Rate limit | Límite de solicitudes permitido en un intervalo. |
| Retry-After | Header que indica cuándo reintentar. |
| Degradación | Funcionar con capacidad reducida en lugar de fallar. |
| Quota | Cupo de solicitudes asignado. |
| Backoff | Espera creciente antes de reintentar. |

---

## Referencias

- [HTTP 429 Spec](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429)
- [AWS/Azure Rate Limit Guidance](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design#throttling)
