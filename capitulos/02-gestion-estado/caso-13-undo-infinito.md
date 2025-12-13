# Caso 13: Undo Infinito
## Historial de Acciones en un Editor de Documentos Financieros

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | undo/redo, historial, editor financiero, trazabilidad |
| **Patrón Técnico** | [Command Pattern](#term-command-pattern "Encapsular acciones como objetos inmutables ejecutables y reversibles."), [Event Log](#term-event-log "Registro secuencial de eventos/acciones aplicados al modelo."), Snapshotting |
| **Stack Seleccionado** | Flutter + Riverpod + Immutable State (Freezed) + SQLite para snapshots |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Sin log persistente/command/event sourcing, undo/redo queda en memoria y se pierde al cerrar la app; no hay auditoría.
- Crecimiento del log sin snapshots degrada arranque y uso; sin compactación es inviable.
- Falta de correlación (usuario, timestamp, versión) impide cumplir SOX/GDPR y reproducir errores.

### Escenario de Negocio

> *"Como analista, necesito deshacer cualquier cambio en un documento financiero y auditar qué ocurrió."*

### Incidentes reportados
- **Caso ERP 2021:** Pérdida de $200K por no poder revertir cambios en pólizas; sin log de acciones.
- **SOX/GDPR:** Exigen trazabilidad completa; undo/redo parcial no cumple.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Auditorías SOX/GDPR (varios) | Global | Falta de log/auditoría es hallazgo frecuente en apps financieras. |
| ACFE Fraud 2022 | Global | Fraudes internos requieren trazabilidad; ausencia de logs aumenta pérdidas. |
| Estudios de productividad (editor apps) | Global | Undo/redo persistente reduce errores y tickets de soporte. |

**Resumen global**
- La ausencia de historial persistente es hallazgo común en auditorías.
- Undo/redo robusto baja errores y soporte; necesario para cumplimiento y forense.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Ajustes erróneos causan pérdidas o cierres de periodo incorrectos |
| **Regulatorio** | Sin trazabilidad, incumplimiento de auditorías |
| **Técnico** | Sin log no se pueden reproducir bugs ni estados previos |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Guardar solo estado final en memoria | **INADECUADO:** No hay undo/redo real, se pierde contexto de acciones. |
| **ACEPTABLE** | Pila de estados en memoria con tope | **MEJORA:** Permite undo limitado, pero se pierde al cerrar app y consume memoria. |
| **ENTERPRISE** | **Command + Event Log + Snapshots:** comandos inmutables, log persistente, snapshots periódicos para acelerar replay | **ÓPTIMO:** Undo/redo infinito, trazabilidad para auditoría, startup rápido gracias a snapshots. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Registrar cada comando con timestamp y usuario. Reconstruir estado reproduciendo eventos desde log + snapshot reciente. Undo/redo persistente entre sesiones. Comparar diffs entre versiones. |
| **Restricciones Duras (NO permite)** | **Crecimiento del log:** Requiere compactación con snapshots; sin ella, arranque lento. **Conflictos concurrentes:** Solo un editor activo por documento o locking; para multiusuario, se necesita OT/CRDT. **PII:** Registrar solo lo necesario. |
| **Criterio de Selección** | Freezed para inmutabilidad y copyWith seguro; Riverpod para providers derivados del log; SQLite para persistencia ligera del log y snapshots. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Comandos aplican/revierten sin mutar estado previo | Equipo móvil, CI |
| Integration (CI) | Replay desde snapshot + log reconstruye documento idéntico | Móvil/Backend, CI + staging |
| Seguridad/consistencia | Ediciones concurrentes bloqueadas/serializadas; no se pierden acciones | QA/Seguridad |
| Observabilidad | Eventos `editor.command` con usuario, timestamp, versión | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Undo/redo | Disponibles entre sesiones; límites según retención/log | UX confiable |
| Snapshots | Guardar cada N eventos/tiempo para arranque rápido | Balance rendimiento |
| [Auditoría](#term-auditoria "Registro fiable para investigación y cumplimiento.") | Mostrar versión/timestamp en vistas de historial | Transparencia |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Snapshots/compactación | Compactar log periódicamente | Control de tamaño y arranque |
| Retención | TTL según regulación; pseudoanonimizar datos sensibles | Cumplimiento |
| Colaboración | Lock de documento o transición a OT/CRDT para multiusuario | Evita conflictos |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Undo limitado en memoria; sin trazabilidad/auditoría. |
| Opciones evaluadas | Pila en memoria; log simple sin snapshots; command + event log + snapshots. |
| Decisión | Command pattern + event log persistente + snapshots periódicos. |
| Consecuencias | Mayor complejidad de manejo de log/snapshots; costo en almacenamiento. |
| Riesgos aceptados | Crecimiento de log si no se compacta; un editor a la vez. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Recuperación de estado | 100% de reconstrucciones iguales desde snapshot+log | Alerta si difiere | Confianza y cumplimiento |
| Tiempo de arranque | p95 < 2 s tras snapshot reciente | Warning si sube | UX rápida |
| Retención/compactación | Cumple TTL y tamaño controlado | Alerta si log crece sin compactar | Control de storage |
| Tickets por pérdida de cambios | ↓ vs baseline | Alerta si no baja | Menos soporte |
| Auditorías | 0 hallazgos por falta de trazabilidad | Crítico si > 0 | Cumplimiento SOX/GDPR |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-command-pattern"></a>Command Pattern | Encapsular acciones como objetos inmutables ejecutables y reversibles. |
| <a id="term-event-log"></a>Event Log | Registro secuencial de eventos/acciones aplicados al modelo. |
| <a id="term-snapshot"></a>Snapshot | Estado compacto guardado periódicamente para acelerar el replay del log. |
| <a id="term-undo-redo"></a>Undo/Redo | Revertir/aplicar nuevamente acciones en orden inverso/normal. |
| <a id="term-inmutabilidad"></a>Inmutabilidad | Estado no se modifica; se crea una nueva copia por cambio. |
| <a id="term-auditoria"></a>Auditoría | Registro fiable para investigación y cumplimiento. |

---

## Referencias

- [Fowler - Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)
- [Command Pattern (GoF)](https://refactoring.guru/design-patterns/command)
- [NIST Audit Requirements](https://csrc.nist.gov/publications/detail/sp/800-92/final)
- [ACFE Report 2022](https://acfepublic.s3-us-west-2.amazonaws.com/2022-Report-to-the-Nations.pdf)
