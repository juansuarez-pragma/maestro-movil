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

### Escenario de Negocio

> *"Como usuario, quiero ver el mismo carrito en web, tablet y móvil; cualquier cambio debe reflejarse al instante y nunca perderse."*

Cuando el usuario abre varias pestañas o dispositivos, modificaciones concurrentes generan "carritos fantasma" (items que aparecen/desaparecen) y pérdidas de sincronía.

### Evidencia de Industria

- **Estudio Baymard 2023:** El 70% de los carritos se abandonan; inconsistencias entre dispositivos aumentan el abandono.
- **Retailer EU 2022:** Incidente de doble checkout por falta de bloqueo optimista provocó devoluciones masivas.

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
