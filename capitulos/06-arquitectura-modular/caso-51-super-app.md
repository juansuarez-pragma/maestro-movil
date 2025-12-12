# Caso 51: Super App Architecture
## Cómo WeChat y Rappi Integran Mini-Apps

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | super app, mini-apps, extensibilidad, sandbox |
| **Patrón Técnico** | Micro-frontends, Plugin Architecture, Sandbox Execution |
| **Stack Seleccionado** | Flutter + módulos desacoplados + Dynamic Feature (Android) + app clips/widgets (iOS) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como plataforma, quiero alojar mini-apps de terceros sin comprometer seguridad ni performance."*

Super apps necesitan aislar módulos, versionarlos y actualizar sin romper el core.

### Evidencia de Industria

- **WeChat/Alipay/Rappi:** Modelos de mini-apps con sandbox y APIs controladas.
- **Riesgos documentados:** Mini-apps mal aisladas filtran datos o degradan UX.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Seguridad** | Acceso indebido a datos del host |
| **UX** | Performance degradada por módulos pesados |
| **Técnico** | Acoplamiento y dificultad de versiones |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Incrustar módulos sin aislamiento | **INADECUADO:** Riesgo de fuga de datos y jank. |
| **ACEPTABLE** | Módulos dinámicos con APIs mínimas | **MEJORA:** Algo de aislamiento, pero sin políticas claras. |
| **ENTERPRISE** | **Micro-frontends sandbox:** APIs contractuales, permisos explícitos, carga dinámica, versionado, monitoreo | **ÓPTIMO:** Extensible, seguro y mantenible. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Cargar mini-apps bajo contratos limitados. Versionar y actualizar módulos sin redeploy completo. Monitorear uso y performance por módulo. Revocar módulos inseguros. |
| **Restricciones Duras (NO permite)** | **Aislamiento total:** Flutter no provee sandbox completo; requiere políticas y checks. **Peso de módulos:** Dynamic Feature tiene límites y latencia de carga. **iOS:** App Clips limitan tamaño y capacidades. |
| **Criterio de Selección** | Micro-frontends para extensibilidad; contratos de APIs y permisos; carga dinámica solo para módulos no críticos. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Micro-frontend | Módulo de UI autónomo integrado en una app mayor. |
| Sandbox | Aislamiento de ejecución y permisos limitados. |
| Dynamic Feature | Módulo Android descargable bajo demanda. |
| App Clip | Módulo ligero de iOS para tareas acotadas. |
| API contractual | Interfaz controlada que limita capacidades de un módulo. |

---

## Referencias

- [Micro-frontends](https://micro-frontends.org/)
- [Android Dynamic Feature Modules](https://developer.android.com/guide/app-bundle/dynamic-delivery)
- [Apple App Clips](https://developer.apple.com/app-clips/)
