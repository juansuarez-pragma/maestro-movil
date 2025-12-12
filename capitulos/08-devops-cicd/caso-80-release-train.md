# Caso 80: Release Train Quincenal
## Coordinar 4 Squads sin Conflictos

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | release train, coordinación, squads, mobile release |
| **Patrón Técnico** | Trunk-Based Development, Branch Policies, Feature Flags |
| **Stack Seleccionado** | Flutter + main trunk + ramas hotfix + flags + CI con ventanas de freeze |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como organización con 4 squads, necesitamos releases quincenales sin pisarnos ni retrasarnos."*

Múltiples squads pueden bloquearse si no hay cadencia, flags y políticas claras.

### Evidencia de Industria

- **Release trains multi-squad:** Usados para coordinar equipos grandes.
- **Feature flags:** Permiten merge temprano sin exponer features incompletas.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Operacional** | Retrasos por conflictos o freeze prolongados |
| **Calidad** | Bugs si no hay gates claros y sincronización |
| **Productivo** | Tiempo perdido en merges/hotfix frecuentes |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Cada squad libera cuando quiere | **INADECUADO:** Caos y conflictos. |
| **ACEPTABLE** | Calendario sin flags ni gates estrictos | **MEJORA:** Más orden, pero riesgo de bloqueos. |
| **ENTERPRISE** | **Train + flags + gates:** trunk-based, merges tempranos, freeze corto, flags para features incompletas, QA por corte | **ÓPTIMO:** Flujo predecible y riesgo controlado. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Cadencia fija (quincenal). Merges a main con flags para aislar features. Freeze breve para estabilizar. QA/regresión por corte. Hotfix branch para emergencias. |
| **Restricciones Duras (NO permite)** | **Feature sin flag:** No debe mergearse. **Squads sin coordinación:** Requiere acuerdos y gobernanza. **Automatización insuficiente:** Sin CI sólido, freeze se alarga. |
| **Criterio de Selección** | Trunk-based con flags; freeze mínimo; QA coordinado; métricas de estabilidad por release. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Release Train | Cadencia fija de releases compartida por squads. |
| Freeze | Ventana para estabilizar antes de liberar. |
| Trunk-Based | Estrategia de branching centrada en main. |
| Hotfix | Corrección urgente fuera del ciclo. |
| QA por corte | Pruebas alineadas al corte del release. |

---

## Referencias

- [Release Train Model](https://martinfowler.com/bliki/ReleaseTrain.html)
- [Trunk-Based Development](https://trunkbaseddevelopment.com/)
