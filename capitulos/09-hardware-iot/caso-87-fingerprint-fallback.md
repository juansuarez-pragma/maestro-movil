# Caso 87: Fingerprint Fallback
## Qué Hacer Cuando el Sensor Falla

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | fingerprint, fallback, biometría, accesibilidad |
| **Patrón Técnico** | Biometric Fallback, Risk-Based Step-up, Graceful Degradation |
| **Stack Seleccionado** | Flutter + local_auth + policies de fallback (PIN/OTP) + Riverpod estado |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, si la huella falla, necesito otra opción segura para no quedar bloqueado."*

Sensores defectuosos o dedos no reconocidos pueden bloquear usuarios y generar soporte.

### Evidencia de Industria

- **Apps bancarias:** Ofrecen fallback a PIN/OTP con límites y controles.
- **Accesibilidad:** Algunos usuarios no pueden usar huella/rostro.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Usuarios bloqueados, churn |
| **Seguridad** | Fallback débil abre vectores de ataque |
| **Operacional** | Aumento de tickets de soporte |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Solo biometría, sin alternativa | **INADECUADO:** Bloqueos y mala UX. |
| **ACEPTABLE** | Fallback a PIN/OTP sin políticas | **MEJORA:** Evita bloqueo, pero puede ser más débil. |
| **ENTERPRISE** | **Fallback gobernado:** límites de intentos, step-up según riesgo, biometría opcional pero recomendada, auditoría de fallos | **ÓPTIMO:** Balance seguridad/UX con control. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Detectar fallos de biometría y ofrecer PIN/OTP. Limitar intentos y aplicar step-up. Registrar fallos para detección de fraude. Permitir desactivar biometría en dispositivos problemáticos. |
| **Restricciones Duras (NO permite)** | **Biometría obligatoria por política:** Si regulatorio exige, fallback puede ser limitado. **Riesgo de PIN débil:** Requiere políticas de longitud/complexidad. **Accesibilidad:** Debe cumplir A11y en prompts. |
| **Criterio de Selección** | Fallback seguro (PIN/OTP) con límites y logging; biometría opcional pero recomendada; mensajes claros al usuario. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Fallback | Alternativa cuando el método principal falla. |
| Step-up | Aumentar seguridad según riesgo. |
| Intentos limitados | Número máximo antes de bloqueo. |
| A11y | Accesibilidad; cumplimiento para lectores de pantalla. |
| PIN | Código personal; debe tener políticas de seguridad. |

---

## Referencias

- [local_auth](https://pub.dev/packages/local_auth)
- [OWASP Mobile Auth](https://owasp.org/www-project-mobile-security-testing-guide/)
