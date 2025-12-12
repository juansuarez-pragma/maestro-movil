# Caso 78: Dark Launch
## Desplegar en Producción sin Activar la Funcionalidad

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | dark launch, progressive delivery, feature flags, riesgo |
| **Patrón Técnico** | Dark Launch, Shadow Traffic, Dual Execution |
| **Stack Seleccionado** | Flutter + flags/experiments SDK + duplicación de requests (shadow) + observabilidad |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo, quiero desplegar una funcionalidad y probarla en producción sin exponerla a usuarios."*

Sin dark launch, el primer uso real ya impacta usuarios; aumenta riesgo.

### Evidencia de Industria

- **Shadow traffic/dark launches:** Reducen riesgo antes del go-live.
- **Progressive delivery:** Permite validar en producción con flags.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Técnico** | Bugs ocultos salen al go-live sin pruebas en prod |
| **UX** | Impacto directo si la funcionalidad falla al activarse |
| **Operacional** | Difícil rollback si no hay flags/killswitch |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Desplegar y activar inmediatamente | **INADECUADO:** Riesgo alto. |
| **ACEPTABLE** | Toggle manual en prod sin shadow | **MEJORA:** Algo de control, pero sin validación previa. |
| **ENTERPRISE** | **Dark launch + shadow:** desplegar apagado, probar con shadow traffic/QA interna, observabilidad, luego activar gradualmente | **ÓPTIMO:** Riesgo reducido y rollback rápido. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Deploy con feature flag apagado. Enviar shadow traffic a la nueva ruta sin afectar usuario. Monitorear métricas y errores. Activar gradualmente a segmentos. Rollback inmediato via flag. |
| **Restricciones Duras (NO permite)** | **Shadow incompleto:** No replica 100% de casos reales. **Costo:** Duplicar requests aumenta carga. **Privacidad:** Debe cumplir con tratamiento de datos aun en shadow. |
| **Criterio de Selección** | Flags con killswitch; shadow para rutas críticas; monitoreo antes de habilitar; activación gradual. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Dark Launch | Desplegar funcionalidad oculta en producción. |
| Shadow Traffic | Duplicar tráfico a nueva funcionalidad sin afectar respuesta al usuario. |
| Flag/Kill Switch | Control remoto para activar/desactivar features. |
| Progressive Delivery | Activar features gradualmente con control. |
| Rollback | Apagar o revertir rápidamente ante fallas. |

---

## Referencias

- [Shadow Launch Patterns](https://martinfowler.com/articles/feature-toggles.html)
- [Progressive Delivery](https://launchdarkly.com/blog/progressive-delivery/)
