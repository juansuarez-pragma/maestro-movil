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

### Escenario de Negocio

> *"Como organización, queremos apagar el legacy sin dejar usuarios bloqueados."*

Apagar sin checklist causa caídas y pérdida de datos; es la fase más riesgosa de la migración.

### Evidencia de Industria

- **Sunsets exitosos:** Plan con validaciones y rollback listo.
- **Incidentes:** Apagados abruptos provocan downtime y rollback desordenado.

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
| **Restricciones Duras (NO permite)** | **Rollback inexistente:** Debe estar listo. **Datos residuales:** Hay que gestionar retención/archivado. **Dependencias ocultas:** Requiere inventario completo. |
| **Criterio de Selección** | Checklist con dueños y tiempos; validaciones previas; monitoreo y equipo de guardia; rollback ensayado. |

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
