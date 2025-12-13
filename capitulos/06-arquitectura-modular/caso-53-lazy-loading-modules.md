# Caso 53: El Módulo que Pesaba 40MB
## Lazy Loading de Features por Demanda

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | lazy loading, módulos, tamaño de app, on-demand |
| **Patrón Técnico** | [Dynamic Feature](#term-dynamic-feature "Módulo descargable de Android App Bundle.") Loading, Code Splitting, Deferred Components |
| **Stack Seleccionado** | Flutter + deferred imports + Android Dynamic Features + Riverpod para gating |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- El bundle principal incluye features raras (KYC/video) que inflan 40MB el binario.
- Instalaciones fallan en redes móviles; el arranque es más lento.
- Sin gating/precarga, la descarga on-demand llega tarde y la UX se rompe.

### Escenario de Negocio

> *"Como equipo, quiero reducir el tamaño inicial descargando features pesadas solo cuando se usan."*

### Incidentes reportados
- **Apps con KYC/video:** Migraron a módulos on-demand para reducir tamaño; sin precarga, los usuarios abandonaron el flujo.
- **App bundles:** Proyectos que no separaron assets/ABIs vieron tasas de instalación más bajas.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Android App Bundle adoption | Global | Dynamic Feature reduce tamaño inicial y mejora conversión de instalación. |
| Estudio de abandono | Global | Descargas tardías en flujos críticos elevan drop-off. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; tamaño/latencia es hallazgo frecuente. |

**Resumen global**
- Separar módulos pesados y precargar con señales mejora instalación y arranque; requiere fallback claro y límites por plataforma.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Descarga tardía frustra y aumenta abandono |
| **Técnico** | Errores de carga en red pobre; acoplamiento si no se diseña el gating |
| **Operacional** | Tamaño/latencia afectan KPIs de instalación y retención |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Todo en el bundle principal | **INADECUADO:** Tamaño grande, arranque más lento. |
| **ACEPTABLE** | Eliminar assets/libros no usados | **MEJORA:** Reduce algo, pero no evita módulos pesados en core. |
| **ENTERPRISE** | **Carga diferida:** deferred imports + Dynamic Features (Android) / segmentar assets, gating y precarga condicional | **ÓPTIMO:** Tamaño inicial menor, arranque rápido, flexibilidad. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Separar features pesadas (KYC, video) en módulos on-demand. Precargar según señales (navegación, elegibilidad). Mostrar progreso y fallback web/light. Reducir tamaño por ABI (split). |
| **Restricciones Duras (NO permite)** | **iOS:** Sin Dynamic Feature; se limita a assets incluidos al build. **Red pobre:** Descargas pueden fallar; requiere reintentos o fallback. **Reflexión:** Deferred rompe si hay reflexión/dynamic imports complejos. |
| **Criterio de Selección** | Deferred para código Dart; Dynamic Feature para nativo/recursos; gating via Riverpod/flags; precarga cuando hay buena red y el usuario es elegible. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Integration (CI) | Carga/descarga de módulos y fallback en error | Móvil/QA |
| Performance | Tamaño de bundle y tiempo de arranque vs baseline | QA/Perf |
| Observabilidad | Eventos `module.load` con latencia/errores | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Prefetch | Descargar cuando hay WiFi/carga y alta probabilidad de uso | Evita espera |
| Progreso | Indicadores claros y opción de reintentar/cancelar | Transparencia |
| Fallback | Versión ligera/web si falla la descarga | Continuidad |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Límite de tamaño | Budget por módulo y por app | Controla bloat |
| Publicación | Checklist de módulos y splits por ABI | Consistencia |
| Cache | Reutilizar módulos descargados y limpiar los obsoletos | Recursos |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Bundle inflado por features raras; arranque lento y menor conversión. |
| Opciones evaluadas | Todo en core; optimizar assets; carga diferida con precarga y fallback. |
| Decisión | Carga diferida con señales de precarga, budgets de tamaño y fallback seguro. |
| Consecuencias | Mayor complejidad de build y gating; requiere telemetría. |
| Riesgos aceptados | Latencia inicial si el prefetch falla; limitaciones en iOS. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Tamaño inicial del app | ↓ vs baseline (MB) | Alerta si sube | Instalación mejora |
| Tiempo de arranque | p95 menor que baseline | Warning si empeora | UX rápida |
| Tasa de abandono en flujos on-demand | ↓ tras precarga | Alerta si no baja | Conversión |
| Errores de carga de módulo | Tendencia a la baja | Crítico si crece | Estabilidad |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-deferred-import"></a>Deferred Import | Carga de librería Dart bajo demanda. |
| <a id="term-dynamic-feature"></a>Dynamic Feature | Módulo descargable de Android App Bundle. |
| <a id="term-gate-flag"></a>Gate/Flag | Condición que habilita la carga de un módulo. |
| <a id="term-preload"></a>Preload | Descargar anticipadamente basado en señal de uso probable. |
| <a id="term-split-per-abi"></a>Split per ABI | Generar artefactos por arquitectura para reducir tamaño. |

---

## Referencias

- [Deferred Components](https://docs.flutter.dev/development/ui/advanced/deferred-components)
- [Android Dynamic Features](https://developer.android.com/guide/app-bundle/dynamic-delivery)
- [Google Play Asset Delivery](https://developer.android.com/guide/playcore/asset-delivery)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
