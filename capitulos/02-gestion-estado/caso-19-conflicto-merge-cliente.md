# Caso 19: Conflicto de Merge en el Cliente
## Resolver Ediciones Concurrentes Offline

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | ediciones concurrentes, offline, merge, conflicto, sincronización |
| **Patrón Técnico** | Conflict Resolution Policies, [CRDT](#term-crdt "Estructuras replicadas sin conflicto, merge determinista.")/OT, Last Writer Wins con contexto |
| **Stack Seleccionado** | Flutter + Riverpod + SQLite/Isar para cola offline + CRDT/patches JSON |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Sin política de merge/CRDT/OT, los parches offline sobrescriben o pierden datos al sincronizar.
- [LWW](#term-lww "Last Writer Wins, estrategia basada en timestamps/versiones.") sin contexto (solo timestamp) descarta ediciones válidas y depende de reloj (skew).
- Sin log de conflictos ni prompts, los usuarios no saben qué cambios ganaron/perdieron.

### Escenario de Negocio

> *"Como usuario, quiero editar datos offline y que al volver a línea mis cambios no se pierdan ni corrompan los de otros."*

### Incidentes reportados
- **Herramientas colaborativas:** Pérdida de datos por merges pobres → adopción de CRDT/OT.
- **Retail app 2022:** Inventario desalineado por merges LWW sin contexto; ventas con stock inexistente.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Colaboración en tiempo real | Global | CRDT/OT adoptados para evitar pérdidas y depender de reloj. |
| Retail 2022 | Apps con inventario offline | LWW sin contexto provocó datos falsos de stock. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; sincronización/merge es fuente de bugs. |

**Resumen global**
- LWW sin contexto es frágil; CRDT/OT reduce pérdida de cambios.
- Registro de conflictos/resoluciones es crítico para soporte y auditoría.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Merge determinista con CRDT/OT; parches idempotentes | Equipo móvil, CI |
| Integration (CI) | Reconcilia cambios offline vs backend sin perder ediciones | Móvil/Backend, CI |
| Seguridad/consistencia | Log de conflictos/resoluciones se guarda y no expone PII | QA/Seguridad |
| Observabilidad | Eventos `sync.merge` con resultado y política aplicada | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Conflictos | Mostrar resumen y opción de resolver cuando la regla de dominio no basta | Transparencia |
| Offline | Encolar parches con TTL; avisar si expiran | Control de expectativas |
| Reloj lógico | Usar vector/lamport clock en CRDT; evitar reliance en reloj de dispositivo | Robusto a skew |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Compactación | TTL para parches y compaction periódica | Control de tamaño |
| Auditoría | Log de merges/conflictos para soporte | Forense y QA |
| Multi-dispositivo | Resolver por política (último actor, dominio) o solicitar usuario | Minimiza pérdida |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Ediciones offline concurrentes se pierden con LWW/simple merge. |
| Opciones evaluadas | Server-wins/client-wins; LWW; CRDT/OT + políticas de dominio + prompts. |
| Decisión | CRDT/OT + políticas por campo/registro + log de conflictos; prompts donde aplique. |
| Consecuencias | Complejidad de modelado y storage de parches; necesidad de UX de conflictos. |
| Riesgos aceptados | Logs crecen si no se compactan; algunos conflictos requieren decisión manual. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Pérdida de cambios | 0 incidentes por merge | Crítico si > 0 | Confianza y datos consistentes |
| Conflictos no resueltos | < 0.5% | Alerta si supera | Soporte manejable |
| Tiempo de reconciliación | p95 < 2 s tras reconexión | Warning si sube | UX fluida |
| Tamaño de log/parches | Controlado (compactación periódica) | Alerta si crece | Rendimiento estable |
| Tickets por merge | ↓ vs baseline | Alerta si no baja | Menos soporte |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-crdt"></a>CRDT | Estructuras replicadas sin conflicto, merge determinista. |
| <a id="term-ot-operational-transformation"></a>OT (Operational Transformation) | Transformación de operaciones concurrentes para mantener consistencia. |
| <a id="term-lww"></a>LWW | Last Writer Wins, estrategia basada en timestamps/versiones. |
| <a id="term-patch-json"></a>Patch JSON | Conjunto de cambios aplicables sobre un documento (diff). |
| <a id="term-resolucion-semantica"></a>Resolución semántica | Decisión basada en reglas de negocio cuando el merge automático no es suficiente. |
| <a id="term-vector-lamport-clock"></a>Vector/Lamport clock | Relojes lógicos para ordenar eventos sin depender del reloj de dispositivo. |

---

## Referencias

- [CRDTs for Mobile](https://martin.kleppmann.com/papers/crdt-apps.pdf)
- [Operational Transformation](https://neil.fraser.name/writing/ot.pdf)
- [RFC 6902 - JSON Patch](https://datatracker.ietf.org/doc/html/rfc6902)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
