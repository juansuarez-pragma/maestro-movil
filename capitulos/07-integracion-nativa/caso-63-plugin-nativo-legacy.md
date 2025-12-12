# Caso 63: Plugin Nativo Legacy
## Migrar un Plugin Obsoleto sin Romper Producción

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | plugin legacy, migración, compatibilidad, nativo |
| **Patrón Técnico** | Strangler Pattern, Compatibility Layer, Dual-stack |
| **Stack Seleccionado** | Flutter + plugin legacy + nuevo plugin paralelo + feature flags |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo, debemos reemplazar un plugin obsoleto sin interrumpir a los usuarios."*

Plugins legacy pueden quedar sin mantenimiento o romperse en nuevas versiones de SO; la migración debe ser segura.

### Evidencia de Industria

- **Migraciones nativas:** Estrangulan componentes legacy con doble stack y cutover gradual.
- **Flutter plugins:** Cambios de APIs nativas requieren coordinar Dart y nativo.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Técnico** | Crashes, incompatibilidad en nuevas versiones de SO |
| **UX** | Funcionalidades degradadas durante migración |
| **Reputacional** | Incidentes en producción por cutover abrupto |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Reemplazo directo en una versión | **INADECUADO:** Alto riesgo de regresiones. |
| **ACEPTABLE** | Branch separada y release único | **MEJORA:** Menos riesgo, pero sin rollback rápido. |
| **ENTERPRISE** | **Strangler/dual-stack:** correr legacy y nuevo en paralelo, feature flags para cutover, telemetría y rollback | **ÓPTIMO:** Migración segura y reversible. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Ejecutar ambos plugins bajo un wrapper unificado. Seleccionar plugin por flag/versión de SO. Medir errores/latencia por plugin. Rollback instantáneo si el nuevo falla. |
| **Restricciones Duras (NO permite)** | **Doble mantenimiento:** Requiere esfuerzos en dos stacks temporalmente. **Tamaño:** Puede aumentar el bundle. **Compatibilidad:** Algunos SO/arquitecturas pueden no soportar ambos. |
| **Criterio de Selección** | Strangler con flags y telemetría; wrapper común; plan de sunset para legacy tras estabilizar el nuevo. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Strangler Pattern | Reemplazar componentes gradualmente ejecutando dual-stack. |
| Cutover | Momento de cambio definitivo a la nueva implementación. |
| Telemetría comparativa | Medir comportamiento de ambas implementaciones. |
| Legacy | Código obsoleto o sin mantenimiento. |
| Rollback | Volver rápidamente a la implementación previa. |

---

## Referencias

- [Strangler Fig Pattern](https://martinfowler.com/bliki/StranglerFigApplication.html)
- [Flutter Plugin Migration Guides](https://docs.flutter.dev/development/packages-and-plugins/developing-packages)
