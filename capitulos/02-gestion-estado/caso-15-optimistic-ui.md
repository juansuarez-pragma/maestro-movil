# Caso 15: Optimistic UI que Mintió
## Revertir Estados Cuando el Backend Rechaza

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | optimistic UI, rollback, consistencia, error handling |
| **Patrón Técnico** | Optimistic Updates, Saga Pattern, Retry with Compensating Action |
| **Stack Seleccionado** | Flutter + Riverpod/StateNotifier + Queue de acciones + SQLite para cola offline |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero ver mis cambios al instante, pero si fallan, que se reviertan sin dejar datos corruptos."*

Optimistic UI mejora la percepción de velocidad, pero si el backend rechaza la operación (validez, stock, permisos), el estado visible puede divergir del real.

### Evidencia de Industria

- **Caso eCommerce 2022:** UI mostraba saldo actualizado; backend rechazó débito. Usuarios confundidos abrieron tickets masivos.
- **Estudio Google UX:** Retroalimentación inmediata reduce abandono, pero requiere manejo robusto de errores.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Transacciones incorrectas, reclamos, reversiones |
| **Reputacional** | Pérdida de confianza por datos "fantasma" |
| **Técnico** | Estados inconsistentes difíciles de depurar |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Update optimista sin manejo de error | **INADECUADO:** Estado queda corrupto si backend rechaza. |
| **ACEPTABLE** | Update optimista + snackbar de error, refresh completo | **MEJORA:** Re-sync parcial, pero UX brusca y costosa en datos. |
| **ENTERPRISE** | **Optimistic + Sagas:** aplicar cambio local + encolar acción; si backend falla, ejecutar acción compensatoria y mostrar causa; retries con backoff | **ÓPTIMO:** Mantiene UX rápida, garantiza consistencia y trazabilidad de fallos. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Encolar acciones con metadatos (versión, timestamp, idempotency) en SQLite. Aplicar UI optimista y revertir con acción compensatoria si falla. Retentar con backoff y límite. Registrar errores para observabilidad. |
| **Restricciones Duras (NO permite)** | **Conflictos de versionado:** Si backend rechaza por versión, requiere re-fetch. **Operaciones no idempotentes:** Necesitan diseño especial de compensación. **Offline largo:** Cola puede expirar o ser invalidada. |
| **Criterio de Selección** | Riverpod/StateNotifier por control explícito de estado; patrón Saga para compensaciones; almacenamiento local para cola y reintentos confiables. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Optimistic UI | Mostrar cambios localmente antes de confirmación del servidor. |
| Acción compensatoria | Operación que revierte o ajusta el estado tras un fallo. |
| Saga Pattern | Coordinación de transacciones con pasos y compensaciones. |
| Backoff | Incrementar espera entre reintentos para reducir carga. |
| Cola de acciones | Lista persistente de operaciones pendientes de confirmación. |

---

## Referencias

- [Fowler - Saga Pattern](https://microservices.io/patterns/data/saga.html)
- [Optimistic UI - Apollo Docs](https://www.apollographql.com/docs/react/performance/optimistic-ui/)
- [Google UX Research on Latency](https://research.google/pubs/pub40801/)
