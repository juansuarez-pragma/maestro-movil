# Caso 14: El Formulario de 47 Campos
## Validación Reactiva en Onboarding Bancario

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | onboarding bancario, validación reactiva, formularios complejos, errores en tiempo real |
| **Patrón Técnico** | Form State Management, Reactive Validation, Debounced Async Validation |
| **Stack Seleccionado** | Flutter + Riverpod/Formz + Debounce + InputMask + Async validators |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero completar un onboarding con 47 campos sin errores sorpresivos al final."*

Formularios extensos generan abandono si los errores se muestran tarde o de forma inconsistente entre secciones y pasos.

### Evidencia de Industria

- **Baymard 2023:** Formularios largos con validación tardía elevan abandono > 25%.
- **Fintech EU 2022:** KYC fallidos por validaciones inconsistentes provocaron reprocesos manuales costosos.

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

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Validación reactiva | Mostrar errores conforme se digita, no solo al enviar. |
| Cross-field validation | Reglas que involucran múltiples campos (ej. país + documento). |
| Debounce | Retrasar ejecución hasta que cese la entrada, evitando llamadas excesivas. |
| Input Mask | Formato guiado para capturar datos (IBAN, teléfono). |
| Form State Management | Gestión centralizada del estado y validaciones de un formulario. |

---

## Referencias

- [Form Validation UX (NNG)](https://www.nngroup.com/articles/errors-forms-design-guidelines/)
- [Formz Package](https://pub.dev/packages/formz)
- [NIST SP 800-63A - Enrollment & Identity Proofing](https://pages.nist.gov/800-63-3/sp800-63a.html)
