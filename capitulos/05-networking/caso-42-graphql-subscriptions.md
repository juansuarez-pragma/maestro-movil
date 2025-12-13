# Caso 42: GraphQL Subscriptions en Trading
## Updates en Tiempo Real de Precios

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | GraphQL, subscriptions, tiempo real, trading, precios |
| **Patrón Técnico** | Real-time Streaming, Backpressure, Delta Updates |
| **Stack Seleccionado** | Flutter + graphql_flutter + WebSocket transport + Riverpod para cache |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Subscriptions sin diffs ni backpressure saturan red/CPU; UI recibe demasiados eventos y jankea.
- Sin reconexión con backoff/heartbeats se pierden eventos y se generan huecos de datos.
- Cache sin normalizar produce estados inconsistentes en reconexiones.

### Escenario de Negocio

> *"Como trader, necesito precios en tiempo real sin perder eventos ni saturar la red."*

### Incidentes reportados
- **Apps de trading:** Subscriptions con diffs/throttle mantienen UX fluida.
- **Apollo/GraphQL specs:** Recomiendan backoff y reintentos para WebSockets.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Apps de trading | Global | Pérdida de eventos y jank sin diffs/backpressure. |
| Apollo/GraphQL | Global | Backoff, heartbeats y re-sync recomendados. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; tiempo real es fuente de bugs/perf. |

**Resumen global**
- Streams sin control generan jank y pérdida de eventos; heartbeats/backoff y diffs son esenciales.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Precios atrasados → decisiones erróneas |
| **UX** | Jank o datos congelados, reconexiones constantes |
| **Técnico** | Pérdida de eventos, estados inconsistentes en cache |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Suscripción simple sin reconexión ni diffs | **INADECUADO:** Pierde eventos, uso excesivo de ancho de banda. |
| **ACEPTABLE** | Subscriptions con reconexión básica | **MEJORA:** Recupera conexión, pero sin diffs ni backpressure. |
| **ENTERPRISE** | **Streams con diffs/backpressure:** reconexión con backoff, heartbeats, aplicar solo diffs, limitar frecuencia a UI, cache normalizado | **ÓPTIMO:** Datos frescos, menor consumo y UI estable. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Heartbeats y reintentos con backoff. Aplicar diffs en cache normalizada. Limitar frecuencia a UI (throttle). Detectar stale y re-sync. Manejo de auth tokens en re-subscribe. |
| **Restricciones Duras (NO permite)** | **Sin backend con diffs:** Si solo manda snapshots completos, consumo sube. **Mobile network variability:** Reconexiones pueden perder eventos; requiere re-sync. **Multiples suscripciones:** Necesita pool y dedup de conexiones. |
| **Criterio de Selección** | graphql_flutter por soporte a cache/subscriptions; backpressure en cliente; heartbeats; re-sync periódico. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Integration (CI) | Reconexión con backoff/heartbeat; re-sync tras corte | Móvil/Backend, CI |
| Performance | Throttle/diff mantiene FPS y consumo razonable | QA/Perf |
| Observabilidad | Métricas `ws.*` (latencia, drop, retries) y cache hits | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Throttle a UI | Limitar frames de precios a 15–20 fps | UX fluida |
| Re-sync | Snap completo tras reconexión | Consistencia |
| Tokens | Re-suscribir al renovar auth | Seguridad |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Pool de conexiones | Dedup de subs por feed | Ahorro de red |
| Backoff/jitter | Evita thundering herd en reconexión | Resiliencia |
| Stale detection | Detectar huecos y forzar snapshot | Datos correctos |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Tiempo real con jank y pérdida de eventos sin control de flujo. |
| Opciones evaluadas | Sub simple; reconexión básica; diffs + backpressure + heartbeats + re-sync. |
| Decisión | Subscriptions con diffs, cache normalizada, throttle, heartbeats, backoff y re-sync. |
| Consecuencias | Mayor complejidad de cliente/observabilidad; requiere soporte de backend. |
| Riesgos aceptados | Cobertura de diffs depende del backend; redes móviles inestables. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Latencia de precios | p95 dentro de SLA | Warning si sube | Decisiones oportunas |
| Pérdida de eventos | 0 tras re-sync | Crítico si > 0 | Datos correctos |
| FPS/jank | Estable con throttle | Alerta si jank sube | UX fluida |
| Reintentos WS | Controlados con backoff | Alerta si explotan | Evita thundering herd |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Subscription | Canal tiempo real en GraphQL sobre WebSocket. |
| Diff | Solo los cambios respecto al estado previo. |
| Backpressure | Controlar flujo para evitar sobrecarga. |
| Heartbeat | Mensaje periódico para verificar conexión viva. |
| Re-sync | Volver a consultar snapshot completo para corregir drift. |

---

## Referencias

- [GraphQL Subscriptions Spec](https://github.com/graphql/graphql-spec)
- [Apollo Subscriptions Best Practices](https://www.apollographql.com/docs/react/data/subscriptions/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
