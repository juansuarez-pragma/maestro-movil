# Caso 24: Build Methods Costosos
## Identificar y Eliminar Rebuilds Innecesarios

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | rendimiento UI, rebuild, memoización, selectors |
| **Patrón Técnico** | Widget Decomposition, Memoization, Selector Pattern |
| **Stack Seleccionado** | Flutter + Riverpod Selectors + `const` widgets + RepaintBoundary |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Build monolítico y setState global causan recomposición masiva ante cambios mínimos → jank y alto consumo.
- Lógica/pParsing en build bloquea frames; falta de memoización/const/RepaintBoundary amplifica repaints.
- Sin medición/selección granular, optimizar es opaco y costoso.

### Escenario de Negocio

> *"Como usuario, la pantalla de productos no debe jankear al cambiar filtros o cantidades."*

### Incidentes reportados
- **Perf audits Flutter:** Rebuilds innecesarios son causa común de jank; descomponer y usar `const` mejora FPS.
- **Retail app 2023:** +25% en frame pacing al mover lógica pesada fuera de build.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Auditorías Flutter perf | Global | Rebuild masivo = jank; composición + const reduce trabajo. |
| Retail 2023 | Catálogos | +25% frame pacing al aislar lógica y RepaintBoundary. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; perf/UI es hallazgo recurrente. |

**Resumen global**
- Rebuild innecesario degrada FPS y UX; descomposición y memoización son claves.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Jank, input lag |
| **Técnico** | Dificultad para optimizar si no hay medición ni desac acoplamiento |
| **Reputacional** | Percepción de app lenta en listados críticos |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | setState global, build monolítico | **INADECUADO:** Rebuild masivo ante cualquier cambio. |
| **ACEPTABLE** | Separar widgets + `const` donde aplica | **MEJORA:** Reduce rebuild, pero sin selectores ni memoización. |
| **ENTERPRISE** | **Granularidad + memoización:** descomponer por responsabilidad, usar Riverpod selectors, `const`, `RepaintBoundary`, mover cálculos fuera de build | **ÓPTIMO:** Rebuild mínimo, UI estable, fácil de perfilar. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Rebuild solo de widgets afectados via selectors. Uso de `const` para evitar diff. Memoizar resultados de cálculos repetitivos. Aislar áreas de repaint con `RepaintBoundary`. Mover parsing/cálculos a init/Isolate. |
| **Restricciones Duras (NO permite)** | **Sin medición:** Necesario perf profiling (DevTools). **Layouts complejos:** Widgets intrínsecos caros siguen costosos. **Estado compartido mal modelado:** Selectors no ayudan si todo cambia. |
| **Criterio de Selección** | Riverpod selectors para granularidad; `const` y composición para minimizar diffs; RepaintBoundary para evitar repaints globales. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Performance (DevTools) | Reducción de rebuilds y frame time | QA/Perf |
| Unit (CI) | Selectors notifican solo cambios necesarios | Equipo móvil, CI |
| Integration (CI) | Lógica pesada fuera de build; parsing en init/Isolate | Móvil/QA, CI |
| Observabilidad | Métricas `ui.rebuilds`, `frame_time_p95` | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Composición | Descomponer por responsabilidad; `const` cuando aplique | Menos diff |
| Aislamiento | `RepaintBoundary` para widgets caros | Aísla repaints |
| Memoización | Cache de cálculos recurrentes | Reduce CPU |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Perfilado continuo | Revisar flame charts en features clave | Detecta regresiones |
| Patrón de estado | Usar selectors para granularidad | Minimiza rebuilds |
| Costo de layout | Evitar widgets intrínsecos y `shrinkWrap` salvo necesidad | Menos trabajo |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Rebuild masivo/jank por build monolítico. |
| Opciones evaluadas | setState global; separación básica; descomposición + selectors + memoización + aislar repaints. |
| Decisión | Descomposición fina + selectors/memoización + RepaintBoundary + lógica fuera de build. |
| Consecuencias | Más código y disciplina; necesidad de medición constante. |
| Riesgos aceptados | Complejidad adicional; riesgo de micro-optimizaciones si se mide mal. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Rebuilds innecesarios | ↓ vs baseline | Alerta si suben | UX más fluida |
| Frame time p95 | < 16 ms | Warning si se acerca | 60fps sostenido |
| CPU en build | Reducción medible en profiling | Alerta si sube | Menos consumo |
| Tickets “app lenta” | ↓ vs baseline | Alerta si no baja | Soporte controlado |
| Jank | Mínimo según DevTools | Alerta si crece | Confianza UX |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Selector | Deriva parte del estado para limitar rebuild. |
| `const` widget | Widget inmutable que no se vuelve a construir si no cambian props. |
| RepaintBoundary | Aísla repaints a una subparte del árbol. |
| Memoización | Cache de resultados de computo para reutilizar. |
| Build tree | Árbol de widgets que describe la UI. |
| Flame chart | Visualización de tiempos de build/layout/paint en profiling. |

---

## Referencias

- [Flutter Performance - Build](https://docs.flutter.dev/perf/best-practices#build-methods)
- [Riverpod Select](https://riverpod.dev/docs/concepts/providers/#select)
- [RepaintBoundary Docs](https://api.flutter.dev/flutter/widgets/RepaintBoundary-class.html)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
