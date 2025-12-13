# Caso 28: El Shader Compilation Jank
## Pre-warming de Shaders en Producción

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | shader jank, prewarming, animaciones, rendimiento |
| **Patrón Técnico** | Shader Precompilation, Warmup, Pipeline Caching |
| **Stack Seleccionado** | Flutter + `[ShaderWarmUp](#term-shaderwarmup "Hook para renderizar escenas y precalentar shaders.")`/`[SkSL](#term-sksl "Shading Language de Skia, se puede capturar y reutilizar.") warmup` + `impeller` (cuando aplique) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Compilación de shaders en runtime causa jank la primera vez que se renderizan animaciones/efectos.
- Sin capturar/precalentar SkSL, cada dispositivo recompila y genera saltos iniciales.
- Tamaño extra de SkSL/compatibilidad puede afectar tamaño y cobertura de rutas.

### Escenario de Negocio

> *"Como usuario, las animaciones no deben saltar la primera vez que se muestran."*

### Incidentes reportados
- **Flutter perf:** Recomienda capturar y precargar SkSL para evitar jank inicial.
- **Apps con animaciones pesadas:** Mejoran al pre-warm shaders en splash.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Flutter perf guías | Global | SkSL warmup reduce jank inicial. |
| Apps con animaciones | Global | Precalentar shaders mejora la primera ejecución. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; perf/gráficos es foco de hallazgos. |

**Resumen global**
- Shader warmup es práctica recomendada para evitar jank inicial en animaciones críticas.

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
| **ENTERPRISE** | **Pre-warm estructurado:** `ShaderWarmUp`, SkSL capturado en staging con rutas críticas, carga en splash, monitoreo en prod; usar [Impeller](#term-impeller "Nuevo render engine de Flutter que reduce jank de shaders.") cuando disponible | **ÓPTIMO:** Minimiza jank inicial en animaciones críticas. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Capturar SkSL en dispositivos representativos, empaquetar en assets y precargar en `runApp`. Usar `ShaderWarmUp` para pre-renderizar paths críticos. Reducir first-frame jank en animaciones intensas. |
| **Restricciones Duras (NO permite)** | **Cobertura incompleta:** SkSL depende de rutas ejercitadas; nuevas animaciones no cubiertas. **Tamaño de assets:** SkSL puede incrementar tamaño. **Compatibilidad:** Impeller aún en progreso en algunas plataformas. |
| **Criterio de Selección** | Pre-warm en splash para animaciones críticas; capturar SkSL en staging; habilitar Impeller donde esté maduro. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Performance | Reducir jank en primera ejecución de animaciones | QA/Perf |
| Integration (CI) | SkSL/ShaderWarmUp se carga en splash y cubre rutas críticas | Móvil/QA |
| Observabilidad | Métricas `ui.jank_first_play` | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Rutas de captura | Ejercitar animaciones críticas en staging para capturar SkSL | Cobertura |
| Tamaño | Controlar tamaño de SkSL; revisar impacto en bundle | Balance |
| Compatibilidad | Probar en GPUs/OS objetivo; usar Impeller cuando esté estable | Estabilidad |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Actualizaciones | Re-capturar SkSL cuando cambian animaciones | Coherencia |
| Fallback | Si SkSL no aplica, usar ShaderWarmUp custom | Mitigación |
| Tamaño de assets | Limitar crecimiento; monitorear bundle size | Control de tamaño |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Jank inicial por compilación de shaders en runtime. |
| Opciones evaluadas | Sin warmup; SkSL capturado; ShaderWarmUp + SkSL + Impeller. |
| Decisión | Captura SkSL + ShaderWarmUp en splash; usar Impeller donde aplique. |
| Consecuencias | Tamaño extra en assets; requiere pipeline de captura. |
| Riesgos aceptados | Cobertura parcial de rutas; compatibilidad dependiente de GPU/OS. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Jank en primera animación | Mínimo | Alerta si jank sube | UX suave |
| Cobertura de warmup | Rutas críticas cubiertas | Alerta si faltan | Calidad consistente |
| Tamaño de assets SkSL | Controlado | Alerta si crece | Bundle razonable |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-shader-jank"></a>Shader Jank | Saltos en animaciones por compilación de shaders en runtime. |
| <a id="term-sksl"></a>SkSL | Shading Language de Skia, se puede capturar y reutilizar. |
| <a id="term-shaderwarmup"></a>ShaderWarmUp | Hook para renderizar escenas y precalentar shaders. |
| <a id="term-impeller"></a>Impeller | Nuevo render engine de Flutter que reduce jank de shaders. |
| <a id="term-pipeline-cache"></a>Pipeline Cache | Cache de pipelines de render para reutilizar compilaciones. |
| <a id="term-sksl"></a>SkSL | Shading Language de Skia capturable para warmup. |

---

## Referencias

- [Flutter SkSL Warmup](https://docs.flutter.dev/perf/rendering/shader)
- [Impeller Overview](https://docs.flutter.dev/perf/impeller)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
