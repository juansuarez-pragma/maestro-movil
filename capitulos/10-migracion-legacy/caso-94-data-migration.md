# Caso 94: Data Migration Nocturna
## Mover 2M de Usuarios sin Downtime

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | migración de datos, downtime, batch, backfill |
| **Patrón Técnico** | Dual Write/Read, Backfill, Phased Cutover |
| **Stack Seleccionado** | Flutter (feature flags para nueva API) + backend dual-read/write + jobs batch |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como banco, debo migrar 2M de usuarios a nueva plataforma sin cortar servicio."*

Migraciones big-bang causan downtime; se requiere cutover gradual y seguro.

### Evidencia de Industria

- **Migraciones exitosas:** Dual write/read, backfill y cutover por cohortes.
- **Incidentes:** Migraciones sin plan generan pérdidas y downtime prolongado.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Operacional** | Downtime, inconsistencias |
| **Económico** | Transacciones fallidas durante migración |
| **Reputacional** | Usuarios sin servicio, mala prensa |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Migración big-bang con downtime | **INADECUADO:** Alto riesgo. |
| **ACEPTABLE** | Backfill y corte en ventana nocturna | **MEJORA:** Menor riesgo, pero sin rollback fácil. |
| **ENTERPRISE** | **Dual read/write + cutover gradual:** backfill, validar, cortar por cohortes, flags en cliente, rollback rápido | **ÓPTIMO:** Minimiza downtime y riesgo. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Dual write a sistemas viejo/nuevo. Dual read con preferencia al nuevo tras backfill validado. Cutover por cohortes con flags. Rollback a viejo si falla. Monitoreo de errores y métricas. |
| **Restricciones Duras (NO permite)** | **Costo:** Dual write aumenta complejidad. **Consistencia eventual:** Requiere reconciliación. **Timeframes:** Backfill grande necesita ventanas y control de carga. |
| **Criterio de Selección** | Dual read/write, backfill con verificación, cutover por cohortes, flags en cliente, rollback plan. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Backfill | Poblar datos históricos en el nuevo sistema. |
| Dual write/read | Escribir/leer en ambos sistemas durante transición. |
| Cutover | Cambio definitivo al nuevo sistema. |
| Cohorte | Grupo de usuarios migrados en un lote. |
| Rollback | Volver al sistema anterior rápidamente. |

---

## Referencias

- [Zero-Downtime Migrations](https://martinfowler.com/articles/evodb.html)
- [Blue-Green/Canary Database Migrations](https://aws.amazon.com/blogs/database/)
