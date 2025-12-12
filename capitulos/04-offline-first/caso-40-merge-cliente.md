# Caso 40: Merge Cliente-Servidor con Tombstones
## Limpiar Datos Huérfanos en Sincronización Offline

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | merge offline, tombstones, sincronización, borrados |
| **Patrón Técnico** | Tombstone Deletion, Conflict Resolution, Garbage Collection |
| **Stack Seleccionado** | Flutter + SQLite/Isar con marcas de borrado + Riverpod + merge guiado |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario offline, si borro un item, no debe reaparecer tras sincronizar."*

Sin tombstones, los borrados se pierden en merges y resurgen registros eliminados; sin GC, el storage crece.

### Evidencia de Industria

- **Sistemas distribuidos:** Tombstones evitan resurrección de datos borrados.
- **Apps offline:** Merge sin borrados marcados genera inconsistencias.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico/UX** | Datos eliminados reaparecen, errores de negocio |
| **Técnico** | Crecimiento de base local, merges incorrectos |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Borrar local sin propagar | **INADECUADO:** Resurrección de datos. |
| **ACEPTABLE** | Marcar borrado local, sync parcial | **MEJORA:** Reduce resurrección, pero requiere manejo de GC. |
| **ENTERPRISE** | **Tombstones y GC:** marcar borrados con timestamp/actor, propagar a servidor, GC controlado tras confirmación | **ÓPTIMO:** Borrados consistentes y almacenamiento controlado. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Registrar borrados como tombstones con metadata. Evitar reaparecer registros. Hacer GC cuando servidor confirma borrado y TTL expira. Merge respetando prioridades (tombstone domina). |
| **Restricciones Duras (NO permite)** | **Sin soporte backend:** Se requiere que servidor entienda tombstones. **GC agresivo:** Puede eliminar evidencia antes de propagar; requiere TTL adecuado. **Conflictos:** Borrado vs actualización simultánea necesita política clara. |
| **Criterio de Selección** | Tombstones con timestamps y actor; GC diferido tras sync; Riverpod para exponer estado y limpieza. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Tombstone | Marcador de borrado que evita resurrección en merges. |
| GC (Garbage Collection) | Limpieza de tombstones/datos ya confirmados. |
| TTL | Tiempo que se conserva un tombstone antes de limpiar. |
| Dominancia de borrado | Política que prioriza tombstone sobre updates. |
| Merge | Proceso de combinar cambios cliente-servidor. |

---

## Referencias

- [CRDTs and Tombstones](https://martin.kleppmann.com/papers/crdt-apps.pdf)
- [Conflict Resolution Patterns](https://martinfowler.com/articles/patterns-of-distributed-systems/conflict-resolution.html)
