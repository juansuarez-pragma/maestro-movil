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

### Problema detectado (técnico)
- Animaciones + ticks de mercado simultáneos exceden el presupuesto de frame (16ms) → jank y toques perdidos.
- setState masivo sin diffs/throttle dispara recomposición global; parsing en UI bloquea rendering.
- Assets no optimizados (Rive/Lottie) y sombras múltiples saturan GPU.

### Escenario de Negocio

> *"Como trader, necesito ver gráficas y animaciones fluidas a 60fps mientras ejecuto órdenes."*

### Incidentes reportados
- **Trading apps:** Jank reduce conversión/confianza; mitigado con throttle y capas separadas.
- **Flutter perf:** Minimizar trabajo en UI thread y recomposición para sostener 60fps.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Apps de trading | Global | Jank por animaciones + datos; requieren throttle/diff. |
| Flutter perf estudios | Global | RepaintBoundary y isolates reducen jank. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; perf/animación es foco de hallazgos. |

**Resumen global**
- Mantener 60fps requiere separar capas, throttle y optimizar assets.
- Jank sostenido impacta decisiones de trading y churn.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Performance (Perfetto/DevTools) | Frame time < 16ms; jank bajo | QA/Perf |
| Integration (CI) | Throttle/diff de ticks evita recomposición masiva | Móvil/Backend, CI |
| Seguridad/consistencia | RepaintBoundary/isolates no rompen UI; assets optimizados | QA |
| Observabilidad | Métricas `ui.jank`, `frame_time_p95`, FPS | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Capas | Separar capa estática (cache) y dinámica (ticks/overlays) | Reduce trabajo por frame |
| Throttle/diff | Actualizar datos a 15–20 fps; animaciones a 60fps | Balance frescura/CPU |
| Assets | Optimizar Rive/Lottie; limitar sombras y blurs | Mantener presupuesto de frame |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Isolates | Parsing de datos pesados fuera del UI thread | Previene jank |
| Backpressure | Drop/latest para feeds secundarios | Ahorra CPU/red |
| Medición continua | Alertas si `ui.jank` supera umbral | Control continuo |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Jank por animaciones + ticks simultáneos. |
| Opciones evaluadas | setState masivo; throttle básico; presupuestar frame con capas, throttle, isolates. |
| Decisión | Capas separadas + throttle/diff + RepaintBoundary + isolates + assets optimizados. |
| Consecuencias | Más tuning de perf; depende de medición continua. |
| Riesgos aceptados | GPU-bound en dispositivos low-end; assets mal optimizados pueden romper metas. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| FPS/jank | 60fps sostenido; jank mínimo | Alerta si `ui.jank` sube | UX fluida |
| Frame time p95 | < 16 ms | Warning si se acerca | Presupuesto cumplido |
| Latencia de ticks | p95 < 200 ms tras throttle | Alerta si sube | Datos útiles |
| Consumo CPU/GPU | Dentro de límites de dispositivo | Alerta si se dispara | Batería y perf |
| Tickets por “app lenta” | ↓ vs baseline | Alerta si no baja | Confianza |

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
| RepaintBoundary | Aislar repaints a subárboles específicos. |

---

## Referencias

- [Flutter Performance Profiling](https://docs.flutter.dev/perf/ui-performance)
- [Animations Best Practices](https://docs.flutter.dev/ui/animations/overview)
- [Rive Performance Tips](https://help.rive.app/runtimes/performance)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
