# Caso 98: Deuda Técnica Cuantificada
## Métricas para Convencer al Negocio

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | deuda técnica, métricas, priorización, [ROI](#term-roi "Retorno de inversión de pagar la deuda.") |
| **Patrón Técnico** | Tech Debt Register, Impact/Risk Scoring, ROI Calculation |
| **Stack Seleccionado** | Flutter repo + métricas (lint, coverage, cycle time) + tablero de deuda |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Sin cuantificar deuda, negocio no prioriza y la deuda crece.
- Listas informales no permiten comparar impacto/ROI.
- Falta de métricas automáticas deja la conversación en opinión.

### Escenario de Negocio

> *"Como equipo, necesito justificar inversión en refactors con datos, no solo intuición."*

### Incidentes reportados
- **Bugs recurrentes:** Áreas con deuda alta generan hotfixes y retrasos.
- **Velocidad baja:** Falta de visibilidad en cycle time/defectos prolongó entregas.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Tech debt practices | Global | Registros con puntajes mejoran priorización. |
| Accelerate metrics | Global | [Cycle time](#term-cycle-time "Tiempo desde commit a producción.")/defectos correlacionan con deuda. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; deuda de configuración/seguridad es común. |

**Resumen global**
- Registro cuantificado con métricas y ROI convierte deuda en decisión de negocio; listas cualitativas son insuficientes.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Datos | Métricas automáticas se generan (lint/coverage/cycle) | DevOps/QA |
| ROI | Cálculo y priorización revisados | Liderazgo/Producto |
| Impacto | Bugs/hotfixes disminuyen tras mitigaciones | Móvil/QA |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Transparencia | Tablero visible a negocio/tech | Alineación |
| Priorización | Incluir deuda en cada planning | Constancia |
| Comunicación | Justificar con métricas, no solo opinión | Credibilidad |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Automatización | Pipelines que exportan métricas | Escalabilidad |
| Gobernanza | Dueño por ítem y fecha de revisión | Accountability |
| Revisión | Cadencia mensual/trimestral | Frescura de datos |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Deuda sin cuantificar no se prioriza. |
| Opciones evaluadas | Listas cualitativas; registro con severidad; registro cuantificado con métricas y ROI. |
| Decisión | Registro cuantificado con métricas automáticas y ROI. |
| Consecuencias | Requiere pipelines y disciplina; aprendizaje en puntuación. |
| Riesgos aceptados | Métricas parciales; juicio humano sigue necesario. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Cycle time | ↓ tras abordar deuda prioritaria | Warning si no baja | Velocidad |
| Bugs/hotfixes recurrentes | ↓ | Crítico si sube | Calidad |
| [Cobertura](#term-cobertura "% de código cubierto por tests.") | ↑ en módulos críticos | Warning si estanca | Confianza |
| ROI de deuda | Casos con ROI>1 priorizados | Alerta si no | Alineación negocio |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-deuda-tecnica"></a>Deuda técnica | Compromisos que incrementan costo futuro de cambios. |
| <a id="term-roi"></a>ROI | Retorno de inversión de pagar la deuda. |
| <a id="term-cycle-time"></a>Cycle time | Tiempo desde commit a producción. |
| <a id="term-cobertura"></a>Cobertura | % de código cubierto por tests. |
| <a id="term-severidad"></a>Severidad | Gravedad/impacto de un ítem de deuda. |

---

## Referencias

- [Tech Debt Quantification](https://martinfowler.com/articles/technicalDebt.html)
- [Accelerate Metrics](https://itrevolution.com/book/accelerate/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
