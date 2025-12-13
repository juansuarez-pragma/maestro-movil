# Caso 100: El Día que Apagamos el Legacy
## Checklist de Decommissioning Seguro

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | decommissioning, apagado legacy, checklist, riesgo |
| **Patrón Técnico** | Decommission Plan, Sunset Checklist, Rollback Preparedness |
| **Stack Seleccionado** | Flutter (flags para rutas legacy) + backend de sunset + monitoreo/rollback |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Apagar legacy sin checklist causa caídas y pérdida de datos.
- Sin rollback probado, un fallo prolonga downtime.
- Rutas/flags legacy residuales mantienen dependencias ocultas.

### Escenario de Negocio

> *"Como organización, queremos apagar el legacy sin dejar usuarios bloqueados."*

### Incidentes reportados
- **Sunsets abruptos:** Provocaron downtime y reversiones desordenadas.
- **Dependencias ocultas:** Sistemas seguían llamando al legacy tras el apagado.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Postmortems de apagado | Global | Falta de checklist y rollback causó incidentes. |
| Buenas prácticas de sunset | Global | Validar paridad, limpiar flags y monitorear en cutover. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; apagados mal gestionados exponen riesgo operacional. |

**Resumen global**
- Decommissioning seguro requiere checklist, monitoreo y rollback ensayado; apagar sin plan es altamente riesgoso.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Operacional** | Corte de servicio, datos inaccesibles |
| **Reputacional** | Pérdida de confianza si el apagado falla |
| **Legal** | Retención de datos/registros mal gestionada |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Apagar sin validaciones ni rollback | **INADECUADO:** Altísimo riesgo. |
| **ACEPTABLE** | Plan básico de apagado, sin monitoreo claro | **MEJORA:** Menos riesgo, pero sin detección temprana. |
| **ENTERPRISE** | **Checklist completa:** validación de paridad, flags removidos, backups, monitoreo en tiempo real, rollback plan, comunicación y soporte | **ÓPTIMO:** Apagado controlado y reversible. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Verificar que no hay tráfico/usuarios en legacy. Remover flags y rutas. Respaldar datos y accesos. Monitoreo intensivo durante cutover. Plan de rollback probado. Comunicación a usuarios/soporte. |
| **Restricciones Duras (NO permite)** | **Rollback inexistente:** Debe estar listo. **Datos residuales:** Gestionar retención/archivado. **Dependencias ocultas:** Requiere inventario completo. |
| **Criterio de Selección** | Checklist con dueños y tiempos; validaciones previas; monitoreo y equipo de guardia; rollback ensayado. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Paridad | Funcionalidad completa en nuevo sistema | QA/Producto |
| Tráfico | Sin requests al legacy antes del apagado | DevOps/SRE |
| Rollback | Ejercicio/prueba del plan de reversión | SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Comunicación | Avisos a usuarios/soporte con ventana | Transparencia |
| Soporte | Guardias y runbook de incidentes | Tiempo de respuesta |
| Flags | Remover/limpiar rutas legacy | Deuda controlada |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Backups | Backup/restore probados antes de apagar | Seguridad |
| Observabilidad | Monitoreo en tiempo real y alertas | MTTR bajo |
| Retención | Plan de datos/registros post-sunset | Cumplimiento |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Apagar legacy sin corte ni pérdida de datos. |
| Opciones evaluadas | Apagado directo; plan básico; checklist completa con rollback. |
| Decisión | Checklist de decommission con validaciones, monitoreo y rollback probado. |
| Consecuencias | Mayor esfuerzo de preparación y coordinación. |
| Riesgos aceptados | Dependencias ocultas no detectadas; costo de guardias/monitoreo intensivo. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Downtime en apagado | 0 | Crítico si ocurre | Continuidad |
| Requests a legacy post-cutover | 0 | Crítico si hay | Limpieza |
| Incidentes post-apagado | Tendencia a la baja | Warning si sube | Estabilidad |
| Ejercicios de rollback | Ejecutados y documentados | Alerta si faltan | Preparación |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Decommissioning | Apagado definitivo de un sistema. |
| Sunset | Proceso de retiro planificado. |
| Rollback | Reversión a estado previo si falla. |
| Backup/Restore | Copia y recuperación de datos. |
| Paridad funcional | Validación de que el nuevo sistema cubre funcionalidades previas. |

---

## Referencias

- [Sunset and Decommission Patterns](https://martinfowler.com/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
