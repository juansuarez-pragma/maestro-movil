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

### Problema detectado (técnico)
- Ignorar 429 y reintentar genera thundering herd, prolonga el bloqueo y empeora UX.
- Sin leer `Retry-After` ni pausar colas, se consumen cuotas rápidamente.
- Falta de degradación clara confunde al usuario y dispara tickets.

### Escenario de Negocio

> *"Como usuario, si la API me limita, quiero ver una degradación clara y no errores inesperados."*

### Incidentes reportados
- **APIs públicas/financieras:** Códigos 429 frecuentes; manejo adecuado reduce churn.
- **SRE practices:** Evitar thundering herd ante rate limits.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| APIs públicas/fintech | Global | 429 comunes; manejo deficiente prolonga bloqueo. |
| SRE practices | Global | Backoff y respeto a `Retry-After` reducen thundering herd. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; manejo de límites/errores es hallazgo común. |

**Resumen global**
- Rate limiting sin manejo degrade UX y alarga bloqueos; awareness (`Retry-After`, pausas, degradación) es clave.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Respeto de `Retry-After` y pausa de colas | Equipo móvil, CI |
| Integration (CI) | Degradación/pausa se aplican en 429 simulados | Móvil/QA |
| Observabilidad | Eventos `ratelimit.*` con tiempos de espera y funciones degradadas | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Mensajes claros | Mostrar countdown/tiempo de espera | Reduce frustración |
| Degradación | Desactivar/limitar funciones no críticas durante bloqueo | Protege UX |
| Reintentos | Backoff y pausa de colas | Evita thundering herd |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Defaults | Si no hay `Retry-After`, usar valores seguros | Robustez |
| Críticos | No reintentar operaciones no seguras | Consistencia |
| Monitoreo | Alertar en picos de 429 | Ajustar políticas |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Reintentos ciegos bajo rate limit provocan bloqueo prolongado y mala UX. |
| Opciones evaluadas | Ignorar 429; error genérico; rate-limit aware con pausa/degradación. |
| Decisión | Manejo de 429 con `Retry-After`, pausa de colas, backoff y degradación controlada. |
| Consecuencias | Requiere telemetría y lógica de estado de cuota; UX específica. |
| Riesgos aceptados | Backends sin headers; posibles falsos positivos en entornos de prueba. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Eventos 429 | ↓ vs baseline | Alerta si sube | Menor bloqueo |
| Tiempo en bloqueo | Reducido con pausas/backoff | Alerta si prolongado | UX protegida |
| Reintentos durante límite | Controlados | Crítico si indiscriminados | Evita herd |
| Tickets por límite | ↓ vs baseline | Alerta si no baja | Soporte controlado |

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
