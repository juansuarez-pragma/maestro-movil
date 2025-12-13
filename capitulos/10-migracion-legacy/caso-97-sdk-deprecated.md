# Caso 97: El Tercero que Cerró
## Reemplazar SDK Deprecated en 30 Días

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | sdk deprecated, terceros, migración urgente, riesgo |
| **Patrón Técnico** | Strangler for SDK, Dual Stack, Feature Flags |
| **Stack Seleccionado** | Flutter + nuevo SDK en paralelo + feature flags + observabilidad comparativa |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- El proveedor cierra en 30 días; SDK queda sin soporte y puede fallar.
- Migración big bang en la última semana es riesgosa sin rollback.
- Sin telemetría comparativa, el cutover es ciego.

### Escenario de Negocio

> *"Como equipo, un proveedor cerró en 30 días; debemos migrar sin romper producción."*

### Incidentes reportados
- **Deprecaciones abruptas:** Cortes por no migrar a tiempo.
- **Fallas en SDK viejo:** Sin soporte, se volvieron vector de riesgo/bugs.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Migraciones urgentes | Global | Dual stack con flags reduce riesgo en ventanas cortas. |
| Postmortems de terceros | Varios | Big bang tardío generó outages. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; dependencias obsoletas son comunes. |

**Resumen global**
- Dual stack + flags + telemetría comparativa permite migrar rápido y revertir; dejar un SDK EOL es un riesgo crítico.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Operacional** | Caída de funcionalidades dependientes del SDK |
| **Seguridad/Legal** | SDK sin soporte puede ser vector de riesgo |
| **Productivo** | Migración apresurada con regresiones |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Reemplazo big-bang en última semana | **INADECUADO:** Riesgo altísimo. |
| **ACEPTABLE** | Migración directa con un release | **MEJORA:** Menos riesgo, pero sin rollback fácil. |
| **ENTERPRISE** | **Dual stack + flags:** integrar nuevo SDK en paralelo, wrapper común, flags para cutover por cohorte, telemetría comparativa | **ÓPTIMO:** Migración rápida y reversible. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Wrap común para ambos SDKs. Medir errores/latencia por SDK. Activar nuevo SDK a cohortes vía flags. Rollback inmediato al viejo hasta su sunset. |
| **Restricciones Duras (NO permite)** | **Tiempo limitado:** 30 días obliga a priorizar. **SDK viejo sin soporte:** Puede fallar; plan de contingencia necesario. **Costo:** Doble mantenimiento temporal. |
| **Criterio de Selección** | Wrapper unificado, dual stack, flags y telemetría comparativa, plan de sunset y comunicación. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Comparativa | Errores/latencia viejo vs nuevo | Móvil/SRE |
| Cutover | Flags por cohorte y rollback operan | QA/Móvil |
| Seguridad | Dependencias y permisos del nuevo SDK | Seguridad |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Mensajes | Transparencia mínima; evitar impacto visible | UX |
| Cohortes | Activar gradualmente para acotar riesgo | Control |
| Soporte | Runbook de rollback rápido | Respuesta |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Telemetría | Eventos `sdk.migration.*` con resultados | Observabilidad |
| Sunset | Fecha límite y checklist de retiro del SDK viejo | Disciplina |
| Dependencias | Revisar licencias y permisos del nuevo SDK | Compliance |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Migrar un SDK EOL en 30 días sin romper producción. |
| Opciones evaluadas | Big bang tardío; migración directa sin rollback; dual stack con flags y telemetría. |
| Decisión | Dual stack con wrapper común, flags por cohorte y telemetría comparativa; rollback hasta sunset del viejo. |
| Consecuencias | Doble mantenimiento temporal; mayor esfuerzo de observabilidad. |
| Riesgos aceptados | Dependencia temporal del SDK viejo; esfuerzo operativo. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Errores por SDK | ↓ en nuevo vs viejo | Warning si no | Estabilidad |
| Tiempo de migración | ≤ 30 días | Crítico si se extiende | Cumplimiento |
| Rollback time | Minutos vía flags | Crítico si tarda | Riesgo acotado |
| Incidentes de dependencia EOL | 0 | Crítico si >0 | Seguridad |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| SDK deprecated | SDK en fin de vida sin soporte. |
| Dual stack | Ejecutar dos implementaciones en paralelo. |
| Cutover | Cambiar definitivamente a la nueva implementación. |
| Telemetría comparativa | Métricas lado a lado para decidir cutover. |
| Sunset plan | Plan de retiro del SDK viejo. |

---

## Referencias

- [Strangler Pattern](https://martinfowler.com/bliki/StranglerFigApplication.html)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
