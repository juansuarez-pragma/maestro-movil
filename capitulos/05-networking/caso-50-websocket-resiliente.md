# Caso 50: WebSocket Resiliente
## Mantener Conexión Viva en Chat de Soporte Bancario

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | websocket, resiliencia, reconexión, chat soporte |
| **Patrón Técnico** | Reconnection Strategy, [Heartbeat](#term-heartbeat "Ping/pong para verificar que la conexión sigue viva.")/Ping-Pong, Message Queue |
| **Stack Seleccionado** | Flutter + WebSocket channel + interceptors + Riverpod estado de conexión |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- WebSockets en móvil se cortan (cambio de radio, suspensión); sin reconexión + buffer se pierden mensajes.
- Sin backoff/jitter, la reconexión crea thundering herd; sin dedup/IDs hay duplicados.
- Sin re-sync desde último ID, el chat queda incompleto tras reconectar.

### Escenario de Negocio

> *"Como usuario, no quiero perder mensajes en el chat si se corta la conexión."*

### Incidentes reportados
- **Chats/soporte:** Requieren reconexión con re-sync para no perder mensajes.
- **Red móvil:** Variabilidad causa desconexiones frecuentes.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Chats en móvil | Global | Cortes frecuentes; sin buffer/dedup se pierden/duplican mensajes. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; tiempo real/chat es fuente de bugs. |
| WebSocket best practices | Global | Backoff, heartbeats y re-sync recomendados. |

**Resumen global**
- Requiere reconexión con backoff/heartbeat, buffer y re-sync para confiabilidad.

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
| **Capacidades (SÍ permite)** | Detectar desconexión y reconectar con backoff+jitter. Heartbeat/ping-pong para salud. [Buffer de mensajes](#term-buffer-de-mensajes "Cola temporal de mensajes pendientes de envío.") salientes y re-envío tras reconectar. [Re-sync](#term-re-sync "Recuperar mensajes faltantes tras reconectar.") de históricos desde último ID. [Deduplicación](#term-deduplicacion "Evitar procesar dos veces el mismo mensaje.") por message ID. |
| **Restricciones Duras (NO permite)** | **Red sin estabilidad:** No garantiza entrega sin soporte backend (acks). **Batería:** Heartbeats frecuentes consumen energía. **Background:** iOS/Android pueden suspender; requiere estrategias de foreground. |
| **Criterio de Selección** | [Backoff + jitter](#term-backoff-jitter "Reconexión con espera creciente y aleatoria."); heartbeats ajustados; cola persistente opcional; IDs para dedup; re-sync al reconectar. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Integration (CI) | Reconexión con backoff y re-sync desde último ID | Móvil/Backend, CI |
| Observabilidad | Métricas `ws.*` (retries, drop, buffer size) | Móvil/SRE |
| Seguridad/consistencia | Dedup y buffer no duplican ni pierden mensajes | QA/Seguridad |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Estado de conexión | Mostrar conectado/reconectando y fallback a histórico cache | Transparencia |
| Buffer | Cola de mensajes salientes con reenvío tras reconectar | Evita pérdida |
| Heartbeats | Ajustar intervalos según energía/red | Balance |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Backoff/jitter | Evita reconexión agresiva | Estabilidad |
| Re-sync | Obtener mensajes faltantes con last_id | Consistencia |
| Batería | Limitar heartbeats; pausar en background según plataforma | Eficiencia |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Pérdida/duplicado de mensajes por reconexión deficiente. |
| Opciones evaluadas | Reconexión simple; polling; WebSocket con backoff+buffer+dedup+re-sync. |
| Decisión | Reconexión con backoff/jitter, heartbeats, buffer y re-sync con dedup por ID. |
| Consecuencias | Complejidad de estado/buffer; impacto de batería si no se ajusta. |
| Riesgos aceptados | Redes muy inestables; suspensión en background limita confiabilidad. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Mensajes perdidos/duplicados | 0 | Crítico si > 0 | Confiabilidad |
| Reconexiones | Controladas con backoff/jitter | Alerta si explotan | Estabilidad |
| Buffer | Tamaño controlado; sin overflow | Alerta si crece | Recursos |
| Tickets por chat caído | ↓ vs baseline | Alerta si no baja | Soporte |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-heartbeat"></a>Heartbeat | Ping/pong para verificar que la conexión sigue viva. |
| <a id="term-backoff-jitter"></a>Backoff + jitter | Reconexión con espera creciente y aleatoria. |
| <a id="term-deduplicacion"></a>Deduplicación | Evitar procesar dos veces el mismo mensaje. |
| <a id="term-buffer-de-mensajes"></a>Buffer de mensajes | Cola temporal de mensajes pendientes de envío. |
| <a id="term-re-sync"></a>Re-sync | Recuperar mensajes faltantes tras reconectar. |

---

## Referencias

- [WebSocket Best Practices](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)
- [Realtime Messaging Patterns](https://ably.com/topic/websockets)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
