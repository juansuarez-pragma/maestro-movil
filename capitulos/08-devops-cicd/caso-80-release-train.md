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

### Problema detectado (técnico)
- Cuatro squads liberando sin cadencia generan conflictos y bloqueos.
- Sin flags ni freeze, features incompletas rompen el release.
- QA y negocio no tienen visibilidad del corte, aumentando retrasos y hotfixes.

### Escenario de Negocio

> *"Como organización con 4 squads, necesitamos releases quincenales sin pisarnos ni retrasarnos."*

### Incidentes reportados
- **Equipos multi-squad:** Releases sin calendario generaron merges tardíos y regresiones.
- **Banca/regulado:** Falta de gates y freeze causó reprobación de QA y retrasos de publicación.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Release trains multi-equipo | Global | Cadencia fija reduce conflictos y hotfixes. |
| Trunk-based | Global | Menos divergencia y menor tiempo de integración. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; gobernanza de releases suele ser débil. |

**Resumen global**
- Release train quincenal con trunk-based, flags y freeze corto reduce conflictos entre squads y mejora previsibilidad; releases ad-hoc escalan regresiones y hotfixes.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Gates | Tests/seguridad/perf por corte | QA/Sec/Perf |
| Release candidate | Build firmado y trazable | Móvil/DevOps |
| Cohesión | Merges tempranos con flags y sin conflictos | Móvil/QA |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Calendario | Visible a squads/negocio | Alineación |
| Flags | Aislar features no listas y limpiar tras release | Deuda controlada |
| Hotfix | Ruta clara fuera del tren | Respuesta rápida |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Branching | Trunk-based + hotfix/release branch cortas | Menos conflicto |
| Freeze | Ventanas cortas definidas | Estabilidad |
| Métricas | Fallas, hotfixes y tiempo de freeze por release | Mejora continua |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Coordinar 4 squads sin conflictos ni retrasos. |
| Opciones evaluadas | Ad-hoc; calendario sin flags; release train quincenal con flags y gates. |
| Decisión | Release train quincenal, trunk-based, freeze corto, flags y QA por corte. |
| Consecuencias | Requiere disciplina y gobernanza; cleanup de flags continuo. |
| Riesgos aceptados | Overhead de coordinación; necesidad de automatización robusta. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Retrasos por conflictos | Tendencia a la baja | Crítico si sube | Cadencia |
| Hotfix fuera de tren | Disminución | Warning si aumenta | Estabilidad |
| Tiempo de freeze | Mantener corto | Alerta si crece | Flujo ágil |
| Bugs por release | ↓ vs baseline | Crítico si sube | Calidad |

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
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
