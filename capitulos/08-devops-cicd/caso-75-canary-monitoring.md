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

### Problema detectado (técnico)
- Rollout 100% sin telemetría lleva bugs a todos los usuarios.
- Sin umbrales/gates, el equipo no detiene a tiempo una regresión.
- Sin segmentación/flags, el rollback es lento y afecta releases completos.

### Escenario de Negocio

> *"Como equipo, quiero detectar regresiones en una pequeña cohorte antes de escalar a todos."*

### Incidentes reportados
- **Progressive rollout:** Equipos sin gates lanzaron regresiones masivas y recibieron bajas reseñas.
- **Banca/fintech:** Crash spikes sin canary generaron caídas de conversión y soporte elevado.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Progressive delivery | Global | Cohortes pequeñas reducen impacto de regresiones. |
| Mobile observability | Global | Crash/ANR y perf son métricas clave para gates. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; falta de monitoreo/gobernanza en releases es común. |

**Resumen global**
- Canary con segmentación, métricas y gates automáticos reduce riesgo y MTTR; rollouts ciegos escalan incidentes a toda la base.

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
| **Capacidades (SÍ permite)** | Desplegar a 1-5% con flags/segmentos. Medir crash rate, ANR, latencia, métricas UX. Definir umbrales de stop automático. [Rollback](#term-rollback "Desactivar rápidamente la versión/feature problemática.") inmediato vía flag. Telemetría por versión/cohorte. |
| **Restricciones Duras (NO permite)** | **Sin telemetría en tiempo real:** Decisiones tardías. **Segmentación limitada:** Necesita SDK que soporte targeting. **Falsos positivos:** Umbrales deben calibrarse. |
| **Criterio de Selección** | Flags/segmentación, monitoreo en tiempo casi real, umbrales de stop, plan de rollback. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Canary | Habilitar 1-5% con métricas por cohorte | Móvil/QA |
| Gates | Stop/rollback al superar umbrales | Móvil/SRE |
| Observabilidad | Crash/ANR/perf/UX por versión/cohorte | SRE/Data |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Comunicación | Estado del canary visible para soporte/negocio | Alineación |
| Rollout | Incrementos controlados (1%→5%→25%→100%) | Riesgo acotado |
| Rollback | Flag/segmento para detener en minutos | MTTR bajo |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Thresholds | Umbrales claros (crash/ANR/perf/UX) | Consistencia |
| Datos | Separar canary vs producción plena | Diagnóstico |
| Auditoría | Registro de cambios de cohorte y resultados | Trazabilidad |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Lanzar sin canary ni métricas eleva riesgo masivo. |
| Opciones evaluadas | Rollout 100%; rollout parcial sin gates; canary con métricas y stop automático. |
| Decisión | Canary con segmentación, métricas y gates de stop/rollback. |
| Consecuencias | Requiere observabilidad fuerte y calibrar umbrales. |
| Riesgos aceptados | Posibles falsos positivos; complejidad de targeting. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Crash/ANR en canary | ≤ baseline | Crítico si sube | Estabilidad |
| Tiempo a rollback | Minutos via flag | Crítico si tarda | MTTR |
| Reseñas negativas | ↓ en releases | Alerta si suben | Reputación |
| Cobertura de cohortes | Rollout escalonado cumplido | Warning si se detiene sin causa | Cadencia |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-canary-release"></a>Canary Release | Rollout a pequeña cohorte para validar antes de 100%. |
| <a id="term-cohorte"></a>Cohorte | Subconjunto de usuarios para un experimento/rollout. |
| <a id="term-gate-de-stop"></a>Gate de stop | Condición que detiene rollout si se supera un umbral. |
| <a id="term-crash-rate"></a>Crash rate | Porcentaje de sesiones con crash. |
| <a id="term-rollback"></a>Rollback | Desactivar rápidamente la versión/feature problemática. |

---

## Referencias

- [Progressive Delivery](https://launchdarkly.com/blog/progressive-delivery/)
- [Mobile Observability Metrics](https://firebase.google.com/docs/crashlytics)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
