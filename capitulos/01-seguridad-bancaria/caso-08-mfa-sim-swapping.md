# Caso 8: OTP Interceptado
## Implementando MFA Resistente a SIM Swapping

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | OTP, SMS, SIM swapping, MFA, autenticación multifactor, TOTP, push notification |
| **Patrón Técnico** | TOTP (RFC 6238), Push-based MFA, Hardware Tokens |
| **Stack Seleccionado** | otp (TOTP) + firebase_messaging (Push MFA) + BLoC (MFABloc) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero recibir un código de verificación para aprobar transacciones importantes, pero me preocupa la seguridad de los SMS."*

Problema fundamental: SMS como segundo factor es vulnerable a SIM swapping, SS7 attacks, y social engineering en carriers.

### Evidencia de Industria

**Caso Twitter/Jack Dorsey (2019):** El CEO de Twitter fue víctima de SIM swap a pesar de tener MFA con SMS. Atacantes convencieron a carrier de transferir número.

**Estadística FBI IC3 (2021):** Ataques de SIM swapping aumentaron 400% entre 2018-2021, con pérdidas reportadas de $68 millones en 2021.

**NIST SP 800-63B (2017):** El National Institute of Standards and Technology desaconseja SMS como único segundo factor desde 2016 por vulnerabilidades conocidas.

**Caso Coinbase (2021):** Usuarios perdieron fondos por SIM swapping a pesar de tener 2FA con SMS habilitado.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Account takeover completo, transferencias fraudulentas |
| **Regulatorio** | NIST y reguladores desaconsejan SMS, PSD2 requiere SCA |
| **Técnico** | SMS tiene latencia variable, puede no llegar, depende de carrier |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | SMS OTP como único segundo factor | **INADECUADO:** Vulnerable a SIM swap, SS7 attacks, interception. Costo por SMS. Latencia. Dependencia de carrier. |
| **ACEPTABLE** | TOTP (Google Authenticator) como alternativa a SMS | **CUMPLE MÍNIMOS:** No depende de carrier, funciona offline, estándar RFC 6238. PERO: seed debe protegerse, requiere otra app, pérdida de dispositivo = pérdida de acceso. |
| **ENTERPRISE** | **MFA Híbrido:** Push MFA primario con aprobación in-app, TOTP como backup offline, SMS solo como último recurso con verificación adicional | **ÓPTIMO PARA BANCA:** Push es más seguro y mejor UX. TOTP para offline. SMS excepcional con fricción extra. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Push MFA con aprobación biométrica en la misma app. TOTP estándar RFC 6238 compatible con authenticators. Múltiples dispositivos MFA registrados. Códigos backup para recovery. Transacciones firmadas criptográficamente. |
| **Restricciones Duras (NO permite)** | **Sin internet:** Push no funciona, TOTP sí. **Dispositivo perdido:** Necesita recovery robusto. **Push delivery:** No garantizado inmediato (APNs/FCM pueden demorar). **Usuarios no técnicos:** TOTP puede confundir. |
| **Criterio de Selección** | Se usa **BLoC** porque flujo MFA tiene múltiples estados y transiciones complejas (requesting→sent→waiting→verifying→success/failure). Eventos claros permiten auditoría del flujo. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| SIM Swapping | Robo o duplicado de SIM para recibir OTP/SMS y tomar control de cuentas. |
| MFA | Autenticación multifactor; combina algo que sabes, tienes o eres. |
| OTP SMS | One-Time Password enviado por SMS; vulnerable a SIM swap y malware de SMS. |
| TOTP | OTP basado en tiempo (RFC 6238), menos dependiente de la red móvil. |
| Push MFA | Aprobación de login mediante notificación push con contexto de riesgo. |
| WebAuthn/Passkeys | Autenticación sin contraseña usando llaves FIDO2 y biometría del dispositivo. |

---

## Referencias

- [NIST SP 800-63B - Authentication Guidelines](https://pages.nist.gov/800-63-3/sp800-63b.html)
- [RFC 6238 - TOTP Algorithm](https://tools.ietf.org/html/rfc6238)
- [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging)
- [FIDO Alliance - Passwordless Authentication](https://fidoalliance.org/)

---

*Anterior: [Remember Me](caso-07-remember-me-seguro.md) | Siguiente: [PCI-DSS Tokenización](caso-09-pci-dss-tokenizacion.md)*
