# Caso 51: Super App Architecture
## Cómo WeChat y Rappi Integran Mini-Apps

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | super app, mini-apps, extensibilidad, sandbox |
| **Patrón Técnico** | Micro-frontends, Plugin Architecture, [Sandbox](#term-sandbox "Aislamiento de ejecución y permisos limitados.") Execution |
| **Stack Seleccionado** | Flutter + módulos desacoplados + [Dynamic Feature](#term-dynamic-feature "Módulo Android descargable bajo demanda.") (Android) + App Clips/widgets (iOS) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Mini-apps sin contratos ni permisos claros pueden leer datos del host o bloquear la UI.
- Cargar todo en el binario principal crece el bundle y frena publicaciones.
- Sin versionado/telemetría por módulo no se detectan regresiones ni abusos.

### Escenario de Negocio

> *"Como plataforma, quiero alojar mini-apps de terceros sin comprometer seguridad ni performance."*

### Incidentes reportados
- **WeChat/Alipay:** Modelos sandbox reducen riesgo de fuga de datos; incidentes previos llevaron a permisos finos.
- **Rappi/Grab:** Requirieron gobernanza y monitoreo por módulo para evitar bloat y jank.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| WeChat/Alipay | APAC | Mini-apps aisladas con APIs limitadas y review central. |
| App bundles dinámicos | Global | Dynamic Feature reduce tamaño inicial, pero penaliza latencia si se abusa. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; permisos y datos compartidos son hallazgos frecuentes. |

**Resumen global**
- Sin sandbox y contratos, las mini-apps elevan riesgo de fuga y degradación. Necesarios permisos explícitos, carga bajo demanda y observabilidad por módulo.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Seguridad** | Acceso indebido a datos del host o APIs internas |
| **UX/Performance** | Jank y bundle inflado por módulos pesados |
| **Operacional** | Sin telemetría no se detectan abusos ni regresiones por módulo |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Incrustar módulos sin aislamiento | **INADECUADO:** Riesgo de fuga y bloat del binario. |
| **ACEPTABLE** | Módulos dinámicos con APIs mínimas | **MEJORA:** Baja tamaño, pero sin permisos ni monitoreo claro. |
| **ENTERPRISE** | **Micro-frontends sandbox:** APIs contractuales, permisos explícitos, carga dinámica, versionado y telemetría por módulo | **ÓPTIMO:** Extensible y controlado, gobernanza central. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Cargar mini-apps con contratos limitados. Versionar y actualizar módulos sin redeploy completo. Telemetría y kill-switch por módulo. |
| **Restricciones Duras (NO permite)** | **Aislamiento total:** Flutter no provee sandbox completo; se requieren políticas y revisión. **Peso de módulos:** Dynamic Feature tiene límites y latencia de carga. **iOS:** App Clips limitan tamaño y capacidades. |
| **Criterio de Selección** | Micro-frontends con permisos explícitos; carga bajo demanda solo para módulos no críticos; monitoreo y revocación rápida. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Seguridad (CI) | APIs expuestas cumplen contratos y permisos | Seguridad/QA |
| Integration (CI) | Carga/descarga de módulos y fallback | Móvil/QA |
| Observabilidad | Eventos por módulo (latencia, errores, consumo) | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Carga bajo demanda | Prefetch en WiFi para módulos frecuentes | Balancea latencia |
| Degradación | Fallback a web/light si falla descarga | Disponibilidad |
| Permisos | Solicitud granular por mini-app | Transparencia al usuario |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Gobernanza | Catálogo de APIs/permiso por módulo; review antes de publicar | Control de calidad |
| Kill-switch | Desactivar módulos inseguros/remotos | Contención |
| Tamaño | Límite por módulo y budget de app | Previene bloat |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Extender la app con mini-apps sin romper seguridad ni performance. |
| Opciones evaluadas | Módulos embebidos; Dynamic Feature sin gobernanza; micro-frontends con contratos/telemetría. |
| Decisión | Micro-frontends con permisos explícitos, carga bajo demanda y telemetría por módulo. |
| Consecuencias | Requiere catálogo de APIs y proceso de review; complejidad de orquestar versiones. |
| Riesgos aceptados | Sandbox incompleto; latencia inicial si el prefetch falla. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Tamaño del binario | ↓ vs baseline por carga dinámica | Alerta si crece | Descarga rápida |
| Latencia de carga de módulo | p95 controlado | Warning si aumenta | UX estable |
| Incidentes de permisos | 0 fugas | Crítico si >0 | Seguridad |
| Crash/jank por módulo | Tendencia a la baja | Alerta si sube | Calidad |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-micro-frontend"></a>Micro-frontend | Módulo de UI autónomo integrado en una app mayor. |
| <a id="term-sandbox"></a>Sandbox | Aislamiento de ejecución y permisos limitados. |
| <a id="term-dynamic-feature"></a>Dynamic Feature | Módulo Android descargable bajo demanda. |
| <a id="term-app-clip"></a>App Clip | Módulo ligero de iOS para tareas acotadas. |
| <a id="term-api-contractual"></a>API contractual | Interfaz controlada que limita capacidades de un módulo. |

---

## Referencias

- [Micro-frontends](https://micro-frontends.org/)
- [Android Dynamic Feature Modules](https://developer.android.com/guide/app-bundle/dynamic-delivery)
- [Apple App Clips](https://developer.apple.com/app-clips/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
