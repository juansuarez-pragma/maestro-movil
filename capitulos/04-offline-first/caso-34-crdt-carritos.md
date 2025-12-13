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

### Problema detectado (técnico)
- LWW/snapshots sin contexto pierden cambios concurrentes y dependen de reloj; carritos se corrompen al merge offline→online.
- Sin OR-Set/G-Counter, remover/añadir items puede “resucitar” o duplicar productos.
- Sin log de eventos ni compactación, el historial crece y dificulta auditoría/perf.

### Escenario de Negocio

> *"Como usuario offline, quiero agregar/eliminar productos y que al sincronizar no se pierdan mis cambios ni se dupliquen."*

### Incidentes reportados
- **Herramientas colaborativas:** CRDT evita conflictos sin servidor central resolviendo merges conmutativos.
- **Retail omnicanal:** Carritos multi-dispositivo sufren duplicados y pérdidas sin merge robusto.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| CRDT en colab retail | Global | Merges deterministas reducen pérdidas de cambios. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; sync/merge es foco de bugs. |
| Estudios omnicanal | Ecommerce | Duplicados/pérdidas en carritos sin merge sólido. |

**Resumen global**
- CRDT/merge determinista minimiza pérdidas/duplicados; logs requieren compactación para perf.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | OR-Set/G-Counter mergea determinísticamente; add/remove sin resurrección | Equipo móvil, CI |
| Integration (CI) | Merge multi-dispositivo/offline converge; log compacta con snapshot | Móvil/Backend, CI |
| Observabilidad | Eventos `cart.merge` con actor/timestamp lógico; tamaño de log | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Conflictos visibles | Mostrar si backend ajusta stock/precio | Transparencia |
| Offline | Permitir edición; sync mostrar progreso y ajustes | UX consistente |
| Compactación | Snapshots periódicos para startup rápido | Rendimiento |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Tombstones | Mantener para deletes; GC con TTL | Consistencia |
| Stock | Validar stock en backend en checkout | Cliente no es verdad absoluta |
| Crecimiento de log | Compactar; límites de tamaño | Control perf |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Pérdida/duplicación de items en carritos multi-dispositivo/offline. |
| Opciones evaluadas | LWW/snapshot; merge manual; CRDT (OR-Set/G-Counter) + log + compactación. |
| Decisión | CRDT OR-Set/G-Counter + log de eventos + compactación/snapshots. |
| Consecuencias | Complejidad de modelado y almacenamiento; requiere compactación. |
| Riesgos aceptados | Tamaño de log; reglas de negocio (promos) requieren validación backend. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Pérdida/duplicado de items | 0 incidentes | Crítico si > 0 | Carrito confiable |
| Convergencia de estado | 100% nodos convergen | Alerta si divergen | Consistencia |
| Tamaño de log | Controlado (compactación) | Alerta si crece | Rendimiento |
| Tickets por carrito inconsistente | ↓ vs baseline | Alerta si no baja | Menos soporte |

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
