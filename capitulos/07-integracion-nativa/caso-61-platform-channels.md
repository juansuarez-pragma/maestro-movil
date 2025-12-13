# Caso 61: Platform Channels sin Dolor
## Integrar SDKs Nativos (Stripe/Facetec) en Flutter

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | platform channels, sdk nativo, integración, kotlin/swift |
| **Patrón Técnico** | Platform Channels, Method/Event Channels, [Bridge](#term-bridge "Capa de traducción entre APIs nativas y Dart.") Pattern |
| **Stack Seleccionado** | Flutter + Kotlin/Swift + [MethodChannel](#term-methodchannel "Canal para invocar métodos nativos desde Dart.")/[EventChannel](#term-eventchannel "Canal para recibir streams de eventos desde nativo.") + Riverpod para estado |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Llamadas directas sin bridge tipado generan crashes, memory leaks y APIs frágiles.
- Errores nativos no mapeados rompen UX y dificultan soporte.
- Sin pruebas por plataforma, las diferencias Android/iOS pasan a producción.

### Escenario de Negocio

> *"Como equipo, necesitamos integrar SDKs nativos críticos (pagos/biometría) sin romper la app Flutter."*

### Incidentes reportados
- **Pagos/biometría:** SDKs nativos con callbacks complejos; integraciones sin bridge tipado causaron crashes y bloqueos de UI.
- **Compatibilidad:** Cambios de SDK nativo rompieron apps Flutter sin sincronizar contratos.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Integraciones de pagos/ID | Global | Requieren flujos nativos y manejo de callbacks/errores consistente. |
| Postmortems de SDKs | Varios | Falta de bridges tipados causó crashes en producción. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; integración nativa/errores es foco recurrente. |

**Resumen global**
- Bridges tipados con manejo de errores y pruebas por plataforma reducen crashes y deuda; integraciones directas elevan riesgo operativo.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Integration (Android/iOS) | Bridge tipado, callbacks y mapping de errores | Móvil/QA |
| Seguridad | Manejo de datos sensibles y permisos nativos | Seguridad |
| Observabilidad | Eventos `channel.*` con errores y latencia | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Errores | Mensajes claros y reintentos seguros | UX transparente |
| [Lifecycle](#term-lifecycle "Ciclo de vida de inicialización/limpieza de un SDK.") | Init/dispose controlado para evitar leaks | Estabilidad |
| Compatibilidad | Matriz de versiones SDK/OS soportadas | Riesgo acotado |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Versionado | Sincronizar versiones de SDK y Dart | Consistencia |
| Threading | Ejecutar APIs en hilo correcto | Previene crashes |
| Testing | CI por plataforma antes de release | Calidad |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Integrar SDKs nativos críticos sin estabilidad ni trazabilidad. |
| Opciones evaluadas | Llamadas directas; wrapper básico; bridge tipado con mapping de errores y pruebas por plataforma. |
| Decisión | Bridge robusto con API tipada, manejo de errores y pruebas Android/iOS. |
| Consecuencias | Requiere disciplina en contratos y CI por plataforma. |
| Riesgos aceptados | Dependencia en SDK de terceros; mantenimiento de compatibilidad. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Crashes de integración | ↓ vs baseline | Crítico si sube | Estabilidad |
| Éxito en flujo nativo | > 99% | Warning si baja | Conversión |
| Tiempo de integración | Predecible por release | Alerta si crece | Productividad |
| Incidentes por versión SDK | 0 | Crítico si >0 | Confiabilidad |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-methodchannel"></a>MethodChannel | Canal para invocar métodos nativos desde Dart. |
| <a id="term-eventchannel"></a>EventChannel | Canal para recibir streams de eventos desde nativo. |
| <a id="term-bridge"></a>Bridge | Capa de traducción entre APIs nativas y Dart. |
| <a id="term-lifecycle"></a>Lifecycle | Ciclo de vida de inicialización/limpieza de un SDK. |
| <a id="term-mapping-de-errores"></a>Mapping de errores | Traducción de códigos nativos a enum Dart. |

---

## Referencias

- [Platform Channels](https://docs.flutter.dev/development/platform-integration/platform-channels)
- [Flutter Integration Testing](https://docs.flutter.dev/testing/integration-tests)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
