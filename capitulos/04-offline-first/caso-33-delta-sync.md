# Caso 33: El [Delta Sync](#term-delta-sync "Descarga solo cambios desde un punto conocido.") Inteligente
## Sincronizar Solo lo Necesario en Redes 2G

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | delta sync, ancho de banda limitado, sincronización incremental, 2G |
| **Patrón Técnico** | Delta Sync, Change Data Capture ([CDC](#term-cdc "Change Data Capture; captura de cambios de base de datos.")), Selective Sync |
| **Stack Seleccionado** | Flutter + SQLite/Isar cache + Riverpod + API de cambios (offset/LSN) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Full sync en redes 2G/3G consume datos y falla por timeouts; sin delta ni resume el usuario queda bloqueado.
- Falta de tombstones/CDC produce datasets inconsistentes (no se aplican borrados).
- Sin priorización, tablas críticas compiten con secundarias y se alarga el sync.

### Escenario de Negocio

> *"Como usuario en red lenta, solo quiero descargar cambios, no todo el dataset."*

### Incidentes reportados
- **Apps de campo/logística:** Delta reduce uso de datos/tiempo de sync.
- **BDs con CDC:** LSN/offset para replicar solo cambios; sin ellos se transfiere todo.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Apps de campo/logística | Global | Delta reduce consumo y tiempos; full sync falla en redes pobres. |
| CDC en BD | Global | Replicación eficiente vía LSN/offset; requiere tombstones. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; sync/eficiencia es hallazgo frecuente. |

**Resumen global**
- Delta/CDC es esencial en redes pobres; full sync genera fallos y costos de datos.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Aplicación de deltas y tombstones en cache local | Equipo móvil, CI |
| Integration (CI) | [Resume](#term-resume "Reanudar descarga desde el último byte/offset recibido.") de descargas tras interrupción; prioridad por tabla | Móvil/Backend, CI |
| Rendimiento | Uso de datos/tiempo de sync en redes simuladas 2G/3G | QA/Perf |
| Observabilidad | Métricas `sync.delta` (latencia, bytes, errores) | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Feedback | Mostrar progreso y modo “ahorro de datos” | Transparencia |
| Priorización | Tablas críticas primero; diferir secundarias | UX más rápida |
| Reintentos | Resume con backoff; limitar intentos en redes pobres | Resiliencia |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Tombstones | Requeridos para borrados; GC con TTL | Consistencia |
| Tokens de sync | Gestionar expiración/rotación de `lastSyncToken` | Seguridad |
| CDC backend | Necesario para eficiencia; fallback a full solo si inevitable | Control de costos |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Full sync en redes lentas falla y consume datos. |
| Opciones evaluadas | Full sync; filtros básicos; delta/CDC con resume/prioridad. |
| Decisión | Delta sync con offset/LSN, resume, compresión, priorización. |
| Consecuencias | Requiere API/CDC y manejo de tombstones; mayor complejidad. |
| Riesgos aceptados | Dependencia del backend; sync parcial si tokens expiran. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Uso de datos en sync | ↓ vs full sync | Alerta si no baja | Costos controlados |
| Tiempo de sync en 2G/3G | p95 dentro de SLA | Warning si sube | Operación viable |
| Reintentos/resume | Éxito > 99% tras interrupción | Alerta si baja | Resiliencia |
| Consistencia (deltas/tombstones) | 0 registros fantasma | Crítico si > 0 | Datos confiables |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-delta-sync"></a>Delta Sync | Descarga solo cambios desde un punto conocido. |
| <a id="term-cdc"></a>CDC | Change Data Capture; captura de cambios de base de datos. |
| <a id="term-lsn-offset"></a>LSN/Offset | Marcador de posición en el log de cambios. |
| <a id="term-tombstone"></a>Tombstone | Marcador de eliminación para que el cliente borre registros. |
| <a id="term-resume"></a>Resume | Reanudar descarga desde el último byte/offset recibido. |

---

## Referencias

- [Change Data Capture Patterns](https://martinfowler.com/articles/cdc.html)
- [SQLite Bulk Operations](https://www.sqlite.org/cli.html)
- [RFC 3229 - Delta encoding in HTTP](https://www.rfc-editor.org/rfc/rfc3229)
