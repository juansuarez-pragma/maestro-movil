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

### Problema detectado (técnico)
- Optimistic UI sin acciones compensatorias deja estados corruptos si el backend rechaza (stock, permisos, versión).
- Sin cola persistente e idempotencia, reintentos pueden duplicar o perder acciones.
- Feedback pobre (solo snackbar) confunde al usuario y genera tickets.

### Escenario de Negocio

> *"Como usuario, quiero ver mis cambios al instante, pero si fallan, que se reviertan sin dejar datos corruptos."*

### Incidentes reportados
- **eCommerce 2022:** UI mostraba saldo actualizado; backend rechazó débito → tickets masivos por saldo “fantasma”.
- **Google UX Research:** Retroalimentación inmediata reduce abandono, pero requiere manejo robusto de errores/rollback.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Google UX (latencia) | Estudios UX | Feedback inmediato reduce abandono. |
| Casos internos ecommerce (2022) | Apps retail | Estados “fantasma” generan tickets y reclamos. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; manejo de errores/estado es fuente frecuente de bugs. |

**Resumen global**
- Optimistic sin rollback consistente provoca soporte y pérdida de confianza.
- UX ágil es clave, pero debe ir con compensaciones e idempotencia.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Acciones optimistas aplican y revertir sin mutar estado previo | Equipo móvil, CI |
| Integration (CI) | Rechazo backend ejecuta acción compensatoria y sincroniza estado | Móvil/Backend, CI + staging |
| Seguridad/consistencia | Idempotency-Key evita duplicar en reintentos; cola persiste offline breve | QA/Seguridad |
| Observabilidad | Eventos `optimistic.*` con motivo y resultado | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Feedback | Mostrar estado optimista y, si falla, revertir con explicación | Transparencia al usuario |
| Reintentos | Backoff y límite; permitir “reintentar” con misma acción | Control de carga y UX |
| Cola | Persistir acciones en SQLite; limpiar al confirmar/rollback | Robustez offline breve |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Idempotencia | `Idempotency-Key` por acción; conservar hasta confirmación | Evita duplicados |
| Compensación | Definir acción inversa por tipo de operación | Consistencia |
| Versionado | OCC en backend para detectar conflictos | Evita overwrites |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Optimistic sin rollback genera estados falsos y tickets. |
| Opciones evaluadas | Solo UI optimista; refresh completo; optimista + sagas/compensación. |
| Decisión | Optimistic + cola persistente + compensación + idempotencia. |
| Consecuencias | Más lógica de estado y manejo de errores; dependencia de backend para OCC. |
| Riesgos aceptados | Offline largo puede expirar acciones; UX depende de claridad del rollback. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Estados “fantasma” | 0 incidentes | Crítico si > 0 | Confianza y consistencia |
| Tiempo de feedback | p95 < 200 ms para UI optimista | Warning si sube | UX ágil |
| Reversiones exitosas | > 99% de fallos con rollback correcto | Alerta si baja | Consistencia garantizada |
| Tickets por discrepancias | ↓ ≥ 50% | Alerta si no baja | Soporte controlado |
| Retrys duplicados | 0 cargos/acciones duplicadas | Crítico si > 0 | Previene reprocesos |

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
| Idempotency-Key | Identificador único para evitar duplicados en reintentos. |

---

## Referencias

- [Fowler - Saga Pattern](https://microservices.io/patterns/data/saga.html)
- [Optimistic UI - Apollo Docs](https://www.apollographql.com/docs/react/performance/optimistic-ui/)
- [Google UX Research on Latency](https://research.google/pubs/pub40801/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
