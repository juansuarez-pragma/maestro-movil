# Caso 27: [Cold Start](#term-cold-start "Inicio desde cero, sin procesos en memoria.") de 8 Segundos
## Optimizar Tiempo de Arranque en Apps Bancarias

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | cold start, tiempo de arranque, rendimiento, splash |
| **Patrón Técnico** | Startup Profiling, Lazy Initialization, Deferred Loading |
| **Stack Seleccionado** | Flutter + Dart Deferred Imports + [AOT](#term-aot "Ahead-of-Time compilation para startups más rápidos.") optimizations + Splash ligero |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Inicializar todo en `main()` y cargar assets pesados retrasa el primer frame y eleva el cold start a 8s.
- Sin diferir módulos no críticos ni optimizar assets, [TTI/TTFF](#term-tti-ttff "Time To Interactive / Time To First Frame.") exceden la meta (<2s).
- Falta de medición (trace-startup, DevTools) oculta los puntos calientes de arranque.

### Escenario de Negocio

> *"Como usuario, la app debe abrir en <2s; no quiero esperar para ver mi saldo."*

### Incidentes reportados
- Arranques lentos elevan abandono y tickets de soporte.
- Flutter perf: mover trabajo fuera de `main` y minimizar sincronía en startup.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Estudios móviles | Global | Cada segundo extra en arranque reduce retención. |
| Flutter perf guías | Global | deferred loading y minimizar trabajo sincronizado reducen TTFF. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; arranque lento es hallazgo frecuente. |

**Resumen global**
- Cold start lento reduce retención y NPS; deferred/loading mínimo es clave.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Abandono en splash, mala primera impresión |
| **Técnico** | Bloqueo por inicializaciones pesadas y descompresión de assets |
| **Reputacional/Económico** | Menor NPS, menos conversión en banca/e-commerce |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Inicializar todo en main() | **INADECUADO:** Bloquea el primer frame, frío extremo. |
| **ACEPTABLE** | Cargar solo servicios críticos, splash con progress | **MEJORA:** Reduce espera, pero aún carga módulos no usados. |
| **ENTERPRISE** | **Inicio mínimo + carga diferida:** solo core (config, analytics diferido), deferred imports para módulos pesados, assets optimizados, precache controlado | **ÓPTIMO:** Primer frame rápido, UX responsiva, menor ANR. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Mover init pesado a futuros post-frame. Usar deferred imports para features no críticas. Reducir tamaño de assets (webp, [LQIP](#term-lqip "Low Quality Image Placeholder para reducir tiempo de carga de assets.")). Precache selectivo. Medir con `flutter run --trace-startup` y DevTools. |
| **Restricciones Duras (NO permite)** | **Dependencias críticas:** Auth/config mínimas deben cargarse para home. **Dispositivos low-end:** AOT y compresión ayudan pero hardware limita. **Assets enormes:** Requieren optimización externa. |
| **Criterio de Selección** | Inicio "just enough": config + routing + base providers; todo lo demás diferido. Medición continua del tiempo a primer frame útil. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Startup profiling | TTFF/TTI < 2s en dispositivos target | QA/Perf |
| Integration (CI) | deferred imports cargan on-demand; core inicia solo lo crítico | Móvil/QA |
| Observabilidad | `trace-startup`, métricas de arranque en DevTools/APM | Móvil/SRE |
| Assets | Optimización (webp/LQIP) y precache selectivo verificados | Móvil/QA |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Splash ligero | Mostrar rápido y pasar al home ASAP | Mejor primera impresión |
| Deferral | Cargar módulos pesados después de primer frame | Reduce TTFF |
| Progresivo | Cargar datos en background con feedback mínimo | Percepción de rapidez |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Depuración | Medir en hardware real (low/mid-end) | Realismo |
| Compresión | AOT y app bundle split por ABI | Menor tamaño/arranque |
| Limits | Evitar trabajo sync en `main`; mover a post-frame | Menos bloqueos |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Cold start > 8s por inicialización pesada y assets sin optimizar. |
| Opciones evaluadas | Init completo en main; carga parcial; inicio mínimo + deferred + assets optimizados. |
| Decisión | Inicio mínimo + carga diferida + assets optimizados + medición continua. |
| Consecuencias | Requiere separar módulos y medir; riesgo de lazy-load mal orquestado. |
| Riesgos aceptados | Dispositivos low-end pueden seguir limitados; dependencia de medición continua. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| TTFF/TTI | < 2s en dispositivos target | Alerta si ≥ 2s | Mejor retención y NPS |
| Tamaño de assets iniciales | Reducido (webp/LQIP) | Alerta si crece | Arranque más rápido |
| Carga diferida | Módulos pesados post-arranque | Alerta si se cargan en main | Menos bloqueo |
| ANR/Timeout en arranque | 0 | Crítico si > 0 | Estabilidad |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-cold-start"></a>Cold Start | Inicio desde cero, sin procesos en memoria. |
| <a id="term-deferred-import"></a>Deferred Import | Carga diferida de librerías Dart bajo demanda. |
| <a id="term-lqip"></a>LQIP | Low Quality Image Placeholder para reducir tiempo de carga de assets. |
| <a id="term-aot"></a>AOT | Ahead-of-Time compilation para startups más rápidos. |
| <a id="term-tti-ttff"></a>TTI/TTFF | Time To Interactive / Time To First Frame. |

---

## Referencias

- [Flutter Startup Profiling](https://docs.flutter.dev/perf/app-size#measure-and-optimize-app-startup)
- [Deferred Components/Imports](https://dart.dev/guides/libraries/deferred-loading)
- [Image Optimization](https://web.dev/fast/#images)
