# Caso 61: Platform Channels sin Dolor
## Integrar SDKs Nativos (Stripe/Facetec) en Flutter

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | platform channels, sdk nativo, integración, kotlin/swift |
| **Patrón Técnico** | Platform Channels, Method/Event Channels, Bridge Pattern |
| **Stack Seleccionado** | Flutter + Kotlin/Swift + MethodChannel/EventChannel + Riverpod para estado |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo, necesitamos integrar SDKs nativos críticos (pagos/biometría) sin romper la app Flutter."*

Integraciones mal encapsuladas generan crashes, memory leaks y APIs frágiles.

### Evidencia de Industria

- **Integraciones de pagos/biometría:** Requieren flujos nativos y callbacks complejos.
- **Buenas prácticas:** Encapsular, tipar y manejar errores nativamente.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Crashes o flujos rotos en biometría/pagos |
| **Técnico** | APIs frágiles, incompatibilidades de plataforma |
| **Reputacional** | Pérdida de confianza si pagos fallan |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Llamadas directas sin encapsulación ni manejo de errores | **INADECUADO:** API frágil y propensa a crashes. |
| **ACEPTABLE** | Wrapper básico en Dart | **MEJORA:** Aísla algo, pero callbacks/eventos pueden ser inestables. |
| **ENTERPRISE** | **Bridge robusto:** API tipada, mapping de errores, lifecycle claro, pruebas en cada plataforma, canal de eventos para callbacks | **ÓPTIMO:** Integración estable y mantenible. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Definir API tipada (métodos y eventos) en un solo punto. Mapear errores nativos a códigos Dart. Manejar lifecycle (init/dispose). Probar canales con integration tests por plataforma. |
| **Restricciones Duras (NO permite)** | **API inestable de terceros:** Depende de SDK. **Versionado:** Cambios en SDK nativo requieren sincronizar Dart. **Threading:** Algunas APIs nativas deben llamarse en UI thread. |
| **Criterio de Selección** | Canal bien definido, wrappers nativos con manejo de errores, pruebas E2E en Android/iOS, documentación de contratos. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| MethodChannel | Canal para invocar métodos nativos desde Dart. |
| EventChannel | Canal para recibir streams de eventos desde nativo. |
| Bridge | Capa de traducción entre APIs nativas y Dart. |
| Lifecycle | Ciclo de vida de inicialización/limpieza de un SDK. |
| Mapping de errores | Traducción de códigos nativos a enum Dart. |

---

## Referencias

- [Platform Channels](https://docs.flutter.dev/development/platform-integration/platform-channels)
- [Flutter Integration Testing](https://docs.flutter.dev/testing/integration-tests)
