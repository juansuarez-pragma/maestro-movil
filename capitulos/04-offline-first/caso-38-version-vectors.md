# Caso 38: Version Vectors en Banca
## Tracking de Cambios Distribuidos

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | version vectors, sincronización, conflictos, offline |
| **Patrón Técnico** | Version Vectors, Conflict Detection, Distributed Sync |
| **Stack Seleccionado** | Flutter + SQLite/Isar para metadata de versiones + Riverpod para estado |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como app bancaria, necesito saber qué nodo tiene la versión más reciente de un dato para resolver conflictos."*

Sin version vectors, se desconoce si un cambio es concurrente o viejo, causando sobrescrituras erróneas.

### Evidencia de Industria

- **Sistemas distribuidos:** Version vectors detectan concurrencia sin depender solo de timestamps.
- **Apps offline:** Necesitan identificar conflictos para decidir merge/CRDT/OT.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico/UX** | Datos incorrectos, decisiones erróneas |
| **Técnico** | Conflictos no detectados, pérdida de cambios |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Timestamps locales | **INADECUADO:** Relojes desincronizados causan errores. |
| **ACEPTABLE** | Versiones simples incrementales | **MEJORA:** Detecta alguna obsolescencia, pero no concurrencia múltiple. |
| **ENTERPRISE** | **Version vectors:** contador por replica/actor, detección de concurrencia, merge guiado | **ÓPTIMO:** Identifica conflictos y orden parcial de cambios. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Detectar si una versión domina otra o es concurrente. Guiar merge (CRDT/OT) o pedir resolución manual. Mantener metadata compacta por actor. Almacenar vector por registro. |
| **Restricciones Duras (NO permite)** | **Overhead:** Vectores crecen con el número de réplicas; requiere garbage collection o bounding. **Semántica de dominio:** No resuelve qué merge aplicar; solo detecta conflicto. **Reloj lógico:** Aún se necesita política de tie-break para ciertos casos. |
| **Criterio de Selección** | Version vectors para detectar concurrencia; combinarlos con CRDT/OT o políticas de dominio; persistir en cache local. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Version Vector | Mapa de contador por replica/actor para detectar orden parcial. |
| Dominancia | Cuando un vector es >= en todos los contadores a otro. |
| Concurrencia | Cambios que no dominan entre sí; requieren merge. |
| Actor/Replica | Nodo que genera cambios. |
| Merge guiado | Estrategia de resolución basada en detectar conflicto. |

---

## Referencias

- [Version Vectors (Petersen et al.)](https://www.cs.cornell.edu/home/rvr/papers/VersionVectors.pdf)
- [Distributed Systems - Conflict Detection](https://martin.kleppmann.com/papers/)
