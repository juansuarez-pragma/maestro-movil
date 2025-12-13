# Caso 66: SDKs de Pagos Nativos con Platform Views
## Integrar WebViews/Views Nativos sin Romper la UX

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | sdk pagos, platform views, webview, nativo, UX |
| **Patrón Técnico** | Platform Views, [Hybrid Composition](#term-hybrid-composition "Estrategia que compone views nativas sobre Flutter; más compatible."), Safe Area Handling |
| **Stack Seleccionado** | Flutter + Platform Views (Hybrid/[Virtual Display](#term-virtual-display "Renderiza view en textura; mejor perf en algunos dispositivos.")) + payment SDK nativo + Riverpod |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Platform Views mal configuradas provocan jank, z-order issues y fallos de teclado/gestos.
- SDKs de pagos en WebView/nativo fallan en dispositivos viejos si no se elige composición correcta.
- Sin pruebas de perf/A11y se rompen flujos de checkout y accesibilidad.

### Escenario de Negocio

> *"Como app, necesito usar SDKs de pagos nativos con vistas embebidas sin jank ni bugs de layout."*

### Incidentes reportados
- **SDKs de pagos/KYC:** Glitches y bloqueos por hybrid composition en Android viejos; lag en scroll interrumpió checkouts.
- **A11y/teclado:** Usuarios no pudieron completar formularios por focus y z-order incorrectos.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| SDKs de pagos en móvil | Global | Requieren WebView/PlatformView con compatibilidad amplia. |
| Flutter perf docs | Global | Hybrid vs virtual display con trade-offs de perf/compatibilidad. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; integración de vistas nativas es fuente de bugs. |

**Resumen global**
- Elegir composición por dispositivo y aislar vistas nativas con pruebas de perf/A11y evita jank y fallos en checkout.

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
| **Restricciones Duras (NO permite)** | **Android viejos:** Hybrid composition puede jankear. **Gestos superpuestos:** Requiere manejo de hit testing. **A11y:** Debe integrarse con [Semantics](#term-semantics "Información de accesibilidad para lectores de pantalla."). |
| **Criterio de Selección** | Elegir composición según dispositivo/SO; encapsular vistas; pruebas en dispositivos representativos; fallback a WebView nativa si necesario. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Performance | FPS, jank y tiempos de render | QA/Perf |
| Integration (Android/iOS) | Focus/teclado, gestos, z-order correctos | Móvil/QA |
| A11y | Semantics y lector de pantalla operan | QA/Accesibilidad |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Aislamiento | Pantallas dedicadas para Platform Views críticas | Menos interferencia |
| Fallback | Cambiar composición o WebView nativa en devices problemáticos | Disponibilidad |
| Mensajes | Indicadores de carga y manejo de errores claros en checkout | Confianza |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Compatibilidad | Lista de dispositivos vetados o forzados a hybrid | Control de riesgo |
| Observabilidad | Eventos `pv.*` con perf/errores | Trazabilidad |
| Seguridad/compliance | Flujos completos del SDK de pagos sin cortes | Cumplimiento |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Embebido de SDKs de pagos con Platform Views provoca jank/bugs de layout. |
| Opciones evaluadas | WebView simple; PlatformView por defecto; Platform Views optimizadas con composición por dispositivo y aislamientos. |
| Decisión | Platform Views optimizadas (hybrid/virtual según device), aislamiento de pantalla, manejo de focus/A11y y fallback. |
| Consecuencias | Matriz de compatibilidad y pruebas de perf; mantenimiento de políticas por device. |
| Riesgos aceptados | Variabilidad en dispositivos; overhead de pruebas de perf. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Tiempos de render/FPS en checkout | p95 estable | Warning si jank > umbral | UX fluida |
| Errores de focus/teclado | 0 | Crítico si >0 | Accesibilidad |
| Abandono en checkout | ↓ vs baseline | Alerta si no baja | Conversión |
| Incidentes por dispositivo | Tendencia a la baja | Crítico si sube | Compatibilidad |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-platform-view"></a>Platform View | Vista nativa embebida en Flutter. |
| <a id="term-hybrid-composition"></a>Hybrid Composition | Estrategia que compone views nativas sobre Flutter; más compatible. |
| <a id="term-virtual-display"></a>Virtual Display | Renderiza view en textura; mejor perf en algunos dispositivos. |
| <a id="term-z-order"></a>Z-order | Orden de apilamiento de vistas. |
| <a id="term-semantics"></a>Semantics | Información de accesibilidad para lectores de pantalla. |

---

## Referencias

- [Platform Views](https://docs.flutter.dev/development/platform-integration/platform-views)
- [Hybrid Composition vs Virtual Display](https://docs.flutter.dev/development/platform-integration/android/platform-views#performance)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
