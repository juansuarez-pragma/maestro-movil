# Caso 11: El Carrito Fantasma
## Sincronizar Estado entre 5 Pestañas y 3 Dispositivos

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | carrito compartido, multi-dispositivo, sincronización en tiempo real, consistencia eventual |
| **Patrón Técnico** | Conflict-free Replicated Data Types (CRDT), Event Sourcing, State Streaming |
| **Stack Seleccionado** | Flutter + Riverpod + WebSockets/GraphQL Subscriptions + SQLite para cache |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Estado local por pestaña/dispositivo sin resolución de conflictos produce “carritos fantasma”: items que aparecen/desaparecen según la réplica.
- Sin CRDT/event sourcing ni binding multi-dispositivo, los merges dependen de timestamps y pierden cambios concurrentes.
- Falta de replay/auditoría dificulta investigar pérdidas de items o doble checkout.

### Escenario de Negocio

> *"Como usuario, quiero ver el mismo carrito en web, tablet y móvil; cualquier cambio debe reflejarse al instante y nunca perderse."*

### Incidentes reportados
- **Baymard 2023:** 70% de carritos se abandonan; inconsistencias entre dispositivos aumentan abandono.
- **Retailer EU 2022:** Doble checkout por falta de control optimista → devoluciones masivas.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Baymard 2023 | Ecommerce global | 70% abandono de carritos; fricción/inconsistencia contribuye. |
| Retailers EU/US (estudios internos 2022) | Multi-device carts | Incidentes de doble checkout por falta de control de concurrencia. |
| NowSecure (2024, apps móviles) | 1,000+ apps | 85% fallan ≥1 control MASVS; sincronización de estado es fuente de bugs UX. |

**Resumen global**
- Alta tasa de abandono; inconsistencias multi-dispositivo agravan pérdidas.
- Falta de controles de concurrencia genera doble checkout y soporte costoso.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Pérdida de ventas, devoluciones, fraude por duplicación de órdenes |
| **Reputacional** | Usuarios pierden confianza al ver carritos inconsistentes |
| **Técnico** | Deuda por soluciones ad-hoc que no escalan con concurrencia |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Estado local por pestaña, sync solo al checkout | **INADECUADO:** Conflictos silenciosos, pérdida de items, UX inconsistente. |
| **ACEPTABLE** | Sync periódico (polling) + merge por timestamp | **CUMPLE MÍNIMOS:** Reduce pérdida, pero sufre de race conditions y alta latencia percibida. |
| **ENTERPRISE** | **CRDT + Event Sourcing:** cambios como eventos idempotentes, merge automático; Riverpod para reactividad; WebSockets para broadcast | **ÓPTIMO:** Resolución determinista de conflictos, consistencia eventual con buena UX, trazabilidad de eventos para auditoría. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Consistencia eventual multi-dispositivo con CRDT (p. ej., GCounter/OR-Set) para items/cantidades. Rehidratación del estado desde Event Store al iniciar. Reconexión automática con replay de eventos pendientes. Caché local en SQLite para funcionamiento offline breve. |
| **Restricciones Duras (NO permite)** | **Orden total:** No garantiza orden global de eventos, solo merge conmutativo. **Conflictos de negocio:** Reglas de inventario/stock deben resolverse en backend; el cliente no puede validar stock definitivo. **Offline prolongado:** Riesgo de expiración de items/promos. |
| **Criterio de Selección** | Riverpod por simplicidad de dependencias y providers selectivos; CRDT sobre locking optimista/pesimista para evitar bloqueos y ofrecer merge determinista; Event Sourcing por trazabilidad y replay. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | CRDT merge produce estado convergente; providers notifican sin fugas | Equipo móvil, CI |
| Integration (CI) | Replay de eventos tras reconexión reconstruye carrito igual en 3 dispositivos | Móvil/Backend, CI + staging |
| Seguridad/consistencia | Eventos duplicados/idempotencia no duplican items | QA/Seguridad, dispositivos reales |
| Observabilidad | Eventos `cart.*` con device_id, trace_id; métricas de divergencia | Móvil/SRE, CI |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Indicadores de sync | Mostrar estado de sincronización/reintento; cola visible | Reduce confusión de usuario |
| Conflictos visibles | Marcar items expirados o sin stock tras sync | Evita sorpresas en checkout |
| Offline breve | Permitir cambios cortos; avisar si no se pudo sincronizar | Controla expectativas |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Retención de eventos | Compaction con snapshots para evitar crecimiento infinito | Startup rápido |
| Inventario | Validar stock/precio en backend al confirmar | Cliente no decide stock |
| Doble checkout | Idempotency-Key en checkout final | Previene cargos duplicados |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Carritos inconsistentes en multi-dispositivo/pestaña y riesgo de doble checkout. |
| Opciones evaluadas | Estado local + polling; locking optimista; CRDT + event sourcing + broadcast. |
| Decisión | CRDT + event sourcing + WebSockets/subs; Riverpod para reactividad selectiva. |
| Consecuencias | Más complejidad de modelado; requiere backend que soporte eventos/merge; compaction necesaria. |
| Riesgos aceptados | Eventual consistency (no orden total); offline prolongado puede expirar promos. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Divergencia entre dispositivos | < 0.5% de sesiones con diferencias | Alerta si ≥ 0.5% | Carrito consistente, menos soporte |
| Doble checkout | 0 incidentes | Crítico si > 0 | Evita pérdidas y devoluciones |
| Tiempo de propagación | p95 < 2 s entre dispositivos | Warning si se acerca | UX de “tiempo real” |
| Reconexión/replay éxito | > 99% | Alerta si baja | Robustez ante desconexiones |
| Abandono atribuido a sync | ↓ vs baseline | Alertar si no mejora | Mayor conversión |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| CRDT | Estructura de datos replicada sin conflicto; merge conmutativo y asociativo. |
| Event Sourcing | Persistir cambios como eventos inmutables y reconstruir estado reproduciéndolos. |
| Consistencia eventual | Estado converge después de propagar y reconciliar eventos. |
| Replay de eventos | Reaplicar eventos desde el log para reconstruir el estado actual. |
| Idempotencia | Aplicar el mismo evento múltiples veces no cambia el resultado final. |

---

## Referencias

 - [CRDTs in Mobile Applications (Martin Kleppmann)](https://martin.kleppmann.com/papers/crdt-apps.pdf)
 - [Nielsen Norman Group - Cart UX](https://www.nngroup.com/articles/shopping-cart-experience/)
 - [RFC 6902 - JSON Patch](https://datatracker.ietf.org/doc/html/rfc6902)
 - [Baymard Institute - Cart Abandonment 2023](https://baymard.com/lists/cart-abandonment-rate)
 - [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
