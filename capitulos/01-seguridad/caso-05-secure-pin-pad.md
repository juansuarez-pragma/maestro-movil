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

### Problema detectado (técnico)
- Teclados de terceros pueden exfiltrar PIN; `TextField` con teclado del sistema no controla data exfiltration.
- Malware con accesibilidad lee eventos de entrada; sin randomización del keypad, el video/screen recording permite inferir PIN.
- Sin [FLAG_SECURE](#term-flag-secure "Flag de Android que bloquea screenshots/recordings.") ni detección de overlays, el PIN se filtra por screen capture o tapjacking.

### Escenario de Negocio

> *"Como usuario, quiero ingresar mi PIN de 6 dígitos para autorizar transferencias, con la certeza de que ningún malware puede capturarlo."*

- Caso: teclado de terceros (ai.type) envía pulsaciones a servidor externo; PIN filtrado → transferencia fraudulenta.
- Caso: malware de accesibilidad (EventBot) captura eventos y tokens; sin mitigaciones, el cliente pierde fondos.

### Incidentes reportados

- **ai.type Keyboard (2017):** Filtró datos de 31M de usuarios a servidores externos.
- **EventBot (2020):** Troyano Android que abusó accesibilidad para capturar credenciales/PIN de 200+ apps bancarias en Europa/América.
- **Kaspersky (2023):** 29% del malware bancario móvil usa keylogging o screen capture como vector principal.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| NowSecure State of Mobile App Security (2024) | 1,000+ apps móviles (US/EU) | 85% fallan ≥1 control MASVS; entrada segura/anti-captura es un fallo recurrente. |
| Kaspersky Mobile Threats (2023) | Global (énfasis LATAM/APAC) | 29% del malware bancario móvil usa keylogging o screen capture. |
| CASB/MDM enterprise reports (2022) | Flotas Android corporativas | ~10-15% de dispositivos con teclados de terceros activos; riesgo de exfiltración de entradas. |

**Resumen global**
- Keylogging/screen capture es uno de los vectores principales de malware bancario (≈30% de muestras).
- Alto porcentaje de apps falla en controles de entrada segura (MASVS), dejando expuesto el PIN.
- Teclados de terceros son frecuentes en flotas (~10-15%), elevando riesgo de exfiltración.

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
| **Capacidades (SÍ permite)** | Evitar teclados de terceros. Aleatorizar posición de dígitos en cada aparición ([Randomized Keypad](#term-random-keypad "Distribución aleatoria de dígitos para mitigar shoulder surfing y patrones.")). Prevenir screenshots/recording con [FLAG_SECURE](#term-flag-secure "Flag de Android que bloquea screenshots/recordings."). Detectar overlays maliciosos. Timeout automático. Transmitir PIN hasheado. |
| **Restricciones Duras (NO permite)** | **Accesibilidad maliciosa:** Con permiso `BIND_ACCESSIBILITY_SERVICE` malware puede observar texto. **Root/Jailbreak:** Atacante puede hookear cualquier función. **FLAG_SECURE bypass:** En algunas ROMs custom no funciona. **iOS:** No hay equivalente 1:1 de FLAG_SECURE. |
| **Criterio de Selección** | Se usa **[Provider](#term-provider "Patrón de estado simple usado para la lógica mínima del PIN.")** porque la lógica es mínima (acumular dígitos, validar longitud) y requiere simplicidad. Custom Widget es necesario porque ningún package existente combina todas las características de seguridad. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | PinPad genera keypad aleatorio y aplica timeout; hash se calcula antes de enviar | Equipo móvil, CI |
| Integration (CI) | FLAG_SECURE activo en pantallas de PIN; overlays disparan bloqueo y mensaje | Móvil/QA, CI + dispositivos reales |
| Seguridad manual | Pruebas con accesibilidad maliciosa y screen recorder → PIN no filtrado; tapjacking bloqueado | Seguridad/QE, dispositivos rooteados |
| Observabilidad | Evento `pin.entry` con métricas (reintentos, overlay_detected) sin capturar PIN | Móvil/SRE, CI |

### 3.2 UX, accesibilidad y privacidad
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Mensajes/overlays | Mensaje claro si se detecta overlay/screen capture; CTA para cerrar apps superpuestas | Reduce riesgo de tapjacking y mejora comprensión |
| Reintentos | Limitar reintentos y mostrar contador; feedback háptico opcional | Control de uso indebido y mejor UX |
| Manejo de PIN | No persistir PIN; hash en memoria temporal y limpiar tras uso | Minimiza superficie de fuga |

### 3.3 Operación y seguridad
| Tema | Política | Umbral/Alerta |
|:-----|:--------|:--------------|
| FLAG_SECURE | Obligatorio en todas las vistas de PIN | Alerta si vista sin FLAG_SECURE |
| Aleatorización | Re-aleatorizar keypad en cada apertura y tras cada intento fallido | Evita patrones y replay visual |
| Monitoreo | `overlay_detected` y `accessibility_detected` alertan si > 1% sesiones | SOC investiga anomalías |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Teclados/overlays/recordings pueden exfiltrar PIN; input estándar no ofrece protección. |
| Opciones evaluadas | Teclado sistema con obscure; custom keypad fijo; SecurePinPad con scramble, FLAG_SECURE, overlay detection. |
| Decisión | SecurePinPad con scramble aleatorio, FLAG_SECURE, detección de overlay, timeout y hash local. |
| Consecuencias | Mayor esfuerzo UI y pruebas en múltiples OEMs; dependencia de policies de accesibilidad. |
| Riesgos aceptados | ROMs que deshabilitan FLAG_SECURE; accesibilidad maliciosa con permisos elevados. |

---

## 4. Impacto esperado

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Captura de PIN por malware/keylogging | Reducción ≥ 80% | Alertar si no mejora tras rollout | Fraude contenido |
| Reintentos promedio | ≤ 1.2 | Alerta si > 1.2 | Mejor UX y menos soporte |
| Tiempo de ingreso p95 | < 8 s | Warning si se aproxima | Conversión y experiencia |
| Eventos overlay/accessibility | < 1% de sesiones | Alerta si ≥ 1% | Detección temprana de tapjacking/abuso |
| Tickets de soporte (teclado) | ↓ 20% | Alertar si no baja | Costos de soporte controlados |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-secure-pin-pad"></a>Secure PIN Pad | Teclado seguro en-app para capturar PIN evitando keyloggers y overlays. |
| <a id="term-random-keypad"></a>Randomized Keypad | Distribución aleatoria de dígitos para mitigar shoulder surfing y patrones. |
| <a id="term-accessibility"></a>Accessibility Shielding | Controles para minimizar filtrado de texto por servicios de accesibilidad maliciosos. |
| <a id="term-overlay"></a>Screen Overlay Protection | Bloqueo/detección de apps que dibujan sobre la pantalla (tapjacking). |
| <a id="term-pci"></a>PCI-DSS Req. 3/8 | Requisitos de protección de datos de autenticación y controles de acceso para PIN/credenciales. |
| <a id="term-anti-screenshot"></a>Anti-Screenshot / FLAG_SECURE | Restricción de capturas de pantalla/recording durante ingreso de PIN. |
| <a id="term-flag-secure"></a>FLAG_SECURE | Flag de Android que bloquea screenshots/recordings. |
| <a id="term-provider"></a>Provider | Patrón de estado simple usado para la lógica mínima del PIN. |

---

## Referencias

- [OWASP Mobile Security Testing Guide - Input Validation](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Android FLAG_SECURE Documentation](https://developer.android.com/reference/android/view/WindowManager.LayoutParams#FLAG_SECURE)
- [iOS Screen Recording Detection](https://developer.apple.com/documentation/uikit/uiscreen/2921651-iscaptured)
- [PCI-DSS Requirements for PIN Security](https://www.pcisecuritystandards.org/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
- [Kaspersky - Mobile Threats 2023](https://securelist.com/mobile-malware-evolution-2023/111742/)

---

*Anterior: [Root Detection](caso-04-root-jailbreak-detection.md) | Siguiente: [Session Hijacking](caso-06-session-hijacking-wifi.md)*
