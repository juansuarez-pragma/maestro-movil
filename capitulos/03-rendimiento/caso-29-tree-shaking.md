# Caso 29: Tree Shaking Agresivo
## Reducir 40% del Bundle Size en Apps Enterprise

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | bundle size, tree shaking, rendimiento, tamaño de app |
| **Patrón Técnico** | Code Splitting, Dead Code Elimination, Asset Optimization |
| **Stack Seleccionado** | Flutter AOT + Dart tree shaking + Deferred imports + ProGuard/R8 para nativo |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero que la app se instale rápido y no ocupe espacio excesivo."*

Bundles grandes aumentan abandono en descargas, afectan cold start y consumen almacenamiento, crucial en mercados con dispositivos de gama media/baja.

### Evidencia de Industria

- **Mercados emergentes:** Tasa de abandono crece con apps pesadas; apps livianas mejoran adopción.
- **Flutter perf:** Tree shaking y deferred loading reducen tamaño efectivo.

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
| **ENTERPRISE** | **Tree shaking + carga diferida:** deferred imports para features pesadas, eliminación de código muerto, assets optimizados/condicionales, R8/ProGuard configurados | **ÓPTIMO:** Bundle pequeño, arranque más rápido, menos uso de almacenamiento. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Separar features poco usadas en deferred imports. Optimizar assets (webp, sprites, fonts recortadas). Configurar R8/ProGuard para remover nativo no usado. Revisar dependencias para remover libs pesadas. |
| **Restricciones Duras (NO permite)** | **Code splitting limitado:** Flutter tiene soporte parcial; cuidado con UX en cargas diferidas. **Recursos obligatorios:** Algunas libs llevan nativo inevitable. **Riesgo de romper reflexión:** Tree shaking puede eliminar clases usadas por reflexión si no se configuran keep rules. |
| **Criterio de Selección** | Deferred para módulos no críticos, AOT con split per ABI, assets optimizados, keep rules cuidadosas para reflexión. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Tree Shaking | Eliminación de código no utilizado en compilación. |
| Deferred Import | Cargar un módulo Dart bajo demanda. |
| Split per ABI | Generar APKs/artefactos por arquitectura para reducir tamaño. |
| R8/ProGuard | Herramientas de minificación/obfuscación para código nativo. |
| Asset Optimization | Reducir tamaño y formatos de recursos (imágenes, fuentes). |

---

## Referencias

- [Flutter App Size Reduction](https://docs.flutter.dev/perf/app-size)
- [Deferred Loading in Dart](https://dart.dev/guides/libraries/deferred-loading)
- [R8/ProGuard Rules](https://developer.android.com/studio/build/shrink-code)
