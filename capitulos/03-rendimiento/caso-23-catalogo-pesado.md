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

### Escenario de Negocio

> *"Como comprador, quiero navegar catálogos con miles de imágenes sin que se sienta lento ni consuma mis datos."*

Imágenes pesadas saturan la red y bloquean el scroll. Sin caching ni placeholders, la UX se degrada y el consumo de datos explota.

### Evidencia de Industria

- **Retail apps:** Implementan placeholders (blurhash) y múltiples resoluciones para mejorar LCP.
- **Google Web Vitals:** Imágenes optimizadas reducen abandono en listados grandes.

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

---

## Referencias

- [cached_network_image](https://pub.dev/packages/cached_network_image)
- [Blurhash](https://blurha.sh/)
- [Image Optimization Best Practices](https://web.dev/fast/#images)
