# Caso 98: Deuda Técnica Cuantificada
## Métricas para Convencer al Negocio

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | deuda técnica, métricas, priorización, ROI |
| **Patrón Técnico** | Tech Debt Register, Impact/Risk Scoring, ROI Calculation |
| **Stack Seleccionado** | Flutter repo + métricas (lint, coverage, cycle time) + tablero de deuda |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo, necesito justificar inversión en refactors con datos, no solo intuición."*

Sin cuantificar deuda, negocio no prioriza mejoras; la deuda crece y ralentiza entregas.

### Evidencia de Industria

- **Equipos maduros:** Mantienen registro de deuda con impacto y costo.
- **ROI en tech debt:** Correlaciona con velocidad y defectos.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Productivo** | Velocidad baja, bugs frecuentes |
| **Económico** | Costos de mantenimiento altos |
| **Reputacional** | Bugs recurrentes afectan confianza |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Listas informales sin métricas | **INADECUADO:** Difícil priorizar. |
| **ACEPTABLE** | Registro con severidad cualitativa | **MEJORA:** Algo de visibilidad, pero subjetivo. |
| **ENTERPRISE** | **Registro cuantificado:** impacto (defectos, tiempo), probabilidad, costo de fix, ROI; métricas automáticas (lint, cobertura, cycle time) | **ÓPTIMO:** Decisiones basadas en datos y ROI. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Registrar deuda con puntaje impacto/probabilidad. Medir coverage, lints, tiempos de ciclo. Asociar bugs/incidentes. Calcular ROI y priorizar. Revisar en cadencias de planificación. |
| **Restricciones Duras (NO permite)** | **Subjetividad residual:** Aún requiere juicio. **Métricas incompletas:** No capturan todo el dolor. **Costo de medición:** Automatizar para evitar sobrecarga. |
| **Criterio de Selección** | Tablero centralizado, métricas automáticas, ROI para priorizar, revisiones periódicas con negocio. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Deuda técnica | Compromisos que incrementan costo futuro de cambios. |
| ROI | Retorno de inversión de pagar la deuda. |
| Cycle time | Tiempo desde commit a producción. |
| Cobertura | % de código cubierto por tests. |
| Severidad | Gravedad/impacto de un ítem de deuda. |

---

## Referencias

- [Tech Debt Quantification](https://martinfowler.com/articles/technicalDebt.html)
- [Accelerate Metrics](https://itrevolution.com/book/accelerate/)
