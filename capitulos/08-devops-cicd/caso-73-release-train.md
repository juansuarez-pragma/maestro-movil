# Caso 73: Release Train
## Calendario de Releases Predecible en Banca

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | release train, cadencia, mobile release, gobernanza |
| **Patrón Técnico** | Release Train, Branching Strategy, Gatekeeping |
| **Stack Seleccionado** | Flutter + Git branching (trunk/hotfix) + CI con ventanas de freeze + feature flags |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo, necesito releases móviles predecibles, con riesgo controlado."*

Sin cadencia y gobernanza, los releases son caóticos y regresan cambios críticos.

### Evidencia de Industria

- **Release trains:** Adoptados en grandes productos para previsibilidad.
- **Banca:** Riesgo regulatorio exige control de cambios.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Operacional** | Releases cancelados o defectuosos |
| **Regulatorio** | Cambios sin trazabilidad pueden incumplir normativas |
| **Productivo** | Retrabajo y soporte elevado |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Releases ad-hoc sin calendario | **INADECUADO:** Impredecible, alto riesgo. |
| **ACEPTABLE** | Cadencia mensual con rama de release | **MEJORA:** Más control, pero con frozen largos. |
| **ENTERPRISE** | **Release train quinquenal/semanal:** trunk-based + flags, ventanas de freeze cortas, gates de calidad, checklist regulatorio | **ÓPTIMO:** Predecible, riesgo acotado, respuesta rápida. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Programar corte de release fijo. Usar feature flags para aislar features no listas. Gates de tests, seguridad, firmas. Hotfix branch para incidentes. |
| **Restricciones Duras (NO permite)** | **Ramas largas:** Evitar divergence; trunk-based recomendado. **Flags olvidados:** Necesitan cleanup. **Capacidad de QA:** Cadencia alta requiere automatización. |
| **Criterio de Selección** | Trunk-based + flags; calendario fijo; gates de calidad y cumplimiento; freeze corto. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Release Train | Cadencia fija de releases, listo o no el feature. |
| Freeze | Ventana donde se limitan cambios para estabilizar. |
| Gate | Criterio obligatorio (tests, seguridad) antes de liberar. |
| Hotfix | Corrección urgente fuera del ciclo normal. |
| Trunk-based | Estrategia con main como rama fuente y ciclos cortos. |

---

## Referencias

- [Release Train Model](https://martinfowler.com/bliki/ReleaseTrain.html)
- [Trunk-Based Development](https://trunkbaseddevelopment.com/)
