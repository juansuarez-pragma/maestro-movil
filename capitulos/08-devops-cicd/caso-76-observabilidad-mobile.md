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

### Escenario de Negocio

> *"Como equipo, necesito ver qué pasa en prod: errores, latencia, flujos críticos, y correlacionar con backend."*

Sin observabilidad, incidentes se diagnostican a ciegas y se alarga el MTTR.

### Evidencia de Industria

- **SRE:** Logs estructurados, métricas y trazas reducen MTTR.
- **Banca:** Cumplimiento y auditoría requieren registros confiables.

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
| **Capacidades (SÍ permite)** | Agregar trace/span IDs a requests y logs. Métricas de negocio (conversion, error rate) y UX (TTI, ANR). Logs estructurados sin PII. Sampling para performance. Alertas con umbrales. |
| **Restricciones Duras (NO permite)** | **PII:** No loggear datos sensibles. **Costo:** APM y trazas generan costos; usar sampling. **Privacidad:** Cumplir GDPR/leyes locales. |
| **Criterio de Selección** | SDK APM confiable; correlación de IDs; políticas de PII; sampling; tableros y alertas claros. |

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
