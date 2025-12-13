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

### Problema detectado (técnico)
- Migración big-bang genera downtime y riesgo de pérdida de datos.
- Sin dual read/write ni backfill validado, se pierden transacciones en transición.
- Sin flags/cohortes, el cutover es todo-o-nada y difícil de revertir.

### Escenario de Negocio

> *"Como banco, debo migrar 2M de usuarios a nueva plataforma sin cortar servicio."*

### Incidentes reportados
- **Migraciones sin dual-write:** Provocaron inconsistencias y reprocesos.
- **Cortes nocturnos mal planificados:** Fallaron por falta de rollback y monitoreo.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Zero-downtime migrations | Global | Dual read/write con cutover por cohortes reduce riesgo. |
| Postmortems de migración | Varios | Big bang sin backfill/validación generó pérdidas y downtime. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; cambios de backend sin plan causan fallas. |

**Resumen global**
- Dual read/write con backfill validado y cutover por cohortes minimiza downtime y riesgo; big bang es un antipatrón.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Backfill | Datos consistentes (checksums/muestras) | Backend/Data |
| Dual read/write | Sin pérdida de operaciones durante transición | QA/Backend |
| Cutover | Flags/cohortes y rollback funcionan | Móvil/QA |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Ventanas | Ejecución en horarios de bajo tráfico | Menor impacto |
| Comunicación | Avisos a soporte/negocio y monitoreo en vivo | Transparencia |
| Soporte | Runbooks de rollback y puntos de control | Respuesta rápida |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Monitoreo | Errores/latencia/consistencia tras cutover | Observabilidad |
| Carga | Throttling en backfill para no saturar | Estabilidad |
| Logs | Auditoría de cohortes migradas | Trazabilidad |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Migrar 2M usuarios sin downtime ni pérdida. |
| Opciones evaluadas | Big bang; ventana única; dual read/write con cohortes y rollback. |
| Decisión | Dual read/write, backfill validado, cutover por cohortes con flags y rollback. |
| Consecuencias | Mayor complejidad operativa; requiere monitoreo y scripts de reconciliación. |
| Riesgos aceptados | Consistencia eventual durante transición; costo de operar dos rutas. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Downtime | 0 | Crítico si ocurre | Continuidad |
| Pérdida de datos | 0 (checksums) | Crítico si >0 | Integridad |
| Errores post-cutover | Tendencia a la baja | Warning si sube | Estabilidad |
| Tiempo de migración | Dentro de ventana planificada | Alerta si se extiende | Operación |

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
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
