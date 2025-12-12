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

## 4. Manos a la Obra: Estrategia de Implementación

### Justificación del Plan

La estrategia se deriva del análisis de EventBot y las técnicas de malware bancario documentadas:

1. **Teclados de terceros son riesgo** → Custom widget que no usa IME del sistema
2. **Screen recording infiere PINs** → Scramble de posiciones + FLAG_SECURE
3. **Overlay attacks engañan usuarios** → Detección activa de overlays
4. **PINs en tránsito interceptables** → Hash con salt antes de transmitir

La arquitectura usa un Widget custom stateful con Provider para estado mínimo, y Platform Channels para funcionalidades nativas de seguridad.

---

### Fase 1: Diseño — Características del SecurePinPad

**Características de Seguridad:**
1. Grid 4x3 con dígitos en posiciones aleatorias (scrambled)
2. FLAG_SECURE activo durante entrada
3. Detección de overlays antes de mostrar
4. Timeout de 30 segundos de inactividad
5. Máximo 3 intentos antes de bloqueo temporal
6. Solo dots (●) para visualizar PIN ingresado
7. Feedback háptico sin feedback visual del dígito tocado

**Estructura de Carpetas:**
```
lib/features/secure_input/
├── data/
│   └── datasources/
│       └── secure_flag_datasource.dart
├── domain/
│   └── usecases/
│       └── hash_pin_usecase.dart
└── presentation/
    ├── providers/
    │   └── pin_entry_provider.dart
    └── widgets/
        ├── secure_pin_pad.dart
        ├── scrambled_digit_button.dart
        └── pin_dots_indicator.dart
```

---

### Fase 2: Implementación por Plataforma

#### 2.1 Flutter (Cross-Platform) — Capa de Aplicación

**PinEntryProvider:**
- Estado: lista de dígitos ingresados (máximo 6)
- Métodos: `addDigit`, `removeLastDigit`, `clear`, `getHashedPin`
- NUNCA exponer PIN en texto plano

**DigitScrambler:**
- Permutación aleatoria de [0-9] usando `Random.secure()`
- Regenerar posiciones en cada `build()` del widget
- Mantener misma permutación durante sesión de entrada

**SecurePinPad Widget:**
- `StatefulWidget` con estado local
- `GridView.count(crossAxisCount: 3)` para layout 4x3
- Solo dots (●) para PIN ingresado, nunca mostrar dígitos
- Feedback háptico via `HapticFeedback.mediumImpact()`
- Timer de 30 segundos que limpia entrada

**HashPinUseCase:**
- Usar PBKDF2 o Argon2 con salt único por sesión
- Salt generado server-side y enviado al iniciar flujo
- NUNCA transmitir PIN en texto plano

---

#### 2.2 Android — Configuración Nativa

**SecureFlagHandler (Platform Channel):**
```
MethodChannel: 'com.bank.app/secure_flag'
- enableSecureFlag(): window.setFlags(FLAG_SECURE, FLAG_SECURE)
- disableSecureFlag(): window.clearFlags(FLAG_SECURE)
```
- Activar en `initState` del SecurePinPad
- Desactivar en `dispose`

**OverlayDetector Android:**
- Verificar `Settings.canDrawOverlays()` de otras apps
- Listar windows visibles con `WindowManager`
- Si overlay sospechoso detectado → no mostrar PIN pad
- Alertar al usuario sobre riesgo

**AccessibilityServiceDetector:**
- Verificar servicios de accesibilidad activos
- Warning si servicios no confiables están habilitados
- Opcionalmente, no mostrar PIN pad

**Consideraciones Android:**
- FLAG_SECURE previene screenshots y screen recording
- FLAG_SECURE también oculta preview en recent apps
- Para API < 21, usar técnicas alternativas

---

#### 2.3 iOS — Configuración Nativa

**SecureScreenHandler (Platform Channel):**
- iOS no tiene equivalente directo de FLAG_SECURE
- Usar `UITextField.isSecureTextEntry` como referencia
- Implementar blur del contenido cuando app va a background

**ScreenCaptureDetector iOS:**
- Detectar `UIScreen.isCaptured` (iOS 11+)
- Suscribirse a `UIScreen.capturedDidChangeNotification`
- Si screen recording activo → ocultar PIN pad o mostrar pantalla de bloqueo

**JailbreakCheck antes de PIN:**
- Verificar estado de jailbreak antes de mostrar PIN pad
- En dispositivos comprometidos, advertir al usuario

**Consideraciones iOS:**
- `isSecureTextEntry` no aplica a custom widgets
- Screen recording detection disponible desde iOS 11
- Overlay attacks son menos comunes en iOS por restricciones de sandbox

---

### Fase 3: Observability — Métricas y Alertas

