# Caso 99: Feature Parity Dashboard
## Tracking de Migración Funcionalidad por Funcionalidad

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | feature parity, migración, dashboard, trazabilidad |
| **Patrón Técnico** | Parity Matrix, Traceability, Rollout Tracking |
| **Stack Seleccionado** | Tablero (Notion/Jira/Sheet) + flags por feature + métricas de adopción |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Sin visibilidad de paridad, se duplican esfuerzos y se olvidan edge cases.
- Listas manuales sin estados/propietarios son poco accionables.
- Sin métricas de adopción/bugs, no se sabe cuándo retirar legacy.

### Escenario de Negocio

> *"Como equipo, necesito saber qué funcionalidades del legacy ya existen en el nuevo stack y cuál es su estado."*

### Incidentes reportados
- **Migraciones grandes:** Features faltantes detectados tarde generaron regresiones.
- **Sin trazabilidad:** Trabajo duplicado y retrasos al no saber qué está listo.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Prácticas de migración | Global | Parity matrices evitan gaps funcionales. |
| Postmortems | Varios | Falta de dashboard causó retrabajo y bugs. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; governance/config limitada. |

**Resumen global**
- Dashboard de paridad con estados, flags y métricas reduce gaps y retrabajo; migrar sin trazabilidad genera regresiones.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Operacional** | Falta de funcionalidades, bugs por gaps |
| **Negocio** | Usuarios afectados por regressiones funcionales |
| **Productivo** | Re-trabajo y retrasos por falta de trazabilidad |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Migrar sin checklist ni seguimiento | **INADECUADO:** Gaps y regresiones. |
| **ACEPTABLE** | Lista manual sin estados claros | **MEJORA:** Algo de visibilidad, pero poco accionable. |
| **ENTERPRISE** | **Dashboard de paridad:** mapa de features legacy→nuevo, estados, flags, métricas de uso y bugs, dueños | **ÓPTIMO:** Trazabilidad y control del avance. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Mapear cada feature legacy a su reemplazo. Estado (pendiente/en progreso/done). Flags y cohortes. [Métricas de adopción](#term-metricas-de-adopcion "Uso de la nueva funcionalidad vs legacy.") y defectos. Dueños claros. |
| **Restricciones Duras (NO permite)** | **Exactitud:** Requiere mantenerlo actualizado. **Scope creep:** Definir claramente qué es "parity". **Automatización limitada:** Algunas métricas deben ingresarse manualmente. |
| **Criterio de Selección** | Dashboard vivo con responsables; flags y métricas de adopción; definición clara de done por feature. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Compleción | Features marcados como done cumplen criterios | QA/Producto |
| Adopción | Uso vs legacy medido | Data |
| Bugs | Defectos por feature rastreados | QA/Móvil |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Flags | Activar por cohorte y limpiar al completar | Deuda controlada |
| Comunicación | Visibilidad de estado a negocio/soporte | Alineación |
| [Definición de done](#term-definicion-de-done "Criterios para considerar un feature completo.") | Checklist con UX/perf/bugs | Claridad |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Gobernanza | Dueños por feature y revisión periódica | Responsabilidad |
| Métricas | Adopción, defectos y tiempo por feature | Priorización |
| Sunset | Retirar legacy al completar paridad | Salud del código |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Falta de visibilidad de paridad funcional durante migración. |
| Opciones evaluadas | Migrar sin checklist; lista manual; dashboard con estados/flags/métricas. |
| Decisión | Dashboard de paridad con estados, dueños, flags y métricas de adopción/defectos. |
| Consecuencias | Requiere mantenimiento y gobernanza; parte de datos puede ser manual. |
| Riesgos aceptados | Desactualización si no se mantiene; esfuerzo de operación. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Gaps funcionales | 0 al cerrar paridad | Crítico si >0 | Calidad |
| Tiempo por feature | Predecible | Warning si crece | Planificación |
| Bugs por feature migrada | Tendencia a la baja | Crítico si sube | Estabilidad |
| Deuda de flags | Flags limpios al completar | Warning si quedan | Salud |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-feature-parity"></a>Feature parity | Equivalencia funcional entre legacy y nuevo sistema. |
| <a id="term-cohorte"></a>Cohorte | Grupo de usuarios migrados a la nueva versión. |
| <a id="term-definicion-de-done"></a>Definición de done | Criterios para considerar un feature completo. |
| <a id="term-metricas-de-adopcion"></a>Métricas de adopción | Uso de la nueva funcionalidad vs legacy. |
| <a id="term-traceabilidad"></a>Traceabilidad | Capacidad de rastrear estado y propietarios. |

---

## Referencias

- [Migration Feature Parity Practices](https://martinfowler.com/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
- [Strangler Fig Pattern](https://martinfowler.com/bliki/StranglerFigApplication.html)
