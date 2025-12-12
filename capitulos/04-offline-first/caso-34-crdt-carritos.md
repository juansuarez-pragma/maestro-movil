# Caso 34: CRDT para Carritos
## Resolución Automática de Conflictos sin Servidor

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | carritos offline, conflictos, crdt, sincronización |
| **Patrón Técnico** | CRDT (OR-Set/G-Counter), Conflict-free Merge, Event Sourcing |
| **Stack Seleccionado** | Flutter + Riverpod + SQLite/Isar para log de eventos + CRDT libs/implementación propia |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario offline, quiero agregar/eliminar productos y que al sincronizar no se pierdan mis cambios ni se dupliquen."*

Sin un mecanismo de merge determinista, los carritos se corrompen al unir cambios de múltiples nodos offline.

### Evidencia de Industria

- **Herramientas colaborativas:** CRDT evita conflictos sin servidor central resolviendo merges conmutativos.
- **Retail omnicanal:** Carritos multi-dispositivo sufren duplicados y pérdidas sin merge robusto.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Órdenes incorrectas, pérdidas por stock inconsistentes |
| **UX** | Carritos que "pierden" items, frustración |
| **Técnico** | Dificultad de depuración si no hay log de eventos |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Último en escribir gana (LWW) sobre snapshot | **INADECUADO:** Pierde cambios concurrentes, dependiente de reloj. |
| **ACEPTABLE** | Merge manual con timestamps por item | **MEJORA:** Reduce pérdidas, pero puede duplicar o borrar sin reglas claras. |
| **ENTERPRISE** | **CRDT OR-Set/G-Counter + log de eventos:** merges conmutativos, idempotentes, deterministas; replay y auditoría | **ÓPTIMO:** Sin conflictos, trazable y robusto en offline. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Merge determinista entre nodos sin coordinación. Idempotencia por IDs de eventos. Replay de log para reconstruir estado. Manejo de add/remove con OR-Set para evitar resurrección. |
| **Restricciones Duras (NO permite)** | **Stock global:** El cliente no valida stock definitivo; requiere backend. **Crecimiento de log:** Necesita compactación y snapshots. **Reordenamientos semánticos:** CRDT no resuelve reglas de negocio complejas (promos). |
| **Criterio de Selección** | OR-Set/G-Counter para items/cantidades; log en SQLite/Isar; Riverpod para exponer estado derivado. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| CRDT | Estructura de datos replicada sin conflictos, merge conmutativo. |
| OR-Set | Observed-Remove Set; soporta add/remove sin resurrección. |
| G-Counter | Contador solo crecimiento; útil para cantidades sumables. |
| Log de eventos | Registro secuencial de operaciones para replay. |
| Snapshot | Estado comprimido para acelerar reconstrucción. |

---

## Referencias

- [CRDTs in Mobile Applications](https://martin.kleppmann.com/papers/crdt-apps.pdf)
- [JSON Patch RFC 6902](https://datatracker.ietf.org/doc/html/rfc6902)
