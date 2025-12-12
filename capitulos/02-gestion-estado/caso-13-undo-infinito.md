# Caso 13: Undo Infinito
## Historial de Acciones en un Editor de Documentos Financieros

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | undo/redo, historial, editor financiero, trazabilidad |
| **Patrón Técnico** | Command Pattern, Event Log, Snapshotting |
| **Stack Seleccionado** | Flutter + Riverpod + Immutable State (Freezed) + SQLite para snapshots |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como analista, necesito deshacer cualquier cambio en un documento financiero y auditar qué ocurrió."*

Sin historial persistente, los cambios no son auditables ni reversibles. En fintech, revertir sin rastrear puede violar cumplimiento y generar errores financieros.

### Evidencia de Industria

- **SOX/GDPR:** Exigen trazabilidad de cambios en datos sensibles.
- **Caso ERP 2021:** Pérdida de $200K por no poder revertir cambios en pólizas; no había log de acciones.

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
| **Capacidades (SÍ permite)** | Registrar cada comando con timestamp y usuario. Reconstruir estado reproduciendo eventos desde log más snapshot reciente. Undo/redo persistente entre sesiones. Comparar diffs entre versiones. |
| **Restricciones Duras (NO permite)** | **Crecimiento del log:** Requiere compactación con snapshots; sin ella, arranque lento. **Conflictos concurrentes:** Solo un editor activo por documento o locking; para multiusuario, se necesita OT/CRDT. **PII:** Registrar solo lo necesario para cumplir privacidad. |
| **Criterio de Selección** | Freezed para inmutabilidad y copyWith seguro; Riverpod para providers derivados del log; SQLite para persistencia ligera del log y snapshots. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Command Pattern | Encapsular acciones como objetos inmutables ejecutables y reversibles. |
| Event Log | Registro secuencial de eventos/acciones aplicados al modelo. |
| Snapshot | Estado compacto guardado periódicamente para acelerar el replay del log. |
| Undo/Redo | Revertir/aplicar nuevamente acciones en orden inverso/normal. |
| Inmutabilidad | Estado no se modifica; se crea una nueva copia por cambio. |

---

## Referencias

- [Fowler - Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)
- [Command Pattern (GoF)](https://refactoring.guru/design-patterns/command)
- [NIST Audit Requirements](https://csrc.nist.gov/publications/detail/sp/800-92/final)
