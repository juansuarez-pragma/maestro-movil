# Caso 95: API Compatibility Layer
## Soportar Clientes v1, v2 y v3 Simultáneamente

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | compatibilidad api, versiones múltiples, adapter, gateway |
| **Patrón Técnico** | Compatibility Layer, Adapter, Feature Flags |
| **Stack Seleccionado** | Flutter + adapters por versión + headers/version en interceptor + API gateway |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como app, debo convivir con v1/v2/v3 de API mientras los usuarios actualizan."*

Sin compatibilidad, clientes antiguos fallan y generan incidentes.

### Evidencia de Industria

- **APIs públicas:** Mantienen compatibilidad y adaptan en gateway.
- **Mobile:** Actualizaciones escalonadas requieren coexistencia temporal.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Técnico** | Bugs al consumir contratos diferentes |
| **UX** | Funciones rotas en clientes legacy |
| **Operacional** | Soporte elevado y hotfixes |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Romper contratos y forzar update | **INADECUADO:** Ruptura y mala experiencia. |
| **ACEPTABLE** | Branches de código por versión dispersos | **MEJORA:** Funciona, pero se vuelve espagueti. |
| **ENTERPRISE** | **Capa de compatibilidad:** adapters por módulo, flags para activar nuevas versiones, pruebas contractuales, headers versionados en interceptor | **ÓPTIMO:** Controla migración y reduce regresiones. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Encapsular diferencias por versión. Activar nuevas versiones por flag. Incluir header/version único en cada request. Probar contratos por versión. Plan de sunset y métricas de uso. |
| **Restricciones Duras (NO permite)** | **Deuda acumulada:** Debe retirarse soporte a versiones viejas. **Matriz de pruebas grande:** Necesita automatización. **Adapters complejos:** Si cambios son profundos, pueden inflarse. |
| **Criterio de Selección** | Adapter/gateway, flags para rollout, tests contractuales, métricas y plan de deprecación. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Compatibility Layer | Capa que adapta múltiples versiones de contrato. |
| Sunset | Retiro planificado de una versión. |
| Header de versión | Header HTTP que indica versión esperada. |
| Adapter | Traduce contratos entre cliente y backend. |
| Matriz de pruebas | Combinaciones de versiones y features a validar. |

---

## Referencias

- [API Versioning](https://martinfowler.com/articles/enterpriseREST.html#versioning)
- [Consumer-Driven Contracts](https://martinfowler.com/articles/consumerDrivenContracts.html)
