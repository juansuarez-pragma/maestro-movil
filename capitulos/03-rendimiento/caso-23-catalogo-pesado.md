# Caso 23: El Catálogo Pesado
## Lazy Loading de Imágenes en MercadoLibre Scale

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | catálogo, imágenes pesadas, lazy loading, caching, CDN |
| **Patrón Técnico** | Lazy Loading, Image Caching, Responsive Images |
| **Stack Seleccionado** | Flutter + `cached_network_image`/`flutter_blurhash` + CDN con imágenes multi-resolución |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Imágenes pesadas sin resize/caching saturan la red y bloquean scroll.
- Sin placeholders (blurhash) hay pop-in y LCP alto; sin multi-res, consumo de datos se dispara.
- Caches sin límites producen uso excesivo de memoria.

### Escenario de Negocio

> *"Como comprador, quiero navegar catálogos con miles de imágenes sin que se sienta lento ni consuma mis datos."*

### Incidentes reportados
- **Retail apps:** Implementan blurhash/variantes multi-res para mejorar LCP; sin ello, abandono aumenta.
- **Google Web Vitals:** Imágenes optimizadas reducen abandono en listados grandes.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Retail/Web Vitals | Global | Imágenes sin optimización elevan LCP y abandono. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; perf/carga de recursos es recurrente. |
| CASB/CDN reports | Ecommerce | Variantes multi-res reducen datos y mejoran percepción de velocidad. |

**Resumen global**
- Imágenes sin optimización degradan LCP/UX y elevan costos de datos/CDN.
- Blurhash/variantes multi-res y cache limitada son prácticas recomendadas.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Scroll janky, placeholders tardíos, pop-in de layout |
| **Económico** | Usuarios abandonan por carga lenta; mayor costo de CDN/datos |
| **Técnico** | Uso excesivo de memoria por caches mal configuradas |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Cargar imágenes originales sin cache ni resize | **INADECUADO:** Pesadas, tardan, consumen datos. |
| **ACEPTABLE** | `cached_network_image` con resize básico | **MEJORA:** Cache y resize, pero sin control de múltiples resoluciones ni placeholders rápidos. |
| **ENTERPRISE** | **Lazy + multi-res + placeholders:** CDN con variantes (1x/2x/3x), headers de cache, blurhash/low-res primero, prefetch en scroll, límites de cache | **ÓPTIMO:** UX fluida, menor consumo, memoria controlada. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Seleccionar URL según densidad de pantalla. Usar placeholders blurhash para evitar pop-in. Prefetch de próximas imágenes cuando el usuario se acerca. Limitar tamaño de cache y uso de memoria. Soportar retry con backoff en errores de red. |
| **Restricciones Duras (NO permite)** | **CDN sin variantes:** Si no hay multi-res, el ahorro es limitado. **Animaciones pesadas:** GIF/APNG grandes seguirán pesados; preferir videos/webp. **Red lenta:** UX depende de ancho de banda; placeholders mitigan pero no eliminan espera. |
| **Criterio de Selección** | `cached_network_image` por facilidad y cache integrada; blurhash para placeholders rápidos; CDN multi-res para optimizar ancho de banda; prefetch controlado para balancear consumo. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Selección de URL por densidad; placeholders no rompen layout | Equipo móvil, CI |
| Integration (CI) | Prefetch y cache limitan pop-in y uso de datos | Móvil/QA, CI |
| Performance | LCP percibido/scroll sin jank con catálogo grande | QA/Perf |
| Observabilidad | Eventos `image.load` con tamaño, densidad, hit/miss cache | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Placeholders | Blurhash/low-res mientras carga | Estabilidad visual |
| Variantes | 1x/2x/3x según DPI y tamaño render | Ahorro de datos |
| Cache | Límites de tamaño/TTL; limpiar en background | Control de memoria |
| Prefetch | Anticipar imágenes próximas al viewport | Scroll suave |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| CDN | Requerir variantes y headers de cache | Eficiencia |
| Formatos | Preferir WebP/AVIF sobre JPG pesados | Ahorro |
| Fallos de red | Retry con backoff; fallback a placeholder si falla | Resiliencia |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Imágenes pesadas y sin optimización causan jank y alto consumo. |
| Opciones evaluadas | Cargar originales; resize básico; multi-res + blurhash + cache controlada. |
| Decisión | Lazy + multi-res + blurhash + cache/TTL + prefetch. |
| Consecuencias | Requiere soporte de CDN y tuning de cache. |
| Riesgos aceptados | CDN sin variantes limita beneficio; depende de ancho de banda del usuario. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| LCP percibido en listados | ↓ vs baseline | Alerta si no mejora | UX más rápida |
| Consumo de datos | ↓ por uso de variantes y cache | Alerta si sube | Costos controlados |
| Jank/pop-in | Minimizado con placeholders/prefetch | Alerta si jank sube | Fluidez |
| Uso de memoria de cache | Dentro de límite configurado | Alerta si crece | Estabilidad |
| Tickets por “catálogo lento” | ↓ vs baseline | Alerta si no baja | Menos soporte |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Lazy Loading | Cargar recursos solo cuando están cerca de ser visibles. |
| Blurhash | Placeholder generado a partir de la imagen para mostrar previo a la carga completa. |
| Multi-resolución | Variantes de imagen por densidad (1x/2x/3x) servidas desde CDN. |
| Prefetch | Descargar recursos anticipadamente basados en el scroll. |
| Cache TTL | Tiempo que una imagen permanece válida antes de volver a descargarse. |
| LCP | Largest Contentful Paint; métrica de rendimiento percibido. |

---

## Referencias

- [cached_network_image](https://pub.dev/packages/cached_network_image)
- [Blurhash](https://blurha.sh/)
- [Image Optimization Best Practices](https://web.dev/fast/#images)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
