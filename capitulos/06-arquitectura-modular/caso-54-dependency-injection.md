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

### Problema detectado (técnico)
- Singletons globales crean acoplamiento y estados compartidos difíciles de depurar.
- Sin scopes ni overrides, el testing de 15 módulos se vuelve frágil y lento.
- Service locator opaco oculta dependencias y dificulta refactors.

### Escenario de Negocio

> *"Como equipo, necesitamos DI que escale a 15 módulos y permita testing y overrides sin dolor."*

### Incidentes reportados
- **Apps modulares:** Refactors fallidos por dependencias ocultas en service locators.
- **Equipos grandes:** Falta de scopes/overrides duplicó el tiempo de pruebas de integración.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Postmortems de DI | Varios | Service locators opacos generan efectos colaterales en producción. |
| Equipos Riverpod | Global | Mejora trazabilidad y testing con scopes/overrides. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; manejo de dependencias/configuración es foco. |

**Resumen global**
- DI declarativa con scopes y overrides reduce acoplamiento y acelera pruebas; locators opacos aumentan deuda y riesgos.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Técnico** | Acoplamiento, estados compartidos impredecibles |
| **QA/Velocidad** | Tests lentos sin overrides; refactors costosos |
| **Operacional** | Difícil auditar dependencias en incidentes |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Singletons manuales | **INADECUADO:** Acopla, difícil de testear. |
| **ACEPTABLE** | GetIt como service locator | **MEJORA:** Centraliza, pero menor trazabilidad y scopes limitados. |
| **ENTERPRISE** | **Riverpod/Injectable:** inyección declarativa, scopes por módulo, overrides en testing, generación de código para wiring | **ÓPTIMO:** Escalable, testeable, modular. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Scoping por feature, overrides en pruebas, auto-dispose donde aplique, generación de wiring con Injectable, debugging de dependencias con Riverpod. |
| **Restricciones Duras (NO permite)** | **Service locator opaco:** GetIt oculta dependencias; menos apto para apps grandes. **Costo de generación:** Injectable requiere build_runner. **Reflexión:** Evitar reflection/dynamic con tree shaking. |
| **Criterio de Selección** | Riverpod para DI y estado reactivo; Injectable para wiring declarativo; GetIt solo para casos simples o legado. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit | Overrides funcionan y scopes liberan recursos | Móvil/CI |
| Integration (CI) | Wiring generado y contratos de dependencias por módulo | QA/Móvil |
| Observabilidad | Logs/diagnóstico de dependencias resueltas | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Performance | Lazy init en dependencias pesadas | Mejora arranque |
| Estados | Auto-dispose para evitar memory leaks | Estabilidad |
| Experimentos | Overrides para features y pruebas A/B | Flexibilidad |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Gobernanza | Catálogo de providers/servicios por módulo | Trazabilidad |
| Build | Automatizar generación y lint de wiring | Consistencia |
| Migraciones | Path de salida de GetIt a Riverpod/Injectable | Reducción de deuda |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | DI opaca frena escalabilidad y testing de 15 módulos. |
| Opciones evaluadas | Singletons; GetIt; Riverpod/Injectable con scopes y generación. |
| Decisión | Riverpod para DI reactivo + Injectable para wiring declarativo; GetIt solo para legado. |
| Consecuencias | Requiere build_runner y disciplina en scopes; aprendizaje inicial. |
| Riesgos aceptados | Overhead de generación; curva de adopción. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Tiempo de pruebas | ↓ con overrides y scopes | Alerta si no mejora | Velocidad de entrega |
| Bugs por estado compartido | Tendencia a la baja | Crítico si se mantiene | Estabilidad |
| Tiempo de onboarding | ↓ con catálogo de dependencias | Alerta si no baja | Productividad |
| Cambios cruzados | Menos regresiones en refactors | Alerta si siguen | Modularidad |

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
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
