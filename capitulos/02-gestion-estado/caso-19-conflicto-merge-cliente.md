# Caso 19: Conflicto de Merge en el Cliente
## Resolver Ediciones Concurrentes Offline

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | ediciones concurrentes, offline, merge, conflicto, sincronización |
| **Patrón Técnico** | Conflict Resolution Policies, CRDT/OT, Last Writer Wins con contexto |
| **Stack Seleccionado** | Flutter + Riverpod + SQLite/Isar para cola offline + CRDT/patches JSON |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero editar datos offline y que al volver a línea mis cambios no se pierdan ni corrompan los de otros."*

Sin una política de resolución, los cambios locales pueden sobrescribir información crítica o ser descartados silenciosamente.

### Evidencia de Industria

- **Herramientas colaborativas:** Casos de pérdida de datos por merges pobres llevaron a la adopción de CRDT/OT.
- **Retail app 2022:** Inventario desalineado por merges LWW sin contexto causó ventas con stock inexistente.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Datos inconsistentes → decisiones erróneas, pérdidas de stock/ventas |
| **Reputacional** | Usuarios pierden confianza si se "pierden" sus cambios |
| **Técnico** | Bugs difíciles de reproducir sin log de merges y políticas claras |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Sobrescribir servidor > cliente (o viceversa) | **INADECUADO:** Pierde cambios, no auditado. |
| **ACEPTABLE** | LWW (Last Writer Wins) con timestamps | **MEJORA:** Simple, pero depende de reloj y puede descartar ediciones válidas. |
| **ENTERPRISE** | **CRDT/OT + políticas de dominio:** merges por campo/registro, auditoría de parches, prompts al usuario si hay conflicto semántico | **ÓPTIMO:** Minimiza pérdida, permite resolución determinista y auditada. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Encolar parches JSON con metadata (actor, timestamp lógico, versión). Aplicar merges con CRDT (p. ej., OR-Set, LWW-Element-Set) o políticas por campo. Guardar log de conflictos y resoluciones. Solicitar intervención del usuario cuando la regla de dominio no es automática. |
| **Restricciones Duras (NO permite)** | **Sin reloj lógico:** LWW depende de reloj; CRDT evita esto pero aumenta complejidad. **Conflictos semánticos:** Algunos requieren decisión humana (ej. editar mismo monto en direcciones opuestas). **Tamaño del log:** Requiere compactación y TTL para parches. |
| **Criterio de Selección** | Riverpod para exponer estado mergeado; storage local para parches offline; CRDT/OT para merges deterministas; prompts UX solo cuando la regla de dominio no resuelve. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| CRDT | Estructuras replicadas sin conflicto, merge determinista. |
| OT (Operational Transformation) | Transformación de operaciones concurrentes para mantener consistencia. |
| LWW | Last Writer Wins, estrategia basada en timestamps/versiones. |
| Patch JSON | Conjunto de cambios aplicables sobre un documento (diff). |
| Resolución semántica | Decisión basada en reglas de negocio cuando el merge automático no es suficiente. |

---

## Referencias

- [CRDTs for Mobile](https://martin.kleppmann.com/papers/crdt-apps.pdf)
- [Operational Transformation](https://neil.fraser.name/writing/ot.pdf)
- [RFC 6902 - JSON Patch](https://datatracker.ietf.org/doc/html/rfc6902)
