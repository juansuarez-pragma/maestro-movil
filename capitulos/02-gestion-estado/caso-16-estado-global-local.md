# Caso 16: Estado Global vs Local
## El Dilema del Saldo en Tiempo Real

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | estado global, estado local, saldo en tiempo real, rendimiento |
| **Patrón Técnico** | State Partitioning, Selector Optimization, Background Refresh |
| **Stack Seleccionado** | Flutter + Riverpod Selectors + Stream caching + InheritedModel para widgets pesados |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero ver mi saldo actualizado en toda la app sin que se congele la UI."*

Un estado global mal diseñado dispara rebuilds masivos y jank; un estado demasiado local produce datos inconsistentes (saldos distintos por pantalla).

### Evidencia de Industria

- **Casos en banca 2022:** Apps con saldos desincronizados generaron cientos de tickets y desconfianza.
- **Flutter perf guides:** Recomiendan granularidad fina y selectors para minimizar rebuilds.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Datos de saldo incorrectos → llamadas al soporte, riesgos de decisiones erróneas |
| **UX** | Jank y pantallas que parpadean por rebuilds innecesarios |
| **Técnico** | Complejidad de depuración por mezcla de estado global/local inconsistente |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Un único singleton con todo el estado | **INADECUADO:** Rebuild global, difícil de testear, acoplamiento. |
| **ACEPTABLE** | Dividir estado por features sin consistencia global | **MEJORA:** Menos rebuild, pero riesgo de desalineación (saldo distinto en pantallas). |
| **ENTERPRISE** | **Partición + sincronización:** estado global para datos compartidos (saldo), estado local para UI; selectors para granularidad; refresh en background | **ÓPTIMO:** Consistencia donde importa, rendimiento controlado, testabilidad. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Mantener saldo en store global con stream cache y TTL. Usar selectors para derivar vistas mínimas sin rebuild masivo. Estado local para inputs/UI. Refresh en background con política de TTL corta y backoff. |
| **Restricciones Duras (NO permite)** | **Sin cache compartida:** Cada pantalla refrescando saldo saturará red. **Race de refresh:** Necesita deduplicación para evitar múltiples fetch simultáneos. **Consistencia fuerte:** En presencia de múltiples dispositivos, puede haber lag hasta converger. |
| **Criterio de Selección** | Riverpod por selectores y scopes; stream caching para evitar N requests; separar Domain state (saldo) de View state (filtros, toggles). |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Estado global | Datos compartidos entre pantallas (saldo, usuario). |
| Estado local | Estado efímero de UI (inputs, toggles). |
| Selector | Función que deriva una porción del estado minimizando rebuilds. |
| TTL | Tiempo máximo que un dato permanece válido antes de refrescar. |
| Stream cache | Almacenar última emisión de un stream para nuevos suscriptores. |

---

## Referencias

- [Flutter Performance Best Practices](https://docs.flutter.dev/perf/best-practices)
- [Riverpod Selectors](https://riverpod.dev/docs/concepts/providers/#select)
- [Nielsen Norman - Data Freshness UX](https://www.nngroup.com/articles/data-freshness/)
