# Caso 77: Feature Analytics Confiable
## Medir Uso de Features sin Duplicar Métricas

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | analytics, eventos, duplicados, medición |
| **Patrón Técnico** | Event Taxonomy, Single Event Pipeline, De-duplication |
| **Stack Seleccionado** | Flutter + SDK analytics (GA4/Segment/Amplitude) + event taxonomy + Riverpod para tracking context |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo, necesito métricas confiables de uso de features sin eventos duplicados ni definiciones ambiguas."*

Métricas inconsistentes generan decisiones erróneas y desconfianza en datos.

### Evidencia de Industria

- **Taxonomía de eventos:** Mejora consistencia y reduce ruido.
- **Casos reales:** Eventos duplicados inflan métricas de conversión.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Negocio** | Decisiones basadas en datos incorrectos |
| **Técnico** | Pipelines de datos contaminados |
| **Reputacional** | Desconfianza en analítica |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Eventos ad-hoc sin estándar | **INADECUADO:** Duplicados, ambigüedad. |
| **ACEPTABLE** | Taxonomía parcial, sin control de duplicados | **MEJORA:** Menos caos, pero aún inconsistencias. |
| **ENTERPRISE** | **Taxonomía + pipeline único:** nombres estándar, contexto común, dedup en cliente/servidor, control de sampling | **ÓPTIMO:** Métricas confiables y auditables. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Definir esquema de eventos (nombres, props). Adjuntar contextos comunes (user/device/version). Deduplicar eventos con IDs/hashes. Controlar sampling. Monitorear tasa de eventos y errores de envío. |
| **Restricciones Duras (NO permite)** | **PII:** No incluir datos sensibles. **SDK limitaciones:** Algunos SDKs no soportan dedup nativo; puede requerir backend. **Sobrecarga:** Contexto excesivo aumenta payload. |
| **Criterio de Selección** | Taxonomía clara, pipeline único, dedup, sampling donde aplique, governance de eventos. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Taxonomía de eventos | Esquema estándar de nombres y propiedades. |
| Deduplicación | Evitar contar varias veces el mismo evento. |
| Sampling de eventos | Enviar solo un porcentaje de eventos para reducir carga. |
| Contexto común | Datos compartidos (app_version, device, user_id anon) en todos los eventos. |
| Pipeline único | Un solo flujo de envío y procesamiento para consistencia. |

---

## Referencias

- [Event Naming Best Practices](https://segment.com/academy/)
- [GA4 Measurement](https://developers.google.com/analytics/devguides/collection/ga4)