**Métricas:**
- `pin.entry.started` / `pin.entry.completed` / `pin.entry.timeout`
- `pin.entry.attempts_failed`
- `pin.security.overlay_detected`
- `pin.security.screen_recording_detected`
- `pin.security.accessibility_warning`

**Alertas:**

| Evento | Severidad | Acción |
|:-------|:----------|:-------|
| `overlay_detected` durante PIN | P2 High | Log + bloquear entrada |
| `screen_recording` durante PIN | P2 High | Log + mostrar warning |
| `3_failed_attempts` | P3 Medium | Bloqueo temporal 5 min |

---

## 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

### TACs Flutter (Cross-Platform)

```
[ ] TAC-5.1-FLUTTER: PIN DEBE ingresarse con teclado custom, NUNCA teclado del sistema.

[ ] TAC-5.2-FLUTTER: Posiciones de dígitos DEBEN aleatorizarse en cada aparición del pad.

[ ] TAC-5.3-FLUTTER: PIN NUNCA debe mostrarse en pantalla, solo dots (●).

[ ] TAC-5.4-FLUTTER: Timeout de 30 segundos de inactividad DEBE limpiar entrada.

[ ] TAC-5.5-FLUTTER: Bloqueo tras 3 intentos fallidos por 5 minutos.

[ ] TAC-5.6-FLUTTER: PIN NUNCA en logs, analytics, ni crash reports.

[ ] TAC-5.7-FLUTTER: Feedback háptico sin feedback visual del dígito presionado.

[ ] TAC-5.8-FLUTTER: PIN transmitido hasheado con salt, NUNCA texto plano.
```

### TACs Android

```
[ ] TAC-5.9-ANDROID: FLAG_SECURE DEBE estar activo durante entrada de PIN.

[ ] TAC-5.10-ANDROID: DEBE detectar overlays y NO mostrar PIN pad si existen.

[ ] TAC-5.11-ANDROID: DEBE advertir si servicios de accesibilidad sospechosos
    están activos.

[ ] TAC-5.12-ANDROID: Preview en recent apps DEBE estar oculto (FLAG_SECURE).
```

### TACs iOS

```
[ ] TAC-5.13-IOS: DEBE detectar screen recording activo (UIScreen.isCaptured).

[ ] TAC-5.14-IOS: Si screen recording detectado, DEBE ocultar o bloquear PIN pad.

[ ] TAC-5.15-IOS: DEBE aplicar blur cuando app va a background durante entrada de PIN.

[ ] TAC-5.16-IOS: DEBE verificar jailbreak antes de mostrar PIN pad en
    operaciones críticas.
```

### TACs Backend (Referencia)

```
[ ] TAC-5.17-BACKEND: DEBE generar salt único por sesión de PIN.

[ ] TAC-5.18-BACKEND: DEBE validar hash de PIN, NUNCA recibir PIN en texto plano.

[ ] TAC-5.19-BACKEND: Rate limiting de intentos de PIN por usuario/dispositivo.
```

---

## 6. Estrategia de Pruebas (Shift-Left)

### Stack de Testing
- **Unit:** `flutter_test` para lógica de scramble y hashing
- **Widget:** `flutter_test` para SecurePinPad comportamiento
- **Integration:** `integration_test` en dispositivo físico
- **Security:** Pruebas manuales con screen recording, overlay apps

### Escenarios Críticos Obligatorios

| # | Escenario | Qué Validar | Plataforma | Tipo |
|:-:|:----------|:------------|:-----------|:-----|
| 1 | **Screenshot durante PIN** | Debe mostrar pantalla negra/vacía | Android | Security (Manual) |
| 2 | **Screen recording activo** | Debe detectar y bloquear/advertir | iOS | Security (Manual) |
| 3 | **App con overlay activo** | PIN pad NO debe mostrarse | Android | Integration |
| 4 | **Timeout de inactividad** | Tras 30 seg sin input, PIN se limpia | Flutter | Widget Test |
| 5 | **3 intentos fallidos** | Bloqueo de 5 minutos, mensaje claro | Flutter | Integration |

---

## Referencias

- [OWASP Mobile Security Testing Guide - Input Validation](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Android FLAG_SECURE Documentation](https://developer.android.com/reference/android/view/WindowManager.LayoutParams#FLAG_SECURE)
- [iOS Screen Recording Detection](https://developer.apple.com/documentation/uikit/uiscreen/2921651-iscaptured)
- [PCI-DSS Requirements for PIN Security](https://www.pcisecuritystandards.org/)

---

*Anterior: [Root Detection](caso-04-root-jailbreak-detection.md) | Siguiente: [Session Hijacking](caso-06-session-hijacking-wifi.md)*
