# Caso 74: Feature Flags como Guardarraíl de Release
## Controlar Impacto sin Retrasar Deploys

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | feature flags, rollout, kill switch, control de cambios |
| **Patrón Técnico** | Feature Flag Governance, Kill Switch, Targeting |
| **Stack Seleccionado** | Flutter + Flags SDK (LaunchDarkly/ConfigCat/Custom) + Riverpod gating + analytics |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo, quiero liberar código a producción y controlar el impacto con flags sin retrasar deploys."*

Sin flags bien gobernados, los releases quedan bloqueados o arriesgan a todos los usuarios.

### Evidencia de Industria

- **Progressive delivery:** Flags son esenciales para rollout y rollback rápido.
- **Incidentes:** Falta de kill switch prolonga problemas en producción.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico/UX** | Impacto masivo ante bugs sin kill switch |
| **Técnico** | Deuda de flags zombies, lógica compleja y opaca |
| **Operacional** | Rollouts desordenados, difícil trazabilidad |

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
| **Capacidades (SÍ permite)** | Activar/desactivar features por segmento/porcentaje. Kill switch inmediato. Expirar y limpiar flags. Métricas de exposición/impacto. Defaults seguros en cliente. |
| **Restricciones Duras (NO permite)** | **Gobernanza ausente:** Flags obsoletos generan deuda. **Latencia de fetch:** Requiere cache y defaults. **Seguridad:** No exponer lógica crítica en flags manipulables. |
| **Criterio de Selección** | SDK con targeting y métricas; Riverpod para gating tipado; proceso de expiración y cleanup; kill switch obligatorio. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Kill Switch | Flag para apagar una función inmediatamente. |
| Targeting | Selección de audiencia para un flag. |
| Expiración | Fecha/lógica para retirar flags antiguos. |
| Zombie flag | Flag que ya no controla nada y queda en el código. |
| Progressive delivery | Deploy rápido, activación gradual con flags. |

---

## Referencias

- [Feature Toggles (Fowler)](https://martinfowler.com/articles/feature-toggles.html)
- [LaunchDarkly Best Practices](https://docs.launchdarkly.com/home/flags/best-practices)
