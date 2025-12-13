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

### Problema detectado (técnico)
- Refactor sin tests introduce regresiones invisibles.
- Cobertura baja en módulos críticos impide detectar roturas de API/UI.
- Golden/contract ausentes dejan cambios de UI/servicios sin control.

### Escenario de Negocio

> *"Como equipo, necesito agregar tests a código legacy antes de refactor sin romper funcionalidad."*

### Incidentes reportados
- **Legacy sin tests:** Refactors rompieron flujos de pago/login.
- **UI crítica sin golden:** Cambios visuales pasaron a producción.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Working with Legacy Code | Global | Characterization tests reducen riesgo antes de refactor. |
| Postmortems móviles | Varios | Falta de contract/golden causó regresiones en releases. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; cobertura/test es brecha común. |

**Resumen global**
- Characterization + contract/golden en áreas críticas crea baseline segura para refactor; sin tests, el riesgo de regresiones es alto.

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
| **Restricciones Duras (NO permite)** | **Comportamiento incorrecto:** Characterization captura errores actuales; decidir qué corregir. **Costo inicial:** Crear baseline lleva tiempo. **Falsos positivos:** Golden sensibles a cambios menores; requiere disciplina. |
| **Criterio de Selección** | Priorizar áreas de alto riesgo; usar characterization para baseline; contract/golden donde aplica; refactor después de cubrir. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Characterization | Baseline de comportamientos actuales | QA/Móvil |
| Contract | Compatibilidad de servicios/APIs | QA/Backend |
| Golden | UI crítica estable | Móvil/QA |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Selección de pruebas | Priorizar flujos críticos (login/pago) | Impacto alto |
| Mantenimiento | Revisar golden al cambiar UI intencionalmente | Disciplina |
| Documentación | Anotar casos límite y hallazgos | Transferencia de conocimiento |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Cobertura | Objetivo por módulo; empezar por lo crítico | Enfoque |
| Observabilidad | Reportes de cobertura y fallos en CI | Visibilidad |
| Refactor | Solo después de baseline de tests | Seguridad |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Refactor sin pruebas en código legacy de alto riesgo. |
| Opciones evaluadas | Refactor directo; tests mínimos; characterization + contract/golden antes de refactor. |
| Decisión | Characterization + contract/golden en áreas críticas antes de refactor. |
| Consecuencias | Inversión inicial en pruebas; mantenimiento de golden/contract. |
| Riesgos aceptados | Falsos positivos en golden; captura de errores existentes. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Cobertura en módulos críticos | ↑ hasta objetivo acordado | Warning si estancada | Confianza en refactor |
| Regresiones post-refactor | ↓ vs baseline | Crítico si sube | Estabilidad |
| Hotfixes por refactor | Tendencia a la baja | Crítico si sube | Productividad |
| Tiempo de QA | Reducido tras baseline | Alerta si crece | Eficiencia |

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
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
