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

### Problema detectado (técnico)
- Sin version vectors, no se sabe si un cambio es concurrente o viejo; LWW con reloj desincronizado sobrescribe datos erróneos.
- Sin metadata por actor, merges pierden contexto y no pueden escalar a multi-nodo.
- Falta de bounding/GC de vectores hace crecer metadata en réplicas numerosas.

### Escenario de Negocio

> *"Como app bancaria, necesito saber qué nodo tiene la versión más reciente de un dato para resolver conflictos."*

### Incidentes reportados
- **Sistemas distribuidos:** Version vectors detectan concurrencia sin depender solo de timestamps.
- **Apps offline:** Necesitan identificar conflictos para decidir merge/CRDT/OT.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Sistemas distribuidos | Global | Version vectors son estándar para detectar concurrencia. |
| Apps offline | Global | Evitan sobrescrituras erróneas al detectar conflictos. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; sync/conflictos son hallazgos comunes. |

**Resumen global**
- Version vectors permiten detectar concurrencia y guiar merges; requieren bounding/GC en muchas réplicas.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Dominancia/concurrencia detectada correctamente | Equipo móvil, CI |
| Integration (CI) | Merge guiado por vector evita sobrescrituras erróneas | Móvil/Backend, CI |
| Observabilidad | Logging de vectores y conflictos detectados | Móvil/SRE |
| Bounding | GC/limit de tamaño de vectores funciona | Móvil/QA |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Conflictos | Si concurrente, aplicar CRDT/OT o pedir decisión | Claridad |
| Metadata | Almacenar vector por registro; limitar tamaño | Eficiencia |
| Sincronización | Incluir vectores en payload para detectar conflictos | Robustez |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Bounding | Limitar tamaño de vector; GC de actores inactivos | Control de overhead |
| Tie-break | Definir política cuando vectores iguales | Determinismo |
| Privacidad | No incluir PII en metadata | Cumplimiento |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Sin detección de concurrencia, merges sobrescriben datos erróneos. |
| Opciones evaluadas | Timestamps locales; versión simple; version vectors + CRDT/OT. |
| Decisión | Version vectors por registro + CRDT/OT o política de dominio; bounding/GC. |
| Consecuencias | Metadata adicional; complejidad en bounding y manejo de conflictos. |
| Riesgos aceptados | Overhead en muchas réplicas; requiere políticas claras de tie-break. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Conflictos no detectados | 0 | Crítico si > 0 | Datos correctos |
| Tamaño de metadata | Controlado por bounding/GC | Alerta si crece | Eficiencia |
| Merges erróneos | 0 sobrescrituras por falta de vector | Crítico si > 0 | Consistencia |
| Tickets por datos inconsistentes | ↓ vs baseline | Alerta si no baja | Menos soporte |

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
