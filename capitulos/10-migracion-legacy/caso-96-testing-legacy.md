# Caso 96: Testing de Regresión Legacy
## Crear Tests para Código sin Cobertura

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | testing legacy, regresión, cobertura, refactor seguro |
| **Patrón Técnico** | Characterization Tests, Golden Tests, Contract Tests |
| **Stack Seleccionado** | Flutter + test/flutter_test + golden tests + contract tests + mocks seguros |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo, necesito agregar tests a código legacy antes de refactor sin romper funcionalidad."*

Sin cobertura, cualquier cambio puede introducir regresiones invisibles.

### Evidencia de Industria

- **Working with Legacy Code (Feathers):** Characterization tests capturan comportamiento actual.
- **Migraciones:** Tests contractuales reducen riesgo al cambiar APIs/servicios.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Técnico** | Regresiones al refactorizar |
| **UX/Negocio** | Bugs en producción por falta de pruebas |
| **Productivo** | Tiempo perdido en hotfixes |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Refactor sin tests | **INADECUADO:** Alto riesgo. |
| **ACEPTABLE** | Tests unitarios mínimos | **MEJORA:** Mejor que nada, pero cobertura baja. |
| **ENTERPRISE** | **Characterization + contract/golden:** capturar comportamiento actual, contratos para APIs, golden para UI crítica, luego refactor | **ÓPTIMO:** Base segura para cambios. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Capturar outputs actuales como baseline. Golden tests para pantallas críticas. Contract tests para servicios. Aumentar cobertura en áreas de refactor. Documentar casos límite. |
| **Restricciones Duras (NO permite)** | **Comportamiento incorrecto:** Characterization captura errores actuales; requiere decisión para corregir. **Costo inicial:** Crear baseline lleva tiempo. **Falsos positivos:** Golden sensibles a cambios menores; requiere disciplina. |
| **Criterio de Selección** | Priorizar áreas de alto riesgo; usar characterization para baseline; contract/golden donde aplica; refactor después de cubrir. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Characterization Test | Test que documenta el comportamiento actual. |
| Golden Test | Comparación de UI/render con un snapshot esperado. |
| Contract Test | Validar acuerdos entre cliente y servicio. |
| Cobertura | Porcentaje de código ejercitado por tests. |
| Refactor | Cambiar estructura interna sin alterar comportamiento. |

---

## Referencias

- [Working Effectively with Legacy Code](https://martinfowler.com/books/feathers.html)
- [Flutter Golden Tests](https://docs.flutter.dev/testing/testing#widget-tests-and-golden-files)
