# Caso 52: Feature Flags en Banca
## Lanzar Funciones a 1% de Usuarios sin Deploy

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | feature flags, rollout gradual, experimentación, toggles |
| **Patrón Técnico** | Feature Toggles, Progressive Delivery, Remote Config |
| **Stack Seleccionado** | Flutter + Remote Config/Flags SDK (LaunchDarkly/ConfigCat/Custom) + Riverpod |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo, quiero habilitar una función a 1% sin publicar nueva versión y con rollback inmediato."*

Sin flags, cada cambio requiere release; sin control, se arriesgan regresiones a toda la base.

### Evidencia de Industria

- **Progressive delivery:** Reduce riesgo de despliegues; usado por empresas de alto tráfico.
- **Banca/fintech:** Flags permiten activar features tras aprobar controles.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico/Reputacional** | Bugs impactan a todos sin rollback rápido |
| **Técnico** | Flags en código sin gobernanza causan deuda y comportamiento impredecible |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Flags hardcodeados y sin expiración | **INADECUADO:** No hay control remoto ni limpieza. |
| **ACEPTABLE** | Remote config básico sin segmentación | **MEJORA:** Permite toggle, pero sin rollout gradual ni targeting. |
| **ENTERPRISE** | **Feature flags gestionados:** targeting, porcentajes, expiración, métricas, kill switch | **ÓPTIMO:** Rollout seguro, rollback instantáneo, gobernanza. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Rollout por porcentaje, segmento, dispositivo. Kill switch inmediato. Expiración de flags. Métricas de exposición y conversión. Evaluación local con cache y fallback offline. |
| **Restricciones Duras (NO permite)** | **Sin cleanup:** Flags viejos deben retirarse. **Latencia de fetch:** Requiere cache inicial y defaults seguros. **Seguridad:** Flags no deben exponer lógica sensible. |
| **Criterio de Selección** | SDK con targeting y gobernanza; Riverpod para exponer flags tipados; política de expiración y limpieza en cada release. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Feature Flag | Condicional remoto que activa/desactiva funcionalidad. |
| Progressive Delivery | Desplegar gradualmente a segmentos/porcentajes. |
| Kill Switch | Mecanismo de apagado inmediato ante incidentes. |
| Targeting | Seleccionar audiencias específicas para un flag. |
| Expiración de flags | Retirar flags antiguos para evitar deuda. |

---

## Referencias

- [Feature Toggles (Fowler)](https://martinfowler.com/articles/feature-toggles.html)
- [LaunchDarkly Docs](https://docs.launchdarkly.com/)
