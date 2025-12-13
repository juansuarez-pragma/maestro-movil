# Caso 92: API v1 Legacy en Producción
## Evolucionar sin Romper Clientes Antiguos

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | api legacy, versionado, compatibilidad, migración |
| **Patrón Técnico** | API Gateway/Adapter, Backward Compatibility, Deprecation Plan |
| **Stack Seleccionado** | Flutter + adapters por versión + feature flags + API gateway |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Cortar v1 sin plan rompe clientes legacy y fuerza updates masivos.
- Mantener v1 indefinidamente perpetúa deuda y frena evolución.
- Sin métricas de uso, no se sabe cuándo ni cómo hacer sunset seguro.

### Escenario de Negocio

> *"Como app, debo soportar v1 mientras migro a v2/v3 sin romper a usuarios antiguos."*

### Incidentes reportados
- **Deprecaciones abruptas:** Rompieron apps legacy y generaron parches urgentes.
- **Falta de métricas:** Equipos no sabían quién seguía en v1 y retrasaron el sunset.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| APIs públicas | Global | Compatibilidad prolongada con adapters/gateway es práctica común. |
| Postmortems de deprecación | Varios | Cortes sin plan causaron fallas masivas. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; contratos/versionado suelen ser frágiles. |

**Resumen global**
- Gateway/adapters con flags y métricas permiten migrar versiones sin romper clientes; deprecaciones sin plan generan incidentes masivos.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Técnico** | Regresiones y errores en clientes no actualizados |
| **UX** | Funciones degradadas o rotas en versiones antiguas |
| **Operacional** | Soporte elevado y parches urgentes |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Forzar update inmediato y cortar v1 | **INADECUADO:** Ruptura y mala experiencia. |
| **ACEPTABLE** | Mantener v1 indefinidamente | **MEJORA:** Evita ruptura, pero perpetúa deuda. |
| **ENTERPRISE** | **Gateway/adapters + plan de sunset:** compatibilidad en gateway, adapters en cliente, flags, métricas de uso, fechas de deprecación | **ÓPTIMO:** Migración gradual, menos riesgo. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Encapsular diferencias en gateway/adapters. Telemetría para saber uso de v1. Comunicar y planificar sunset. Usar flags para activar nuevas versiones por segmento. |
| **Restricciones Duras (NO permite)** | **Compatibilidad infinita:** Deuda aumenta; requiere sunset. **Testing complejo:** Matriz de versiones. **Payload divergente:** Adapters crecen si contratos difieren mucho. |
| **Criterio de Selección** | Adapter/gateway, flags para rollout, métricas y fechas de deprecación, plan de comunicación. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Contract tests | Adapters por versión y gateway traducen correctamente | QA/Backend |
| Rollout | Flags por versión/cohorte funcionan | Móvil/QA |
| Observabilidad | Uso de v1/v2/v3 y errores por versión | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Mensajes | Notificar deprecación y fechas límite | Transparencia |
| Fallback | Mantener experiencia estable en legacy hasta sunset | UX |
| Soporte | Plan de acompañamiento para actualizar | Operación |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Sunset | Fechas y checklist de retiro de v1 | Disciplina |
| Debt | Retirar código v1 tras sunset | Salud |
| Gobernanza | Catálogo de versiones y contratos | Control |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Migrar v1 a v2/v3 sin romper clientes legacy. |
| Opciones evaluadas | Cortar v1; mantener indefinidamente; gateway/adapters + plan de sunset con flags/metrics. |
| Decisión | Gateway/adapters con flags, telemetría y plan de sunset comunicado. |
| Consecuencias | Matriz de versiones y mantenimiento temporal; requiere gobernanza. |
| Riesgos aceptados | Deuda temporal; complejidad de testing multiplataforma. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Uso de v1 | ↓ progresivo | Alerta si no baja | Migración |
| Regresiones por versión | Tendencia a la baja | Crítico si sube | Estabilidad |
| Tickets por ruptura | ↓ vs baseline | Alerta si sube | Soporte |
| Tiempo de sunset | Cumplido según plan | Warning si se extiende | Deuda controlada |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| API Gateway | Capa que adapta y enruta versiones de API. |
| Deprecation/Sunset | Proceso de retirar una versión antigua. |
| Adapter | Traduce entre contratos viejos y nuevos. |
| Compatibilidad hacia atrás | Mantener clientes antiguos funcionando. |
| Métricas de uso | Datos de qué clientes siguen en v1. |

---

## Referencias

- [API Versioning](https://martinfowler.com/articles/enterpriseREST.html#versioning)
- [Deprecation Policies](https://cloud.google.com/apis/design/versioning)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
