# Caso 75: Canary + Monitoring en Móvil
## Detectar Reversiones Antes de Impactar a Todos

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | canary release, monitoreo, rollback, mobile |
| **Patrón Técnico** | Canary Deploy, Progressive Rollout, Telemetry/Gates |
| **Stack Seleccionado** | Flutter + feature flags/segments + observabilidad (crashlytics/APM) + métricas de UX |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo, quiero detectar regresiones en una pequeña cohorte antes de escalar a todos."*

Sin canary y monitoreo, bugs llegan a toda la base de usuarios.

### Evidencia de Industria

- **Progressive rollout:** Minimiza impacto de regresiones.
- **Observabilidad móvil:** Crash rates y métricas UX deben guiar el rollout.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico/UX** | Bugs masivos si se despliega a todos sin probar cohortes |
| **Técnico** | Sin métricas/gates, el rollout es ciego |
| **Reputacional** | Malas reseñas y churn por regresiones masivas |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Rollout 100% sin observabilidad | **INADECUADO:** Riesgo total. |
| **ACEPTABLE** | Rollout parcial sin métricas claras | **MEJORA:** Menor riesgo, pero sin gatillos de stop. |
| **ENTERPRISE** | **Canary con monitoreo/gates:** cohortes pequeñas, métricas de crash/perf/UX, umbrales de stop/rollback, flags para detener | **ÓPTIMO:** Control del riesgo y rollback rápido. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Desplegar a 1-5% con flags/segmentos. Medir crash rate, ANR, latencia, métricas UX. Definir umbrales de stop automático. Rollback inmediato via flag. Telemetría por versión/cohorte. |
| **Restricciones Duras (NO permite)** | **Sin telemetría en tiempo real:** Decisiones tardías. **Segmentación limitada:** Necesita SDK que soporte targeting. **Falsos positivos:** Umbrales deben calibrarse para no detener sin motivo. |
| **Criterio de Selección** | Flags/segmentación, monitoreo en tiempo casi real, umbrales de stop, plan de rollback. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Canary Release | Rollout a pequeña cohorte para validar antes de 100%. |
| Cohorte | Subconjunto de usuarios para un experimento/rollout. |
| Gate de stop | Condición que detiene rollout si se supera un umbral. |
| Crash rate | Porcentaje de sesiones con crash. |
| Rollback | Desactivar rápidamente la versión/feature problemática. |

---

## Referencias

- [Progressive Delivery](https://launchdarkly.com/blog/progressive-delivery/)
- [Mobile Observability Metrics](https://firebase.google.com/docs/crashlytics)
