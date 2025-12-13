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

### Problema detectado (técnico)
- Sin breaker, el cliente bombardea servicios caídos → cascadas y peor MTTR.
- Retries ciegos y timeouts altos degradan UX y saturan backend en recuperación.
- Sin telemetría de breaker, no se detectan aperturas/half-open ni su impacto.

### Escenario de Negocio

> *"Como usuario, prefiero funciones degradadas antes que ver la app rota cuando el backend falla."*

### Incidentes reportados
- **Patrón clásico de resiliencia:** Adoptado en apps de alto tráfico.
- **Incidentes SRE:** Breakers evitan thundering herd y reducen MTTR.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| SRE/Resilience patterns | Global | Circuit breakers protegen servicios bajo falla. |
| Apps de alto tráfico | Global | Reducen tráfico durante incidentes y mejoran UX. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; manejo de fallos es crítico. |

**Resumen global**
- Breaker cliente reduce cascadas y ofrece UX degradada controlada durante incidentes.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Breaker abre/cierra según umbrales y half-open | Equipo móvil, CI |
| Integration (CI) | Fallback/degradación se muestra en incidente simulado | Móvil/QA |
| Observabilidad | Eventos `cb.state` y métricas de aperturas/latencia | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Mensajes claros | Mostrar degradación/solo lectura cuando breaker abierto | Confianza |
| Probe controlado | Half-open con pocas requests | Evita carga |
| Cache | Servir cache cuando aplique | UX mejor |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Umbrales | Ajustar por endpoint crítico vs secundario | Precisión |
| Coordinación | Considerar señal de salud desde backend | Coherencia |
| Idempotencia | Fallback no debe violar consistencia | Seguridad |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Tráfico ciego a backend caído causa cascadas y mala UX. |
| Opciones evaluadas | Retries limitados; breaker sin fallback; breaker con fallback/degradación. |
| Decisión | Circuit breaker con umbrales, half-open, fallback y telemetría. |
| Consecuencias | Complejidad de configuración y monitoreo; riesgo de falsos positivos. |
| Riesgos aceptados | Clientes sin coordinación global; heurísticas de red móvil. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Aperturas de breaker | Controladas; half-open exitoso | Alerta si se dispara | Protección backend |
| Latencia/errores en incidentes | Degradación controlada | Alerta si UX rota | Mejor experiencia |
| Tráfico durante falla | Reducido vs baseline | Alerta si no baja | Evita cascada |
| Tickets por caída | ↓ vs baseline | Alerta si no baja | Soporte controlado |

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
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
