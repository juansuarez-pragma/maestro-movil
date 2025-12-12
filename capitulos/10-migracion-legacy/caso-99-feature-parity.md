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

### Escenario de Negocio

> *"Como equipo, necesito saber qué funcionalidades del legacy ya existen en el nuevo stack y cuál es su estado."*

Sin visibilidad, la migración se alarga, se duplican esfuerzos y se olvidan casos edge.

### Evidencia de Industria

- **Migraciones grandes:** Parity matrices para controlar avance y evitar gaps.
- **Errores comunes:** Funcionalidades faltantes detectadas tarde.

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
| **Capacidades (SÍ permite)** | Mapear cada feature legacy a su reemplazo. Estado (pendiente/en progreso/done). Flags y cohortes. Métricas de adopción y defectos. Dueños claros. |
| **Restricciones Duras (NO permite)** | **Exactitud:** Requiere mantenerlo actualizado. **Scope creep:** Definir claramente qué es "parity". **Automatización limitada:** Algunas métricas deben ingresarse manualmente. |
| **Criterio de Selección** | Dashboard vivo con responsables; flags y métricas de adopción; definición clara de done para cada feature. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Feature parity | Equivalencia funcional entre legacy y nuevo sistema. |
| Cohorte | Grupo de usuarios migrados a la nueva versión. |
| Definición de done | Criterios para considerar un feature completo. |
| Métricas de adopción | Uso de la nueva funcionalidad vs legacy. |
| Traceabilidad | Capacidad de rastrear estado y propietarios. |

---

## Referencias

- [Migration Feature Parity Practices](https://martinfowler.com/)
