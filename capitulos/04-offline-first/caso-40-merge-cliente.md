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

### Problema detectado (técnico)
- Sin tombstones, los borrados se pierden y resurgen tras sync; merge cliente-servidor queda inconsistente.
- Sin GC, tombstones/datos huérfanos crecen y degradan almacenamiento/perf.
- Borrado vs actualización concurrente requiere política clara o se producen errores de negocio.

### Escenario de Negocio

> *"Como usuario offline, si borro un item, no debe reaparecer tras sincronizar."*

### Incidentes reportados
- **Sistemas distribuidos:** Tombstones evitan resurrección de datos borrados.
- **Apps offline:** Merge sin borrados marcados genera inconsistencias.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| CRDT/sistemas distribuidos | Global | Tombstones previenen resurrección; requieren GC. |
| Apps offline | Global | Sin borrados marcados, resurgen registros. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; sync/merge/borrados son hallazgos frecuentes. |

**Resumen global**
- Tombstones con GC controlado evitan resurrección y controlan almacenamiento; requieren soporte backend y política de TTL.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Tombstone domina merge; no resurrección | Equipo móvil, CI |
| Integration (CI) | GC ejecuta tras confirmación; payloads sincronizan deletes | Móvil/Backend, CI |
| Seguridad/consistencia | TTL aplicado; no se pierde evidencia antes de propagar | QA/Seguridad |
| Observabilidad | Eventos `merge.delete` con actor, timestamp, estado GC | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Visibilidad | Mostrar estado “eliminado” hasta confirmar sync | Transparencia |
| Conflictos | Si hay update vs delete, aplicar política dominio o pedir acción | Claridad |
| GC | Informar si borrados antiguos se depuran | Confianza |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| TTL de tombstones | Definir según dominio/regulación | Evita pérdida prematura |
| Backlog | Monitorear tamaño de GC/backlog | Rendimiento |
| Soporte backend | Validar que servidor respete tombstones y priorice delete | Consistencia |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Borrados reaparecen sin tombstones; storage crece sin GC. |
| Opciones evaluadas | Delete local sin propagar; delete con marca sin GC; tombstones + GC. |
| Decisión | Tombstones con metadata + GC tras confirmación + política de resolución delete/update. |
| Consecuencias | Requiere soporte backend y política de TTL/GC; mayor complejidad de sync. |
| Riesgos aceptados | TTL mal calibrado puede borrar evidencia o inflar storage. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Resurrección de registros | 0 | Crítico si > 0 | Consistencia |
| Tamaño de storage local | Controlado con GC | Alerta si crece | Rendimiento |
| Confirmación de borrados | Alta tasa de sync exitosa | Alerta si baja | Confiabilidad |
| Tickets por datos reaparecidos | ↓ vs baseline | Alerta si no baja | Soporte |

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
- [CRDTs (overview)](https://crdt.tech/)
