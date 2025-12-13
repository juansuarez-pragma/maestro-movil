# Caso 29: [Tree Shaking](#term-tree-shaking "Eliminación de código no utilizado en compilación.") Agresivo
## Reducir 40% del Bundle Size en Apps Enterprise

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | bundle size, tree shaking, rendimiento, tamaño de app |
| **Patrón Técnico** | Code Splitting, Dead Code Elimination, [Asset Optimization](#term-asset-optimization "Reducir tamaño y formatos de recursos (imágenes, fuentes).") |
| **Stack Seleccionado** | Flutter AOT + Dart tree shaking + Deferred imports + ProGuard/R8 para nativo |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Bundle grande (código + assets) aumenta abandono en descarga y empeora cold start.
- Sin tree shaking/deferred/loading condicional, se incluye código/recursos no usados.
- Falta de split per ABI y keep rules correctas desperdicia espacio y puede romper reflexión.

### Escenario de Negocio

> *"Como usuario, quiero que la app se instale rápido y no ocupe espacio excesivo."*

### Incidentes reportados
- **Mercados emergentes:** Abandono crece con apps pesadas; apps livianas mejoran adopción.
- **Flutter perf:** Tree shaking y deferred loading reducen tamaño y mejoran arranque.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Mercado emergente (reportes varios) | LATAM/APAC | Apps pesadas incrementan abandono de instalación. |
| Flutter app size studies | Global | Tree shaking/deferred reduce tamaño efectivo y cold start. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; tamaño/arranque son hallazgos frecuentes. |

**Resumen global**
- Reducir bundle mejora adopción y arranque; tree shaking/deferred/split ABI son prácticas clave.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Descargas lentas, falta de espacio, desinstalaciones |
| **Técnico** | Cold start más lento, mayor consumo de memoria |
| **Reputacional/Económico** | Menor adopción en mercados clave |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Incluir todas las libs y assets en el main bundle | **INADECUADO:** Tamaño excesivo, assets no usados. |
| **ACEPTABLE** | Remover assets no usados, minificar nativo | **MEJORA:** Reduce algo, pero no separa módulos. |
| **ENTERPRISE** | **Tree shaking + carga diferida:** deferred imports para features pesadas, eliminación de código muerto, assets optimizados/condicionales, [R8/ProGuard](#term-r8-proguard "Herramientas de minificación/obfuscación para código nativo.") configurados | **ÓPTIMO:** Bundle pequeño, arranque más rápido, menos uso de almacenamiento. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Separar features poco usadas en deferred imports. Optimizar assets (webp, sprites, fonts recortadas). Configurar R8/ProGuard para remover nativo no usado. Revisar dependencias para remover libs pesadas. |
| **Restricciones Duras (NO permite)** | **Code splitting limitado:** Flutter tiene soporte parcial; cuidado con UX en cargas diferidas. **Recursos obligatorios:** Algunas libs llevan nativo inevitable. **Riesgo de romper reflexión:** Tree shaking puede eliminar clases usadas por reflexión si no se configuran keep rules. |
| **Criterio de Selección** | Deferred para módulos no críticos, AOT con split per ABI, assets optimizados, keep rules cuidadosas para reflexión. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| App size report | Reducción del bundle/base APK y por ABI | Equipo móvil |
| Startup perf | Mejora en TTFF/TTI tras optimizaciones | QA/Perf |
| Integration (CI) | Módulos deferred cargan on-demand sin romper flujo | Móvil/QA |
| Seguridad | Keep rules no exponen/reflejan más de lo necesario | Seguridad |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Carga diferida | Módulos pesados cargados on-demand con feedback | UX consistente |
| Assets | WebP/AVIF, fonts subset, sprites | Menos peso |
| [Split per ABI](#term-split-per-abi "Generar APKs/artefactos por arquitectura para reducir tamaño.") | Distribuir por arquitectura | Descarga menor |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Dependencias | Auditar libs pesadas; remover no usadas | Tamaño controlado |
| Reflexión | Configurar keep rules para frameworks con reflexión | Evita crashes |
| Monitoreo | Vigilar tamaño en cada release | Previene regresiones |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Bundle grande reduce adopción y arranque. |
| Opciones evaluadas | Sin optimizar; minificar assets; tree shaking + deferred + split ABI. |
| Decisión | Tree shaking + deferred imports + assets optimizados + split ABI + keep rules. |
| Consecuencias | Más pasos de build/release; riesgo de romper reflexión si se omiten keep rules. |
| Riesgos aceptados | Soporte limitado de code splitting; UX de carga diferida debe ser clara. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Tamaño de app/base APK | Reducción ≥ objetivo (p.ej., -20-40%) | Alerta si crece | Mejor adopción |
| TTFF/TTI | Mejora tras reducción de bundle | Warning si no mejora | Arranque más rápido |
| Tamaño por ABI | Distribución separada (split) | Alerta si se entrega universal pesada | Menor descarga |
| Cargas diferidas | Sin bloqueos; feedback visible | Alerta si causa UX mala | Flujo estable |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-tree-shaking"></a>Tree Shaking | Eliminación de código no utilizado en compilación. |
| <a id="term-deferred-import"></a>Deferred Import | Cargar un módulo Dart bajo demanda. |
| <a id="term-split-per-abi"></a>Split per ABI | Generar APKs/artefactos por arquitectura para reducir tamaño. |
| <a id="term-r8-proguard"></a>R8/ProGuard | Herramientas de minificación/obfuscación para código nativo. |
| <a id="term-asset-optimization"></a>Asset Optimization | Reducir tamaño y formatos de recursos (imágenes, fuentes). |
| <a id="term-keep-rules"></a>Keep Rules | Configuración para preservar clases/métodos usados por reflexión. |

---

## Referencias

- [Flutter App Size Reduction](https://docs.flutter.dev/perf/app-size)
- [Deferred Loading in Dart](https://dart.dev/guides/libraries/deferred-loading)
- [R8/ProGuard Rules](https://developer.android.com/studio/build/shrink-code)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
