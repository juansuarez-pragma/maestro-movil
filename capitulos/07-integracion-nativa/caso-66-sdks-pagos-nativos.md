# Caso 66: SDKs de Pagos Nativos con Platform Views
## Integrar WebViews/Views Nativos sin Romper la UX

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | sdk pagos, platform views, webview, nativo, UX |
| **Patrón Técnico** | Platform Views, Hybrid Composition, Safe Area Handling |
| **Stack Seleccionado** | Flutter + Platform Views (Hybrid/Virtual Display) + payment SDK nativo + Riverpod |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como app, necesito usar SDKs de pagos nativos con vistas embebidas sin jank ni bugs de layout."*

Platform Views pueden causar problemas de performance y compatibilidad si no se configuran bien.

### Evidencia de Industria

- **SDKs de pagos KYC:** Usan WebView/nativo; requieren integración cuidadosa.
- **Flutter perf:** Hybrid composition tiene trade-offs en rendimiento/compatibilidad.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Jank, glitches en scroll/gestos |
| **Técnico** | Problemas de z-order, lag en Android viejos, issues con teclado/semantics |
| **Compliance** | SDKs de pagos requieren flows completos sin fallos |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Embeder WebView sin ajustes ni pruebas de performance | **INADECUADO:** Jank y bugs de layout. |
| **ACEPTABLE** | PlatformView por defecto | **MEJORA:** Funciona, pero sin optimizar composición ni manejo de focus. |
| **ENTERPRISE** | **Platform Views optimizadas:** elegir hybrid/virtual según caso, aislar en pantallas dedicadas, manejar teclado/focus, medir perf, fallback si device no soporta | **ÓPTIMO:** UX estable con SDKs nativos/pagos. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Usar hybrid composition para compatibilidad, virtual display para mejor perf en dispositivos soportados. Aislar Platform Views en pantallas modales. Manejar z-order y focus/teclado. Limitar animaciones detrás de views. |
| **Restricciones Duras (NO permite)** | **Android viejos:** Hybrid composition puede jankear. **Gestos superpuestos:** Requiere manejo de hit testing. **A11y:** Debe integrarse con Semantics. |
| **Criterio de Selección** | Elegir composición según dispositivo/SO; encapsular vistas; pruebas en dispositivos representativos; fallback a WebView nativa si necesario. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Platform View | Vista nativa embebida en Flutter. |
| Hybrid Composition | Estrategia que compone views nativas sobre Flutter; más compatible. |
| Virtual Display | Renderiza view en textura; mejor perf en algunos dispositivos. |
| Z-order | Orden de apilamiento de vistas. |
| Semantics | Información de accesibilidad para lectores de pantalla. |

---

## Referencias

- [Platform Views](https://docs.flutter.dev/development/platform-integration/platform-views)
- [Hybrid Composition vs Virtual Display](https://docs.flutter.dev/development/platform-integration/android/platform-views#performance)
