# Caso 28: El Shader Compilation Jank
## Pre-warming de Shaders en Producción

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | shader jank, prewarming, animaciones, rendimiento |
| **Patrón Técnico** | Shader Precompilation, Warmup, Pipeline Caching |
| **Stack Seleccionado** | Flutter + `ShaderWarmUp`/`SkSL warmup` + `impeller` (cuando aplique) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, las animaciones no deben saltar la primera vez que se muestran."*

La compilación de shaders en runtime causa jank en la primera renderización de animaciones/efectos.

### Evidencia de Industria

- **Flutter perf:** Recomienda capturar y precargar SkSL para evitar jank en primera ejecución.
- **Apps con animaciones pesadas:** Mejoran al pre-warm shaders en splash.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Animaciones con saltos iniciales, percepción de mala calidad |
| **Técnico** | Trabajo extra en GPU/CPU en runtime |
| **Reputacional** | Menor confianza en apps financieras con UI deficiente |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | No hacer nada; aceptar jank inicial | **INADECUADO:** UX pobre. |
| **ACEPTABLE** | Capturar SkSL en debug y empaquetar | **MEJORA:** Reduce jank, pero depende de rutas ejercitadas en captura. |
| **ENTERPRISE** | **Pre-warm estructurado:** `ShaderWarmUp`, SkSL capturado en staging con rutas críticas, carga en splash, monitoreo en prod; usar Impeller cuando disponible | **ÓPTIMO:** Minimiza jank inicial en animaciones críticas. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Capturar SkSL en dispositivos representativos, empaquetar en assets y precargar en `runApp`. Usar `ShaderWarmUp` para pre-renderizar paths críticos. Reducir first-frame jank en animaciones intensas. |
| **Restricciones Duras (NO permite)** | **Cobertura incompleta:** SkSL depende de rutas ejercitadas; nuevas animaciones no cubiertas. **Tamaño de assets:** SkSL puede incrementar tamaño. **Compatibilidad:** Impeller aún en progreso en algunas plataformas. |
| **Criterio de Selección** | Pre-warm en splash para animaciones críticas; capturar SkSL en staging; habilitar Impeller donde esté maduro. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Shader Jank | Saltos en animaciones por compilación de shaders en runtime. |
| SkSL | Shading Language de Skia, se puede capturar y reutilizar. |
| ShaderWarmUp | Hook para renderizar escenas y precalentar shaders. |
| Impeller | Nuevo render engine de Flutter que reduce jank de shaders. |
| Pipeline Cache | Cache de pipelines de render para reutilizar compilaciones. |

---

## Referencias

- [Flutter SkSL Warmup](https://docs.flutter.dev/perf/rendering/shader)
- [Impeller Overview](https://docs.flutter.dev/perf/impeller)
