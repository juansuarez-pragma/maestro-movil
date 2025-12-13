# Caso 73: Release Train
## Calendario de Releases Predecible en Banca

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | release train, cadencia, mobile release, gobernanza |
| **Patrón Técnico** | Release Train, Branching Strategy, Gatekeeping |
| **Stack Seleccionado** | Flutter + Git branching (trunk/hotfix) + CI con ventanas de freeze + feature flags |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Releases ad-hoc generan regresiones y falta de trazabilidad.
- Ramas largas divergen y complican merges; QA se satura sin calendario fijo.
- Sin freeze ni flags, features a medias rompen el release.

### Escenario de Negocio

> *"Como equipo, necesito releases móviles predecibles, con riesgo controlado."*

### Incidentes reportados
- **Banca/regulado:** Releases urgentes sin gates causaron incidentes y reprocesos.
- **Equipos grandes:** Divergencia de ramas provocó bloqueos y hotfixes frecuentes.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Release trains en producto | Global | Cadencia fija reduce retrabajo y alinea QA/negocio. |
| Trunk-based adoption | Global | Menos conflictos y ciclos más cortos. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; gobernanza de releases es punto débil. |

**Resumen global**
- Release trains con trunk-based, freeze corto y flags reducen riesgo y mejoran previsibilidad; releases ad-hoc incrementan retrabajo y riesgos regulatorios.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Operacional** | Releases cancelados o defectuosos |
| **Regulatorio** | Cambios sin trazabilidad pueden incumplir normativas |
| **Productivo** | Retrabajo y soporte elevado |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Releases ad-hoc sin calendario | **INADECUADO:** Impredecible, alto riesgo. |
| **ACEPTABLE** | Cadencia mensual con rama de release | **MEJORA:** Más control, pero con frozen largos. |
| **ENTERPRISE** | **Release train quinquenal/semanal:** trunk-based + flags, ventanas de freeze cortas, gates de calidad, checklist regulatorio | **ÓPTIMO:** Predecible, riesgo acotado, respuesta rápida. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Programar corte de release fijo. Usar feature flags para aislar features no listas. Gates de tests, seguridad, firmas. Hotfix branch para incidentes. |
| **Restricciones Duras (NO permite)** | **Ramas largas:** Evitar divergence; trunk-based recomendado. **Flags olvidados:** Necesitan cleanup. **Capacidad de QA:** Cadencia alta requiere automatización. |
| **Criterio de Selección** | Trunk-based + flags; calendario fijo; gates de calidad y cumplimiento; freeze corto. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Gates | Tests/seguridad/perf antes de cut | QA/Sec/Perf |
| Release candidate | Build firmado y trazable | Móvil/DevOps |
| Auditoría | Evidencia de checklist regulatorio | Cumplimiento |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Comunicaciones | Calendario visible para negocio/QA | Alineación |
| Flags | Aislar features y limpiar tras release | Deuda controlada |
| Hotfix | Proceso claro y ramas cortas | Respuesta rápida |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Branching | Trunk-based + ramas de hotfix/release candidates | Menos conflicto |
| Freeze | Ventanas cortas y definidas | Estabilidad |
| Métricas | Fallas por release, tiempo de aprobación | Mejora continua |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Releases impredecibles y riesgosos. |
| Opciones evaluadas | Ad-hoc; release mensual; release train con trunk-based y flags. |
| Decisión | Release train fijo con freeze corto, flags y gates. |
| Consecuencias | Requiere disciplina de calendario y cleanup de flags. |
| Riesgos aceptados | Overhead de coordinación; necesidad de automatización de QA. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Fallas por release | Tendencia a la baja | Crítico si sube | Calidad |
| Tiempo de ciclo release | Predecible (semanal/quincenal) | Warning si se extiende | Cadencia |
| Hotfixes fuera de tren | Disminución | Alerta si sube | Estabilidad |
| Cumplimiento regulatorio | 100% evidenciado | Crítico si falla | Cumplimiento |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Release Train | Cadencia fija de releases, listo o no el feature. |
| Freeze | Ventana donde se limitan cambios para estabilizar. |
| Gate | Criterio obligatorio (tests, seguridad) antes de liberar. |
| Hotfix | Corrección urgente fuera del ciclo normal. |
| Trunk-based | Estrategia con main como rama fuente y ciclos cortos. |

---

## Referencias

- [Release Train Model](https://martinfowler.com/bliki/ReleaseTrain.html)
- [Trunk-Based Development](https://trunkbaseddevelopment.com/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
