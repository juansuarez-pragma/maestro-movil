# Caso 50: WebSocket Resiliente
## Mantener Conexión Viva en Chat de Soporte Bancario

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | websocket, resiliencia, reconexión, chat soporte |
| **Patrón Técnico** | Reconnection Strategy, Heartbeat/Ping-Pong, Message Queue |
| **Stack Seleccionado** | Flutter + WebSocket channel + interceptors + Riverpod estado de conexión |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, no quiero perder mensajes en el chat si se corta la conexión."*

WebSockets en móviles sufren cortes de red, cambios de radio y suspensiones de app; se necesita reconexión y buffer.

### Evidencia de Industria

- **Chats/soporte:** Requieren reconexión con re-sync para no perder mensajes.
- **Red móvil:** Variabilidad causa desconexiones frecuentes.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Mensajes perdidos, chat inestable |
| **Técnico** | Reenvío duplicado si no hay dedup; latencia alta al reconectar |
| **Reputacional** | Soporte percibido como poco confiable |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Conexión simple sin retry ni buffer | **INADECUADO:** Pierde mensajes, UX pobre. |
| **ACEPTABLE** | Reconexión básica | **MEJORA:** Recupera conexión, pero puede perder mensajes. |
| **ENTERPRISE** | **Resiliencia completa:** backoff y jitter, heartbeats, cola de mensajes pendientes, re-sync al reconectar, dedup por IDs | **ÓPTIMO:** Conexión estable y segura para chat. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Detectar desconexión y reconectar con backoff+jitter. Heartbeat/ping-pong para salud. Buffer de mensajes salientes y re-envío tras reconectar. Re-sync de históricos desde último ID. Deduplicación por message ID. |
| **Restricciones Duras (NO permite)** | **Red sin estabilidad:** No garantiza entrega sin soporte backend (acks). **Batería:** Heartbeats frecuentes consumen energía. **Background:** iOS/Android pueden suspender; requiere estrategias de foreground. |
| **Criterio de Selección** | Backoff + jitter; heartbeats ajustados; cola persistente opcional; IDs para dedup; re-sync al reconectar. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Heartbeat | Ping/pong para verificar que la conexión sigue viva. |
| Backoff + jitter | Reconexión con espera creciente y aleatoria. |
| Deduplicación | Evitar procesar dos veces el mismo mensaje. |
| Buffer de mensajes | Cola temporal de mensajes pendientes de envío. |
| Re-sync | Recuperar mensajes faltantes tras reconectar. |

---

## Referencias

- [WebSocket Best Practices](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
- [Realtime Messaging Patterns](https://ably.com/topic/websockets)
