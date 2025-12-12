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

### Escenario de Negocio

> *"Como app, debo soportar v1 mientras migro a v2/v3 sin romper a usuarios antiguos."*

Cambios de API sin compatibilidad generan fallas masivas en clientes legacy.

### Evidencia de Industria

- **APIs públicas:** Mantienen compatibilidad mientras migran; adaptan en gateway.
- **Incidentes:** Deprecaciones abruptas rompen apps y obligan a updates forzados.

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
