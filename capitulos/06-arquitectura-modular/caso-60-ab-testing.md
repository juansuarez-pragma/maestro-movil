# Caso 60: A/B Testing Arquitectónico
## Servir Diferentes UIs desde el Mismo Código

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | A/B testing, experimentación, UI variantes, flags |
| **Patrón Técnico** | Experimentation Framework, Feature Flags, Variant Routing |
| **Stack Seleccionado** | Flutter + Remote Flags/Experiments SDK + Riverpod para gating + analytics |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo, quiero probar dos UIs sin mantener ramas separadas ni romper consistencia."*

Sin experimentos controlados, los cambios se despliegan a todos y es difícil medir impacto.

### Evidencia de Industria

- **Experimentation en producto:** Mejora conversión y UX; flags permiten rollout seguro.
- **Riesgos:** Experimentos mal gobernados generan deuda y UX inconsistente.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico/UX** | Variantes malas impactan conversión |
| **Técnico** | Código muerto si no se limpian experimentos |
| **Operacional** | Difícil rastrear qué variante vio cada usuario sin analytics |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Hardcodear variantes y publicar | **INADECUADO:** Sin control ni medición. |
| **ACEPTABLE** | Flags simples sin tracking | **MEJORA:** Permite variantes, pero sin análisis riguroso. |
| **ENTERPRISE** | **Framework de experimentos:** asignación controlada, métricas por variante, expiración y cleanup, flags tipados | **ÓPTIMO:** Medición confiable y rollout seguro. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Asignar variantes por usuario/segmento. Medir métricas (conversión, tiempo) por variante. Expirar y limpiar experimentos. Combinar con feature flags para rollback. |
| **Restricciones Duras (NO permite)** | **Sin governance:** Riesgo de proliferación de flags. **Latencia de fetch:** Necesita defaults y cache. **Privacidad:** Cumplir GDPR/consentimientos para tracking. |
| **Criterio de Selección** | SDK de experiments/flags; Riverpod para exponer variantes; analytics integrados; política de expiración/cleanup. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| A/B Testing | Experimento controlado con variantes A y B. |
| Experimento | Configuración que asigna variantes y mide métricas. |
| Variante | Diferente UI/feature bajo prueba. |
| Cleanup | Remover código de experimentos concluidos. |
| Assignment | Lógica que decide qué variante recibe un usuario. |

---

## Referencias

- [A/B Testing Best Practices](https://www.optimizely.com/optimization-glossary/ab-testing/)
- [Feature Flags/Experimentation](https://martinfowler.com/articles/feature-toggles.html)
