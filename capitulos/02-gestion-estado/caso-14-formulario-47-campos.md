# Caso 14: El Formulario de 47 Campos
## Validación Reactiva en Onboarding Bancario

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | onboarding bancario, validación reactiva, formularios complejos, errores en tiempo real |
| **Patrón Técnico** | [Form State Management](#term-form-state-management "Gestión centralizada del estado y validaciones de un formulario."), Reactive Validation, Debounced Async Validation |
| **Stack Seleccionado** | Flutter + Riverpod/Formz + [Debounce](#term-debounce "Retrasar ejecución hasta que cese la entrada, evitando llamadas excesivas.") + InputMask + Async validators |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Validación tardía (solo al submit) provoca que el usuario vea errores al final y abandone.
- Falta de validación cross-field y async (listas de sanciones, RFC/SSN) genera rechazos de KYC y reprocesos manuales.
- Sin máscaras ni UX accesible, aumenta la tasa de error y el tiempo de captura.

### Escenario de Negocio

> *"Como usuario, quiero completar un onboarding con 47 campos sin errores sorpresivos al final."*

### Incidentes reportados
- **Baymard 2023:** Formularios largos con validación tardía elevan abandono > 25%.
- **Fintech EU 2022:** KYC fallidos por validaciones inconsistentes provocaron reprocesos manuales costosos.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Baymard 2023 | Ecommerce/onboarding | > 25% abandono por errores tardíos/fricción. |
| Fintech EU 2022 | Onboarding KYC | Reprocesos manuales por validación inconsistente; impacto en SLA. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; formularios/validaciones son fuente frecuente de bugs UX. |

**Resumen global**
- Errores tardíos/fricción elevan abandono y reprocesos.
- Validación inconsistente afecta cumplimiento KYC/AML y genera costos operativos.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Pérdida de conversiones en onboarding, costos de soporte |
| **Regulatorio** | Datos incompletos/inválidos afectan cumplimiento KYC/AML |
| **UX** | Frustración por errores tardíos o contradictorios |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | `TextEditingController` por campo con validación al submit | **INADECUADO:** Errores tardíos, difícil de mantener, sin consistencia. |
| **ACEPTABLE** | Validators síncronos en onChanged + submit final | **CUMPLE MÍNIMOS:** Muestra errores antes, pero no cubre reglas cross-field ni async (listas de sanciones, RFC, etc.). |
| **ENTERPRISE** | **Form state centralizado + validación reactiva:** Riverpod/Formz, validación cross-field, debounce para reglas async, máscaras de entrada, reglas declarativas reutilizables | **ÓPTIMO:** Consistencia, UX fluida, fácil testeo y trazabilidad. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Validación en tiempo real por campo y cross-field (ej. dirección y código postal). Validadores async con debounce (RFC/SSN en listas de sanciones). Máscaras para formatos (IBAN, CURP). Estados derivados para habilitar botones paso a paso. Persistencia temporal para reanudar formularios. |
| **Restricciones Duras (NO permite)** | **Validación offline de listas externas:** Depende de red para sanciones/KYC. **Performance:** 47 campos con validación pesada pueden jankear si no se debouncean/aislan. **Accesibilidad:** Requiere manejo ARIA/semántica correcto para lectores de pantalla. |
| **Criterio de Selección** | Riverpod/Formz por validación declarativa y estado centralizado; debounce para evitar spam de requests; input masks para consistencia de formato. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Validadores síncronos/async funcionan y manejan estados de error | Equipo móvil, CI |
| Integration (CI) | Cross-field y debounce evitan spam y bloquean datos inválidos | Móvil/QA, CI + staging |
| Seguridad/consistencia | No se filtran datos sensibles en logs; máscaras no exponen datos crudos | Seguridad/QE |
| Observabilidad | Eventos `form.*` con errores típicos y tiempos de llenado | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Errores | Mostrar inline y en tiempo real; resumen al final del paso | Reduce abandono |
| Debounce | 300–500 ms para validación async | Balance UX/backend |
| Accesibilidad | Labels/aria, foco en primer error, lectura por screen reader | Inclusión y cumplimiento |
| Persistencia | Guardar progreso temporal y restaurar | Menos frustración |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Reglas | Centralizar reglas para reutilizar entre pasos | Consistencia |
| Retención de datos | En memoria o cifrado si se guarda temporalmente | Privacidad |
| Async KYC | Retries con backoff y manejo de timeouts | Estabilidad |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Validación tardía e inconsistente provoca abandono y reprocesos. |
| Opciones evaluadas | Validación al submit; validación onChanged sin cross-field; form state centralizado + validación reactiva/async. |
| Decisión | Form state centralizado + validación reactiva (sync/async cross-field) + debounce + máscaras. |
| Consecuencias | Más esfuerzo inicial en reglas y pruebas; dependencia de servicios KYC. |
| Riesgos aceptados | Fallas de red en validaciones async; necesidad de UX clara para errores. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Abandono por fricción | ↓ vs baseline | Alerta si no mejora | Conversión mayor |
| Errores tardíos | 0 errores críticos al submit | Alerta si aparecen | Menos reprocesos |
| Latencia validación async | p95 < 800 ms | Warning si sube | UX ágil |
| Reprocesos KYC | ↓ reprocesos manuales | Alerta si no baja | Costo operativo menor |
| Accesibilidad | 0 hallazgos críticos en QA accesible | Alerta si hallazgos | Cumplimiento/inclusión |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-validacion-reactiva"></a>Validación reactiva | Mostrar errores conforme se digita, no solo al enviar. |
| <a id="term-cross-field-validation"></a>Cross-field validation | Reglas que involucran múltiples campos (ej. país + documento). |
| <a id="term-debounce"></a>Debounce | Retrasar ejecución hasta que cese la entrada, evitando llamadas excesivas. |
| <a id="term-input-mask"></a>Input Mask | Formato guiado para capturar datos (IBAN, teléfono). |
| <a id="term-form-state-management"></a>Form State Management | Gestión centralizada del estado y validaciones de un formulario. |
| <a id="term-cross-field-validation"></a>Cross-field validation | Reglas que involucran múltiples campos (ej. país + documento). |
| <a id="term-async-validation"></a>Async validation | Validación contra servicios externos (listas de sanciones, KYC). |

---

## Referencias

- [Form Validation UX (NNG)](https://www.nngroup.com/articles/errors-forms-design-guidelines/)
- [Formz Package](https://pub.dev/packages/formz)
- [NIST SP 800-63A - Enrollment & Identity Proofing](https://pages.nist.gov/800-63-3/sp800-63a.html)
- [Baymard Institute - Form Abandonment](https://baymard.com/research/ecommerce-form-usability)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
