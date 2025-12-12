# Caso 5: Keyloggers en el Teclado Virtual
## Protegiendo PINs en Apps de Banca Móvil

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | PIN seguro, teclado virtual, keylogger, entrada segura, banca móvil, scramble pad |
| **Patrón Técnico** | Secure Input, Custom Keyboard, Screen Recording Protection |
| **Stack Seleccionado** | Custom Widget (SecurePinPad) + Platform Channels (FLAG_SECURE) + Provider (PinEntryProvider) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero ingresar mi PIN de 6 dígitos para autorizar transferencias, con la certeza de que ningún malware puede capturarlo."*

El problema: Teclados de terceros pueden enviar datos a servidores externos. Malware con permisos de accesibilidad puede leer inputs. Screen recording captura posición de toques.

### Evidencia de Industria

**Caso ai.type Keyboard (2017):** Teclado de terceros que filtró datos de 31 millones de usuarios a servidores externos.

**EventBot Malware (2020):** Troyano Android que abusaba servicios de accesibilidad para capturar credenciales de 200+ apps bancarias en Europa y América.

**Estadística Kaspersky (2023):** 29% del malware bancario móvil usa técnicas de keylogging o screen capture como vector principal.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | PIN comprometido = cuenta comprometida, transferencias fraudulentas |
| **Compliance** | PCI-DSS requiere protección de datos de autenticación |
| **Reputacional** | Crisis de PR si malware captura PINs de clientes |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | `TextField` con `obscureText: true` usando teclado del sistema | **INADECUADO:** Teclado de terceros captura input. Screen recording ve posiciones de toques. Accessibility services pueden leer el campo. |
| **ACEPTABLE** | Custom `PinPad` Widget con botones propios, sin teclado del sistema | **CUMPLE MÍNIMOS:** Evita teclados de terceros. PERO: posición fija de dígitos permite inferir PIN por análisis de video/recording. |
| **ENTERPRISE** | **SecurePinPad:** Scramble aleatorio, FLAG_SECURE activo, detección de overlays, timeout, hash antes de transmitir | **ÓPTIMO PARA BANCA:** Resistente a keyloggers, screen capture, overlay attacks. Cumple PCI-DSS. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Evitar teclados de terceros completamente. Aleatorizar posición de dígitos en cada aparición. Prevenir screenshots y screen recording (FLAG_SECURE). Detectar overlays maliciosos. Timeout automático. Transmitir PIN hasheado. |
| **Restricciones Duras (NO permite)** | **Accesibilidad maliciosa:** Con permiso `BIND_ACCESSIBILITY_SERVICE` malware puede observar texto. **Root/Jailbreak:** Atacante puede hookear cualquier función. **FLAG_SECURE bypass:** En algunas ROMs custom no funciona. **iOS limitación:** No existe equivalente directo de FLAG_SECURE. |
| **Criterio de Selección** | Se usa **Provider simple** porque lógica es mínima (acumular dígitos, validar longitud). Custom Widget es necesario porque ningún package existente combina todas las características de seguridad requeridas. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Secure PIN Pad | Teclado seguro en-app para capturar PIN evitando keyloggers y overlays. |
| Randomized Keypad | Distribución aleatoria de dígitos para mitigar shoulder surfing y patrones. |
| Accessibility Shielding | Controles para minimizar filtrado de texto por servicios de accesibilidad maliciosos. |
| Screen Overlay Protection | Bloqueo/detección de apps que dibujan sobre la pantalla (tapjacking). |
| PCI-DSS Req. 3/8 | Requisitos de protección de datos de autenticación y controles de acceso para PIN/credenciales. |
| Anti-Screenshot | Restricción de capturas de pantalla/recorder durante ingreso de PIN. |

---

## Referencias

- [OWASP Mobile Security Testing Guide - Input Validation](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Android FLAG_SECURE Documentation](https://developer.android.com/reference/android/view/WindowManager.LayoutParams#FLAG_SECURE)
- [iOS Screen Recording Detection](https://developer.apple.com/documentation/uikit/uiscreen/2921651-iscaptured)
- [PCI-DSS Requirements for PIN Security](https://www.pcisecuritystandards.org/)

---

*Anterior: [Root Detection](caso-04-root-jailbreak-detection.md) | Siguiente: [Session Hijacking](caso-06-session-hijacking-wifi.md)*
