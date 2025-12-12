# Caso 22: 60 FPS o Muerte
## Animaciones Complejas en Pantallas de Trading

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | 60fps, trading, animaciones, rendimiento, jank |
| **Patrón Técnico** | Frame Budgeting, Layering, Precomposición |
| **Stack Seleccionado** | Flutter + `AnimatedBuilder`/`ImplicitlyAnimatedWidgets` + `TickerProvider` + `rive` (opcional) |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como trader, necesito ver gráficas y animaciones fluidas a 60fps mientras ejecuto órdenes."*

Animaciones y actualizaciones de mercado concurrentes pueden exceder el presupuesto de frame (16ms), provocando jank y errores de interacción.

### Evidencia de Industria

- **Trading apps:** Requieren UI fluida para confianza; jank reduce conversión.
- **Flutter perf:** Recomienda minimizar trabajo en el thread UI y reducir recomposición.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Decisiones tardías por UI lenta; pérdida de operaciones |
| **UX** | Jank perceptible, toques no responden |
| **Técnico** | Sobrecarga de CPU/GPU, battery drain |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | setState masivo en cada tick de datos | **INADECUADO:** Rebuild global, jank severo. |
| **ACEPTABLE** | `AnimatedBuilder` focalizado + throttling de actualizaciones | **MEJORA:** Menos rebuild, pero sin separar capas ni precomponer. |
| **ENTERPRISE** | **Presupuesto de frame:** separar capa estática vs dinámica, usar widgets animados eficientes, throttle/diff de datos, trabajo pesado en Isolate, precompose vectores (Rive/Lottie) | **ÓPTIMO:** Mantiene 60fps, reduce jank y consumo. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Throttle de updates de mercado (p. ej., 15–20 fps) mientras mantiene animaciones a 60fps. Dividir UI en capas: estática cacheada y overlays dinámicos. Uso de `RepaintBoundary` para aislar regiones. Preprocesar datos en Isolate. |
| **Restricciones Duras (NO permite)** | **GPU bound:** Animaciones pesadas o sombras múltiples siguen afectando GPU. **Layouts profundos:** Árbol muy grande aumenta costo de build. **Sin assets optimizados:** Rive/Lottie mal optimizados rompen presupuesto. |
| **Criterio de Selección** | Animations con `AnimatedBuilder`/implicitos para granularidad; RepaintBoundary para aislar; throttling/diff de streams; Isolate para parsing de ticks. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Presupuesto de frame | Tiempo máximo por frame (16ms a 60fps). |
| RepaintBoundary | Widget que aísla repaints a una subparte del árbol. |
| Throttle/Diff | Limitar frecuencia y enviar solo cambios respecto al estado previo. |
| ImplicitlyAnimatedWidget | Widget que anima propiedades sin manejar `AnimationController`. |
| Precomposición | Preparar animaciones/recursos para reducir trabajo en runtime. |

---

## Referencias

- [Flutter Performance Profiling](https://docs.flutter.dev/perf/ui-performance)
- [Animations Best Practices](https://docs.flutter.dev/ui/animations/overview)
- [Rive Performance Tips](https://help.rive.app/runtimes/performance)
