# Caso 43: El Timeout que Quebró el Banco
## Configurar Retry Policies Inteligentes

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | retry, timeout, resiliencia, políticas de reintento |
| **Patrón Técnico** | Exponential Backoff, Jitter, Circuit Breaker Light |
| **Stack Seleccionado** | Flutter + Dio interceptors + Riverpod para estado de red + backoff con jitter |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, no quiero ver fallos al pagar por timeouts intermitentes; el sistema debe reintentar con inteligencia."*

Retries mal configurados causan tormentas de tráfico o dobles cargos; sin ellos, la UX sufre por fallos transitorios.

### Evidencia de Industria

- **SRE practices:** Backoff con jitter evita sincronización de thundering herd.
- **Incidente bancario 2020:** Retries sin idempotencia generaron cargos duplicados.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Doble cargo o caída de servicios por tormenta de retries |
| **UX** | Errores visibles en fallos transitorios |
| **Técnico** | Saturación de backend, timeouts en cascada |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Retries infinitos o sin backoff | **INADECUADO:** Thundering herd, más fallos. |
| **ACEPTABLE** | Retries fijos (pocos) | **MEJORA:** Menos riesgo, pero sin adaptación a latencia/errores. |
| **ENTERPRISE** | **Backoff exponencial con jitter + límites:** retries limitados, timeouts adecuados por operación, idempotencia garantizada, cancelación si sin red | **ÓPTIMO:** Resiliencia sin tormentas ni duplicados. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Configurar timeouts por tipo de operación. Retries con backoff y jitter. Cancelar si no hay red. Idempotency-Key para operaciones financieras. Telemetría de retries y motivos. |
| **Restricciones Duras (NO permite)** | **Sin idempotencia backend:** Riesgo de duplicados. **Operaciones no reintentar:** Algunas no son seguras (mutaciones sin idempotencia). **Red caída prolongada:** Retries deben detenerse. |
| **Criterio de Selección** | Interceptor en Dio para política uniforme; Riverpod para exponer estado de red; límites estrictos de intentos; idempotencia en operaciones críticas. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Backoff exponencial | Incrementar el intervalo entre reintentos. |
| Jitter | Aleatorizar intervalos para evitar sincronía. |
| Idempotencia | Misma operación repetida no cambia el resultado final. |
| Thundering herd | Multiples clientes reintentando a la vez causando pico de carga. |
| Timeout | Tiempo límite para abortar una operación. |

---

## Referencias

- [Google SRE Book - Handling Overload](https://sre.google/sre-book/handling-overload/)
- [AWS Architecture Blog - Exponential Backoff](https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/)
