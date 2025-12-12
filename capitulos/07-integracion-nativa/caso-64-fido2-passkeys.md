# Caso 64: FIDO2/Passkeys en Flutter
## Autenticación Sin Contraseña con WebAuthn

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | fido2, passkeys, weauthn, autenticación sin contraseña |
| **Patrón Técnico** | WebAuthn, Public Key Credentials, Device-bound Keys |
| **Stack Seleccionado** | Flutter + Platform Channels (FIDO2/Passkeys APIs) + secure storage de credenciales |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero autenticación segura sin contraseñas vulnerables a phishing."*

Passkeys reemplazan contraseñas con claves públicas ligadas al dispositivo y biometría.

### Evidencia de Industria

- **FIDO Alliance:** Passkeys reducen phishing y password reuse.
- **Apple/Google/Microsoft:** Soporte nativo en plataformas actuales.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Seguridad** | Phishing y credential stuffing se reducen, pero requieren UX clara |
| **Compatibilidad** | Soporte varía por SO/versión; fallback necesario |
| **Técnico** | Integración nativa y manejo de claves requiere cuidado |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Solo contraseñas/OTP | **INADECUADO:** Riesgo alto de phishing. |
| **ACEPTABLE** | OTP + biometría local_auth | **MEJORA:** Mejor, pero aún depende de secrets débiles. |
| **ENTERPRISE** | **Passkeys/WebAuthn:** registro y autenticación con claves públicas, challenge-response, fallback controlado | **ÓPTIMO:** Sin contraseñas, resistente a phishing, UX moderna. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Generar credenciales FIDO2 ligadas al dispositivo. Autenticación con challenge-response y biometría. Sin almacenar contraseñas. Fallback a OTP solo cuando necesario. |
| **Restricciones Duras (NO permite)** | **Compatibilidad parcial:** Versiones antiguas pueden no soportar. **Multi-dispositivo:** Sin sync de passkeys se requiere registro por dispositivo. **Errores UX:** Necesario guiar al usuario en prompts nativos. |
| **Criterio de Selección** | Usar APIs nativas FIDO2/Passkeys via Channels; backend con WebAuthn; fallback seguro; almacenamiento de credenciales en secure storage. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Passkey | Credencial FIDO2 ligada a dispositivo/biometría. |
| WebAuthn | Estándar de autenticación basada en claves públicas. |
| Challenge | Valor único enviado por el servidor para validar autenticación. |
| Credential ID | Identificador de la credencial almacenada. |
| Relying Party | Servicio que solicita autenticación WebAuthn. |

---

## Referencias

- [FIDO2 / WebAuthn](https://fidoalliance.org/specifications/)
- [Passkeys Platform Docs](https://developer.apple.com/passkeys/)
