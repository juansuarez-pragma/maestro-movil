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

### Problema detectado (técnico)
- Romper contratos y forzar update provoca fallas masivas en clientes legacy.
- Branches por versión dispersos crean espagueti y regresiones.
- Sin métricas y sunset, la deuda de versiones crece sin control.

### Escenario de Negocio

> *"Como app, debo convivir con v1/v2/v3 de API mientras los usuarios actualizan."*

### Incidentes reportados
- **Deprecaciones abruptas:** Rompieron clientes no actualizados.
- **Código espagueti:** `if version` disperso generó bugs al migrar a v3.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| APIs públicas | Global | Compatibilidad prolongada con adapters/gateway. |
| Postmortems de migración | Varios | Falta de capa de compatibilidad causó regresiones. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; contratos/versionado son fuente de bugs. |

**Resumen global**
- Capa de compatibilidad con adapters, flags y pruebas contractuales permite migrar sin romper clientes; branches ad-hoc crean deuda y fallas.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Contract tests | v1/v2/v3 cumplen contratos en adapters | QA/Backend |
| Integration (CI) | Headers/version y flags se aplican | Móvil/QA |
| Observabilidad | Uso por versión y errores asociados | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Rollout | Flags por cohorte/versión | Migración controlada |
| Sunset | Comunicar y retirar versiones | Deuda bajo control |
| Errores | Mensajes claros en versión no soportada | UX |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Gobernanza | Catálogo de endpoints/contratos por versión | Trazabilidad |
| Testing | Matriz de versiones automatizada | Previene regresiones |
| Deprecación | Fechas y checklist de retiro | Disciplina |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Convivir con v1/v2/v3 sin romper clientes ni crear espagueti. |
| Opciones evaluadas | Forzar update; branches dispersos; capa de compatibilidad con adapters/flags/tests. |
| Decisión | Capa de compatibilidad con adapters por módulo, flags y pruebas contractuales + plan de sunset. |
| Consecuencias | Matriz de pruebas mayor; requiere gobernanza de contratos. |
| Riesgos aceptados | Deuda temporal; complejidad de adapters si divergencia es grande. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Regresiones por versión | ↓ vs baseline | Crítico si sube | Estabilidad |
| Uso de versiones legacy | ↓ progresivo | Alerta si no baja | Migración |
| Tickets por ruptura | ↓ | Alerta si suben | Soporte |
| Tiempo de sunset | Cumplido según plan | Warning si se extiende | Deuda controlada |

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
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
