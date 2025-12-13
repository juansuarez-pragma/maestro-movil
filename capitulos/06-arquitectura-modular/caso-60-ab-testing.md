# Caso 60: [A/B Testing](#term-a-b-testing "[Experimento](#term-experimento "Configuración que asigna variantes y mide métricas.") controlado con variantes A y B.") Arquitectónico
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

### Problema detectado (técnico)
- Hardcodear variantes obliga a publicar y no permite rollback controlado.
- Sin tracking, no se sabe qué variante ve cada usuario ni su impacto.
- Experimentos sin cleanup dejan código muerto y deuda.

### Escenario de Negocio

> *"Como equipo, quiero probar dos UIs sin mantener ramas separadas ni romper consistencia."*

### Incidentes reportados
- **Experimentación en producto:** Mejora conversión; experimentos sin expiración han roto UX meses después.
- **Banca/fintech:** Necesitan gobernanza y privacidad para tracking de variantes.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Product experimentation | Global | A/B bien gobernado mejora métricas; sin cleanup genera deuda. |
| Postmortems de flags | Varios | Falta de expiración provocó comportamientos inconsistentes. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; configuración/flags suele carecer de controles. |

**Resumen global**
- Framework de experimentos + flags tipados habilita medición y rollback seguro; requiere expiración, privacidad y catálogo de variantes.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit/Contract | Asignación y defaults de variantes | Móvil/CI |
| Integration (CI) | Tracking por variante y rollback vía flag | QA/Móvil |
| Observabilidad | Métricas `exp.*` (exposición, conversión, error) | Móvil/Data |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Consistencia | Un usuario siempre recibe la misma variante | Evita confusión |
| Mensajes | No revelar experimentos al usuario | Profesional |
| Expiración | TTL y limpieza de código al cerrar experimento | Sin deuda |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Gobernanza | Catálogo de experimentos y dueños | Control |
| Privacidad | Anonimizar IDs y respetar consentimientos | Cumplimiento |
| Rollback | Kill switch si métricas empeoran | Riesgo acotado |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Probar variantes sin ramas separadas ni pérdida de control. |
| Opciones evaluadas | Hardcode; flags simples; framework de experimentos con métricas/cleanup. |
| Decisión | Framework de experimentos con asignación estable, métricas y expiración/cleanup. |
| Consecuencias | Requiere catálogo y gobernanza; integra analytics/privacidad. |
| Riesgos aceptados | Sobrecarga de configuración; dependencia del SDK. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Tiempo de rollback de variante | Inmediato vía flag | Crítico si > minutos | Riesgo acotado |
| Debt de experimentos | 0 experimentos vencidos sin cleanup | Warning si hay | Código limpio |
| Conversión por variante | Medida y comparada | Alerta si cae | Decisiones basadas en datos |
| Incidentes por variante | 0 | Crítico si >0 | Calidad |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-a-b-testing"></a>A/B Testing | Experimento controlado con variantes A y B. |
| <a id="term-experimento"></a>Experimento | Configuración que asigna variantes y mide métricas. |
| <a id="term-variante"></a>Variante | Diferente UI/feature bajo prueba. |
| <a id="term-cleanup"></a>Cleanup | Remover código de experimentos concluidos. |
| <a id="term-assignment"></a>Assignment | Lógica que decide qué variante recibe un usuario. |

---

## Referencias

- [A/B Testing Best Practices](https://www.optimizely.com/optimization-glossary/ab-testing/)
- [Feature Flags/Experimentation](https://martinfowler.com/articles/feature-toggles.html)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
