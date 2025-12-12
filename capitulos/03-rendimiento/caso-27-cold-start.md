# Caso 27: Cold Start de 8 Segundos
## Optimizar Tiempo de Arranque en Apps Bancarias

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | cold start, tiempo de arranque, rendimiento, splash |
| **Patrón Técnico** | Startup Profiling, Lazy Initialization, Deferred Loading |
| **Stack Seleccionado** | Flutter + Dart Deferred Imports + AOT optimizations + Splash ligero |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, la app debe abrir en <2s; no quiero esperar para ver mi saldo."*

Arranques lentos elevan la tasa de abandono y tickets de soporte. Cargas pesadas en main() y assets grandes retrasan el primer frame.

### Evidencia de Industria

- **Estudios móviles:** Cada segundo extra en arranque reduce retención.
- **Flutter perf:** Recomienda mover trabajo fuera de main isolate al inicio, minimizar trabajo sincronizado.

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
| **Capacidades (SÍ permite)** | Mover init pesado a futuros post-frame. Usar deferred imports para features no críticas. Reducir tamaño de assets (webp, LQIP). Precache selectivo. Medir con `flutter run --trace-startup` y DevTools. |
| **Restricciones Duras (NO permite)** | **Dependencias críticas:** Auth/config mínimas deben cargarse para home. **Dispositivos low-end:** AOT y compresión ayudan pero hardware limita. **Assets enormes:** Requieren optimización externa. |
| **Criterio de Selección** | Inicio "just enough": config + routing + base providers; todo lo demás diferido. Medición continua del tiempo a primer frame útil. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Cold Start | Inicio desde cero, sin procesos en memoria. |
| Deferred Import | Carga diferida de librerías Dart bajo demanda. |
| LQIP | Low Quality Image Placeholder para reducir tiempo de carga de assets. |
| AOT | Ahead-of-Time compilation para startups más rápidos. |
| TTI/TTFF | Time To Interactive / Time To First Frame. |

---

## Referencias

- [Flutter Startup Profiling](https://docs.flutter.dev/perf/app-size#measure-and-optimize-app-startup)
- [Deferred Components/Imports](https://dart.dev/guides/libraries/deferred-loading)
- [Image Optimization](https://web.dev/fast/#images)
