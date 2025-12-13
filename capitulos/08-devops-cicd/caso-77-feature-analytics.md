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

### Problema detectado (técnico)
- Eventos ad-hoc generan duplicados y definiciones ambiguas.
- Sin deduplicación ni esquema, conversiones/embudos quedan inflados o incompletos.
- Falta de contexto común dificulta atribución y debugging de datos.

### Escenario de Negocio

> *"Como equipo, necesito métricas confiables de uso de features sin eventos duplicados ni definiciones ambiguas."*

### Incidentes reportados
- **Taxonomía pobre:** Infló conversiones por duplicados; negocio perdió confianza.
- **Múltiples SDKs/pipelines:** Datos inconsistentes por falta de fuente única.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Guías de analytics (Segment/GA4) | Global | Taxonomía estándar y contexto común reducen ruido. |
| Postmortems de datos | Varios | Duplicados y eventos mal definidos generan decisiones erróneas. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; gestión de datos/config limitada. |

**Resumen global**
- Taxonomía + pipeline único + dedup/sampling producen métricas confiables; eventos ad-hoc erosionan la confianza en los datos.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Contract tests | Taxonomía y contexto común | Data/QA |
| Dedup | IDs/hashes evitan duplicados | Móvil/Data |
| Observabilidad | Tasa de eventos, errores de envío, sampling efectivo | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Consentimiento | Respetar opt-in/opt-out | Cumplimiento |
| Payload | Props mínimas, sin PII | Seguridad |
| Gobernanza | Dueños por evento y changelog | Trazabilidad |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Fuente única | Evitar múltiples pipelines paralelos | Consistencia |
| Sampling | Ajustar en picos para costo/calidad | Eficiencia |
| Auditoría | Revisiones periódicas de taxonomía | Calidad de datos |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Métricas inconsistentes por eventos ad-hoc y duplicados. |
| Opciones evaluadas | Ad-hoc; taxonomía parcial; taxonomía + pipeline único con dedup/sampling. |
| Decisión | Taxonomía estándar, pipeline único, dedup/sampling y governance. |
| Consecuencias | Requiere disciplina y soporte de datos; coordinación en cambios de taxonomía. |
| Riesgos aceptados | Overhead inicial; dependencia del SDK/backend para dedup. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Eventos duplicados | 0 | Crítico si >0 | Calidad de datos |
| Confianza en métricas | Alta (sin divergencias entre fuentes) | Warning si hay gaps | Decisiones fiables |
| Latencia de ingesta | Estable | Alerta si sube | Oportunidad |
| Incidentes por PII | 0 | Crítico si >0 | Cumplimiento |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-taxonomia-de-eventos"></a>Taxonomía de eventos | Esquema estándar de nombres y propiedades. |
| <a id="term-deduplicacion"></a>Deduplicación | Evitar contar varias veces el mismo evento. |
| <a id="term-sampling-de-eventos"></a>Sampling de eventos | Enviar solo un porcentaje de eventos para reducir carga. |
| <a id="term-contexto-comun"></a>Contexto común | Datos compartidos (app_version, device, user_id anon) en todos los eventos. |
| <a id="term-pipeline-unico"></a>Pipeline único | Un solo flujo de envío y procesamiento para consistencia. |

---

## Referencias

- [Event Naming Best Practices](https://segment.com/academy/)
- [GA4 Measurement](https://developers.google.com/analytics/devguides/collection/ga4)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
