# Caso 76: Observabilidad Mobile End-to-End
## Logs, Métricas y Trazas en Banca Flutter

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | observabilidad, logs, métricas, trazas, mobile |
| **Patrón Técnico** | Structured Logging, Distributed Tracing, Mobile APM |
| **Stack Seleccionado** | Flutter + SDK APM (Datadog/Firebase/Sentry) + OpenTelemetry donde aplique + Riverpod context for tracing |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Logs locales y crash reports básicos no permiten correlacionar con backend.
- Sin métricas UX/perf y umbrales, el equipo detecta incidentes tarde.
- Falta de políticas de PII genera riesgo de cumplimiento.

### Escenario de Negocio

> *"Como equipo, necesito ver qué pasa en prod: errores, latencia, flujos críticos, y correlacionar con backend."*

### Incidentes reportados
- **SRE:** Falta de trazas y contextos prolonga MTTR.
- **Banca:** Auditorías fallaron por registros incompletos y PII en logs.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| SRE reports | Global | Observabilidad 3 pilares reduce MTTR. |
| Auditorías banca | LATAM/EU | PII en logs y falta de correlación son hallazgos frecuentes. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; observabilidad/config es brecha común. |

**Resumen global**
- Observabilidad completa (logs estructurados, métricas UX/negocio, trazas correlacionadas) acelera diagnóstico y cumple normativas; ausencia de PII policies y correlación incrementa MTTR y riesgo legal.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Operacional** | Incidentes prolongados, falta de diagnóstico |
| **Regulatorio** | Auditoría incompleta, incumplimiento |
| **UX** | Bugs no detectados afectan usuarios y NPS |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Logs locales y crash reports básicos | **INADECUADO:** Sin correlación ni métricas. |
| **ACEPTABLE** | Crashlytics/APM básico | **MEJORA:** Mejor visibilidad, pero sin trazas ni contexto de negocio completo. |
| **ENTERPRISE** | **Observabilidad 3 pilares:** logs estructurados, métricas de negocio/UX, trazas con IDs correlacionados, sampling, alertas con umbrales | **ÓPTIMO:** Diagnóstico rápido y cumplimiento. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Agregar trace/span IDs a requests y logs. Métricas de negocio (conversión, error rate) y UX (TTI, ANR). Logs estructurados sin PII. Sampling para performance. Alertas con umbrales. |
| **Restricciones Duras (NO permite)** | **PII:** No loggear datos sensibles. **Costo:** APM/trazas generan costos; usar sampling. **Privacidad:** Cumplir GDPR/leyes locales. |
| **Criterio de Selección** | SDK APM confiable; correlación de IDs; políticas de PII; sampling; tableros y alertas claros. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Logs estructurados | Sin PII; campos clave presentes | Móvil/Sec |
| Trazas y métricas | Correlación front-back, TTI/ANR/perf | Móvil/SRE |
| Alertas | Umbrales de crash/ANR/latencia operan | SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Tableros | Flujos críticos con métricas visibles | Prioriza respuesta |
| Consentimiento | Respetar opt-in/opt-out de analítica | Cumplimiento |
| Sampling | Balance costo vs detalle | Eficiencia |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| PII | Mascarar/omitir datos sensibles | Legal |
| Retención | Políticas de retención alineadas a regulación | Cumplimiento |
| Instrumentación | Checklist de spans/logs en features nuevas | Consistencia |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Incidentes sin visibilidad y riesgo de PII en logs. |
| Opciones evaluadas | Crash básico; APM parcial; observabilidad completa con 3 pilares y políticas de PII. |
| Decisión | Observabilidad 3 pilares con correlación, métricas UX/negocio y políticas de PII/sampling. |
| Consecuencias | Requiere inversión en APM y disciplina de instrumentación. |
| Riesgos aceptados | Costos de APM; sobrecarga si sampling es bajo. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| MTTR | ↓ vs baseline | Warning si no baja | Respuesta a incidentes |
| Crash/ANR | Tendencia a la baja | Crítico si sube | Estabilidad |
| Cobertura de trazas | Alta en flujos críticos | Alerta si baja | Diagnóstico |
| Hallazgos de PII | 0 en auditorías | Crítico si >0 | Cumplimiento |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| APM | Application Performance Monitoring. |
| Trace/Span ID | Identificadores para correlacionar trazas. |
| Sampling | Tomar una fracción de eventos/trazas para reducir costo. |
| ANR | App Not Responding; bloqueo prolongado del hilo UI. |
| PII | Información personal identificable; no se debe loggear. |

---

## Referencias

- [OpenTelemetry](https://opentelemetry.io/)
- [Firebase Crashlytics/Performance](https://firebase.google.com/docs)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
