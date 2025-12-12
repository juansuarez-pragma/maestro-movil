# Caso 33: El Delta Sync Inteligente
## Sincronizar Solo lo Necesario en Redes 2G

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | delta sync, ancho de banda limitado, sincronización incremental, 2G |
| **Patrón Técnico** | Delta Sync, Change Data Capture (CDC), Selective Sync |
| **Stack Seleccionado** | Flutter + SQLite/Isar cache + Riverpod + API de cambios (offset/LSN) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario en red lenta, solo quiero descargar cambios, no todo el dataset."*

Sin delta sync, las sincronizaciones son pesadas, consumen datos y fallan en 2G/3G o con planes limitados.

### Evidencia de Industria

- **Apps de campo/logística:** Delta reduce uso de datos y tiempo de sync.
- **BDs con CDC:** Usan LSN/offset para replicar solo cambios.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Costos de datos altos, tiempos de sync que bloquean operación |
| **UX** | Esperas largas, fallos de sync frecuentes |
| **Técnico** | Errores por timeouts y reintentos excesivos |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Full sync siempre | **INADECUADO:** Costoso, lento, falla en redes pobres. |
| **ACEPTABLE** | Sync con filtros básicos | **MEJORA:** Reduce algo, pero sigue transfiriendo más de lo necesario. |
| **ENTERPRISE** | **Delta sync:** API que expone cambios desde LSN/offset, cache local, retries con resume, compresión, priorización por tabla | **ÓPTIMO:** Menor ancho de banda, sync rápido y reanudable. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Usar `lastSyncToken`/offset para pedir solo cambios. Aplicar cambios en lote en SQLite/Isar. Reanudar descargas parciales. Comprimir payloads. Priorizar tablas críticas y diferir secundarias. |
| **Restricciones Duras (NO permite)** | **Sin soporte backend:** Requiere API/CDC. **Conflictos:** Cambios concurrentes necesitan merge/OT/CRDT. **Datos borrados:** Necesita tombstones o GC controlado. |
| **Criterio de Selección** | Delta sobre full sync para usuarios en redes pobres; resume para resiliencia; compresión y priorización para eficiencia. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Delta Sync | Descarga solo cambios desde un punto conocido. |
| CDC | Change Data Capture; captura de cambios de base de datos. |
| LSN/Offset | Marcador de posición en el log de cambios. |
| Tombstone | Marcador de eliminación para que el cliente borre registros. |
| Resume | Reanudar descarga desde el último byte/offset recibido. |

---

## Referencias

- [Change Data Capture Patterns](https://martinfowler.com/articles/cdc.html)
- [SQLite Bulk Operations](https://www.sqlite.org/cli.html)
