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

### Escenario de Negocio

> *"Como equipo, un proveedor cerró en 30 días; debemos migrar sin romper producción."*

SDKs deprecated dejan de funcionar o recibir soporte; se necesita migración acelerada y segura.

### Evidencia de Industria

- **Incidentes de terceros:** Cortes por deprecación sin migración a tiempo.
- **Buenas prácticas:** Dual stack y cutover gradual con flags.

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
