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

### Escenario de Negocio

> *"Como trader, necesito precios en tiempo real sin perder eventos ni saturar la red."*

Feeds rápidos generan cuellos de botella y jank si no se gestionan diffs, backpressure y cache.

### Evidencia de Industria

- **Apps de trading:** Subscriptions con diffs y throttling mantienen UX fluida.
- **Apollo/GraphQL specs:** Recomiendan backoff y reintentos para WebSockets.

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
