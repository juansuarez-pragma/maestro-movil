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

### Escenario de Negocio

> *"Como usuario, la pantalla de productos no debe jankear al cambiar filtros o cantidades."*

Build methods que reconstruyen árboles grandes ante cualquier cambio degradan FPS y consumo. Falta de granularidad produce recomposición excesiva.

### Evidencia de Industria

- **Perf audits en Flutter:** Rebuilds innecesarios son causa común de jank; se recomienda descomponer y usar `const`.
- **Retail app 2023:** Mejora del 25% en frame pacing al mover lógica pesada fuera de build.

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

---

## Referencias

- [Flutter Performance - Build](https://docs.flutter.dev/perf/best-practices#build-methods)
- [Riverpod Select](https://riverpod.dev/docs/concepts/providers/#select)
- [RepaintBoundary Docs](https://api.flutter.dev/flutter/widgets/RepaintBoundary-class.html)
