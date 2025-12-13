# Caso 63: Plugin Nativo [Legacy](#term-legacy "Código obsoleto o sin mantenimiento.")
## Migrar un Plugin Obsoleto sin Romper Producción

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | plugin legacy, migración, compatibilidad, nativo |
| **Patrón Técnico** | [Strangler Pattern](#term-strangler-pattern "Reemplazar componentes gradualmente ejecutando dual-stack."), Compatibility Layer, Dual-stack |
| **Stack Seleccionado** | Flutter + plugin legacy + nuevo plugin paralelo + feature flags |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Plugin legacy sin mantenimiento se rompe en nuevas versiones de SO.
- Reemplazo directo sin dual-stack provoca regresiones y fallas en producción.
- Sin telemetría comparativa no se sabe cuándo el nuevo plugin es estable.

### Escenario de Negocio

> *"Como equipo, debemos reemplazar un plugin obsoleto sin interrumpir a los usuarios."*

### Incidentes reportados
- **Migraciones de SDKs nativos:** Reemplazos directos rompieron apps en nuevas versiones de SO.
- **Plugins sin soporte:** Fallaron tras cambios de permisos/arquitecturas (ARM64/vulkan, etc.).

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Migraciones nativas | Global | Estrangulamiento con dual-stack reduce riesgo de cutover. |
| Plugins Flutter | Global | Cambios de API nativa requieren coordinación Dart+nativo y pruebas por plataforma. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; dependencias nativas desactualizadas son hallazgo común. |

**Resumen global**
- Migrar con dual-stack, flags y telemetría permite rollback rápido; reemplazos directos elevan riesgo de incidentes en producción.

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
| **Capacidades (SÍ permite)** | Ejecutar ambos plugins bajo un wrapper unificado. Seleccionar plugin por flag/versión de SO. Medir errores/latencia por plugin. [Rollback](#term-rollback "Volver rápidamente a la implementación previa.") instantáneo si el nuevo falla. |
| **Restricciones Duras (NO permite)** | **Doble mantenimiento:** Requiere esfuerzos en dos stacks temporalmente. **Tamaño:** Puede aumentar el bundle. **Compatibilidad:** Algunos SO/arquitecturas pueden no soportar ambos. |
| **Criterio de Selección** | Strangler con flags y telemetría; wrapper común; plan de sunset para legacy tras estabilizar el nuevo. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Integration (CI) | Wrapper usa plugin correcto por flag/SO | Móvil/QA |
| Comparativa | Errores/latencia legacy vs nuevo | Móvil/SRE |
| Seguridad | Permisos y cambios de SO soportados | Seguridad |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Rollout | Flags por cohorte/SO para cutover gradual | Control de riesgo |
| Mensajes | Errores claros y fallback a legacy si falla nuevo | UX estable |
| Sunset | Fecha y checklist para retirar legacy | Reduce deuda |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Telemetría | Eventos `plugin.*` con versión/origen | Observabilidad |
| Compatibilidad | Matriz de SO/arquitecturas probadas | Previene sorpresas |
| Tamaño | Medir impacto de doble plugin | Bloat controlado |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Reemplazar plugin obsoleto sin interrumpir producción. |
| Opciones evaluadas | [Cutover](#term-cutover "Momento de cambio definitivo a la nueva implementación.") directo; branch única; dual-stack con flags y telemetría. |
| Decisión | Dual-stack con wrapper común, flags y rollback rápido. |
| Consecuencias | Mantener dos plugins temporalmente; monitoreo adicional. |
| Riesgos aceptados | Mayor tamaño del app; doble mantenimiento transitorio. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Incidentes en cutover | 0 | Crítico si >0 | Estabilidad |
| Crash rate por plugin | ↓ en nuevo vs legacy | Warning si sube | Calidad |
| Tiempo de migración | Predecible con flags | Alerta si se extiende | Productividad |
| Peso del binario | Controlado con doble plugin | Alerta si crece | Experiencia descarga |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-strangler-pattern"></a>Strangler Pattern | Reemplazar componentes gradualmente ejecutando dual-stack. |
| <a id="term-cutover"></a>Cutover | Momento de cambio definitivo a la nueva implementación. |
| <a id="term-telemetria-comparativa"></a>Telemetría comparativa | Medir comportamiento de ambas implementaciones. |
| <a id="term-legacy"></a>Legacy | Código obsoleto o sin mantenimiento. |
| <a id="term-rollback"></a>Rollback | Volver rápidamente a la implementación previa. |

---

## Referencias

- [Strangler Fig Pattern](https://martinfowler.com/bliki/StranglerFigApplication.html)
- [Flutter Plugin Migration Guides](https://docs.flutter.dev/development/packages-and-plugins/developing-packages)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
