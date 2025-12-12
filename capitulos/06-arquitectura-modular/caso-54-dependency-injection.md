# Caso 54: Dependency Injection at Scale
## GetIt vs Riverpod vs Injectable

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | dependency injection, modularidad, escalabilidad, arquitectura |
| **Patrón Técnico** | Inversion of Control, Service Locator, Provider-based DI |
| **Stack Seleccionado** | Flutter + Riverpod/Injectable/GetIt (comparativo) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo, necesitamos DI que escale a 15 módulos y permita testing y overrides sin dolor."*

Sin DI claro, aumenta el acoplamiento y se dificulta el testeo y la modularidad.

### Evidencia de Industria

- **Apps modulares:** DI consistente acelera onboarding y refactors.
- **Patrones recomendados:** Evitar singletons globales y favorecer inyección explícita.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Técnico** | Acoplamiento, tests frágiles, refactors costosos |
| **UX/Negocio** | Bugs por estados compartidos mal gestionados |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Singletons manuales | **INADECUADO:** Acopla, difícil de testear. |
| **ACEPTABLE** | GetIt como service locator | **MEJORA:** Centraliza, pero menor trazabilidad de dependencias. |
| **ENTERPRISE** | **Riverpod/Injectable:** inyección declarativa, scopes por módulo, overrides en testing, generación de código para wiring | **ÓPTIMO:** Escalable, testeable, modular. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Scoping por feature, overrides en pruebas, auto-dispose donde aplique, generación de wiring con Injectable, debug de dependencias con Riverpod. |
| **Restricciones Duras (NO permite)** | **Service locator opaco:** GetIt oculta dependencias; menos adecuado para apps muy grandes. **Costo de generación:** Injectable requiere build_runner. **Reflexión:** Evitar patrones que dependan de reflexión con tree shaking. |
| **Criterio de Selección** | Riverpod para DI y estado reactivo; Injectable para wiring declarativo; GetIt solo para casos simples o legado. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| DI | Dependency Injection; proveer dependencias externamente. |
| Service Locator | Registro centralizado de dependencias accesibles globalmente. |
| Scope | Ámbito de vida de una dependencia. |
| Override | Reemplazar una dependencia para testing/entornos. |
| Wiring | Definición de cómo se construyen e inyectan las dependencias. |

---

## Referencias

- [Inversion of Control](https://martinfowler.com/bliki/InversionOfControl.html)
- [Riverpod Docs](https://riverpod.dev/)
- [Injectable + GetIt](https://pub.dev/packages/injectable)
