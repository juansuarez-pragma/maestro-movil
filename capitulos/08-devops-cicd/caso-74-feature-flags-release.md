# Caso 74: Feature Flags como Guardarraíl de Release
## Controlar Impacto sin Retrasar Deploys

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | feature flags, rollout, kill switch, control de cambios |
| **Patrón Técnico** | Feature Flag Governance, [Kill Switch](#term-kill-switch "Flag para apagar una función inmediatamente."), [Targeting](#term-targeting "Selección de audiencia para un flag.") |
| **Stack Seleccionado** | Flutter + Flags SDK (LaunchDarkly/ConfigCat/Custom) + Riverpod gating + analytics |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Releases se bloquean por features a medias o despliegan riesgos a todos sin control.
- Sin kill switch, un bug en producción tarda horas en mitigarse.
- Flags sin expiración se vuelven deuda y opacan la lógica.

### Escenario de Negocio

> *"Como equipo, quiero liberar código a producción y controlar el impacto con flags sin retrasar deploys."*

### Incidentes reportados
- **[Progressive delivery](#term-progressive-delivery "Deploy rápido, activación gradual con flags."):** Falta de kill switch prolongó incidentes en producción.
- **Equipos sin gobernanza:** Flags zombies generaron comportamiento impredecible meses después.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Progressive delivery reports | Global | Flags con kill switch reducen MTTR en incidentes. |
| Postmortems de flags | Varios | [Expiración](#term-expiracion "Fecha/lógica para retirar flags antiguos.") ausente causa deuda y bugs tardíos. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; gobernanza de configuración/flags es débil. |

**Resumen global**
- Flags gobernados (kill switch, expiración, targeting, métricas) permiten deploy rápido y mitigación; flags sin control crean deuda y riesgo.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico/UX** | Impacto masivo ante bugs sin kill switch |
| **Técnico** | Deuda de flags zombies, lógica compleja |
| **Operacional** | Rollouts desordenados, trazabilidad baja |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Flags hardcodeados, sin limpieza | **INADECUADO:** Riesgo y deuda. |
| **ACEPTABLE** | Flags remotos sin kill switch ni expiración | **MEJORA:** Control parcial, pero sin gobernanza. |
| **ENTERPRISE** | **Flags gobernados:** kill switch, expiración, targeting, métricas, checklist de limpieza en cada release | **ÓPTIMO:** Control fino y riesgo acotado. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Activar/desactivar por segmento/porcentaje. Kill switch inmediato. Expirar y limpiar flags. Métricas de exposición/impacto. Defaults seguros en cliente. |
| **Restricciones Duras (NO permite)** | **Gobernanza ausente:** Flags obsoletos generan deuda. **Latencia de fetch:** Requiere cache y defaults. **Seguridad:** No exponer lógica crítica en flags manipulables. |
| **Criterio de Selección** | SDK con targeting y métricas; Riverpod para gating tipado; proceso de expiración/cleanup; kill switch obligatorio. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit/Contract | Defaults seguros y tipos de flags | Móvil/CI |
| Integration (CI) | Kill switch y targeting por cohorte/porcentaje | QA/Móvil |
| Observabilidad | Métricas de exposición/conversión/errores por flag | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Mensajes | Flags no deben exponer roadmap | Privacidad |
| Rollout | Progresivo con cohorts y rollback rápido | Control de riesgo |
| Expiración | TTL y limpieza en cada release | Menos deuda |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Gobernanza | Dueño por flag y catálogo central | Trazabilidad |
| Seguridad | Flags solo gating, no secretos | Protección |
| Auditoría | Logs de cambios y toggles | Cumplimiento |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Controlar impacto de features sin frenar deploys. |
| Opciones evaluadas | Flags hardcodeados; flags remotos sin gobernanza; flags gobernados con kill switch/expiración/métricas. |
| Decisión | Flags gobernados con kill switch, expiración y métricas. |
| Consecuencias | Requiere proceso de limpieza y catálogo; dependencia del SDK. |
| Riesgos aceptados | Complejidad de configuración; retrasos de fetch si se abusa. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Tiempo de rollback | Inmediato vía kill switch | Crítico si > minutos | Riesgo acotado |
| Flags zombies | 0 por release | Warning si hay | Código limpio |
| Incidentes por flag | Tendencia a la baja | Crítico si sube | Estabilidad |
| Cobertura de targeting | Medida por cohorte | Alerta si inconsistente | Control |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-kill-switch"></a>Kill Switch | Flag para apagar una función inmediatamente. |
| <a id="term-targeting"></a>Targeting | Selección de audiencia para un flag. |
| <a id="term-expiracion"></a>Expiración | Fecha/lógica para retirar flags antiguos. |
| <a id="term-zombie-flag"></a>Zombie flag | Flag que ya no controla nada y queda en el código. |
| <a id="term-progressive-delivery"></a>Progressive delivery | Deploy rápido, activación gradual con flags. |

---

## Referencias

- [Feature Toggles (Fowler)](https://martinfowler.com/articles/feature-toggles.html)
- [LaunchDarkly Best Practices](https://docs.launchdarkly.com/home/flags/best-practices)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
