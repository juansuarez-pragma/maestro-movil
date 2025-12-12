# Caso 7: El Dilema del Remember Me
## Persistencia Segura de Credenciales en E-commerce

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | remember me, persistencia de sesión, login automático, credenciales guardadas, e-commerce |
| **Patrón Técnico** | Secure Credential Storage, Biometric-Gated Access, Device Trust |
| **Stack Seleccionado** | flutter_secure_storage + local_auth + Hive (encrypted) + Cubit (RememberMeCubit) |
| **Nivel de Criticidad** | Medio |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario de e-commerce, quiero que la app recuerde mi sesión para no tener que ingresar mi contraseña cada vez que abro la app."*

Balance crítico: Fricción cero vs Seguridad. E-commerce tolera más riesgo que banca, pero maneja datos de pago y direcciones.

### Evidencia de Industria

**Amazon 1-Click Patent:** Amazon patentó y popularizó la compra con un click, demostrando que reducir fricción aumenta conversión. Balancea conveniencia con re-auth para cambios de cuenta.

**Estudio Baymard Institute 2023:** 17% de abandonos de carrito ocurren por proceso de checkout largo, incluyendo re-autenticación forzada.

**Caso Uber (2016):** Brecha expuso datos de 57M usuarios. Sesiones persistentes sin protección adecuada facilitaron el acceso.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Compras fraudulentas si dispositivo robado o prestado |
| **Usabilidad** | Demasiada fricción = abandono de carrito, churn |
| **Compliance** | PCI-DSS requiere protección de credenciales almacenadas |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Guardar user/password en SharedPreferences para auto-login | **INADECUADO:** Credenciales en texto plano. NUNCA guardar contraseña. Backup extrae datos. |
| **ACEPTABLE** | Refresh token en flutter_secure_storage, auto-login sin interacción | **CUMPLE MÍNIMOS:** No guarda contraseña, usa token. PERO: dispositivo desbloqueado = acceso total sin verificación. |
| **ENTERPRISE** | **Biometric-Gated Access:** Token cifrado en secure storage, biometría para desbloquear sesión, step-up para operaciones sensibles | **ÓPTIMO:** Conveniencia + capa de protección biométrica. Cumple balance UX/Seguridad. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Auto-login rápido protegido por biometría. Sin almacenar contraseña en dispositivo. Step-up configurable por tipo de operación. Revocación remota de sesiones. Múltiples dispositivos gestionados. |
| **Restricciones Duras (NO permite)** | **Sin biometría:** Dispositivos sin biometría deben ofrecer alternativa (PIN de app). **Biometría comprometida:** Si alguien enrolla su huella, protección es nula. **Dispositivo compartido:** No recomendado para tablets familiares. **Biometría opcional:** Usuario puede rechazar, debe existir fallback. |
| **Criterio de Selección** | Se usa **Cubit** porque estado de Remember Me es simple (enabled/disabled/authenticating), sin eventos complejos. **flutter_secure_storage** para refresh token, **Hive encrypted** para preferencias y metadata no crítica. |

---

## 4. Manos a la Obra: Estrategia de Implementación

### Justificación del Plan

La estrategia se deriva del balance demostrado por Amazon y las estadísticas de abandono de Baymard:

1. **Fricción mata conversión** → Remember Me como opt-in con protección
2. **Contraseñas no deben almacenarse** → Solo tokens con TTL
3. **Dispositivo desbloqueado no es suficiente** → Gate biométrico
4. **No todas las operaciones son iguales** → Step-up por riesgo

La arquitectura usa Cubit para estado simple, con flutter_secure_storage para tokens y local_auth para gate biométrico.

---

### Fase 1: Diseño — Flujos de Remember Me

**Flujo de Activación:**
1. Login exitoso con credenciales
2. Prompt: "¿Desea que recordemos su sesión?"
3. Usuario acepta → Refresh token almacenado en secure storage
4. Token protegido con flag de biometría requerida

**Flujo de Auto-Login:**
1. App se abre
2. Detecta Remember Me activo
3. Solicita biometría/PIN de app
4. Si OK → Usa refresh token para obtener access token
5. Si falla biometría 3 veces → Login tradicional

**Operaciones con Step-Up:**
- Cambiar dirección de envío → Re-auth biométrica
- Agregar método de pago → Re-auth + OTP
- Compra > $500 → Re-auth biométrica

**Estructura de Carpetas:**
```
lib/features/remember_me/
├── data/
│   ├── datasources/
│   │   ├── remember_me_local_datasource.dart
│   │   └── biometric_datasource.dart
│   └── repositories/
│       └── remember_me_repository_impl.dart
├── domain/
│   ├── entities/
│   │   └── remember_me_config.dart
│   └── usecases/
│       ├── enable_remember_me.dart
│       ├── authenticate_biometric.dart
│       └── disable_remember_me.dart
└── presentation/
    ├── cubit/
    │   ├── remember_me_cubit.dart
    │   └── remember_me_state.dart
    └── widgets/
        └── remember_me_prompt.dart
```

---

### Fase 2: Implementación por Plataforma

#### 2.1 Flutter (Cross-Platform) — Capa de Aplicación

**RememberMeCubit:**
- Estados: `initial`, `enabled`, `disabled`, `authenticating`, `authenticated`, `failed`
- Métodos: `enable()`, `disable()`, `authenticateWithBiometric()`, `checkStatus()`
- Persistir estado en Hive encrypted

**SecureTokenStorage:**
- Usar flutter_secure_storage para refresh token
- iOS: Configurar `authenticationRequired: true` en Keychain options
- Android: Requerir BiometricPrompt antes de lectura

**BiometricGate:**
- Wrapper sobre `local_auth` package
- Flujo: biometría → PIN de app → contraseña (fallback chain)
- Tracking de intentos fallidos
- Bloqueo temporal tras 3 fallos

**StepUpAuthenticationService:**
- Determina cuándo requerir re-auth según operación
- Configurable desde remote config
- UI components para prompts de re-auth

---

#### 2.2 Android — Configuración Nativa

**BiometricPrompt Integration:**
- `local_auth` usa BiometricPrompt internamente
- Configurar `setAllowedAuthenticators()` para Class 2/3
- `setNegativeButtonText()` para opción de PIN

**SecureStorage Android:**
```
AndroidOptions(
  encryptedSharedPreferences: true,
  keyCipherAlgorithm: KeyCipherAlgorithm.RSA_ECB_PKCS1Padding,
  storageCipherAlgorithm: StorageCipherAlgorithm.AES_GCM_NoPadding,
)
```

**Consideraciones Android:**
- Biometría Class 3 (Strong) para operaciones financieras
- Fallback a PIN/Pattern si biometría no disponible
- Manejar cambios en enrolled biometrics (invalidar tokens)

---

#### 2.3 iOS — Configuración Nativa

**LocalAuthentication Framework:**
- `local_auth` usa LAContext internamente
- Configurar `localizedReason` claro para el usuario
- Soportar Face ID y Touch ID

**Keychain con Biometría:**
```
IOSOptions(
  accessibility: KeychainAccessibility.whenPasscodeSetThisDeviceOnly,
  useAccessControl: true,
  accessControl: AccessControl.biometryCurrentSet,
)
```

**Consideraciones iOS:**
- `biometryCurrentSet` invalida token si biometría cambia
- Requiere passcode configurado en dispositivo
- `whenPasscodeSetThisDeviceOnly` no se sincroniza a iCloud
- Info.plist: `NSFaceIDUsageDescription` requerido

---

### Fase 3: Observability — Métricas y Alertas

**Métricas:**
- `remember_me.enabled` / `remember_me.disabled`
- `remember_me.biometric.success` / `remember_me.biometric.failure`
- `remember_me.fallback_to_password`
- `remember_me.stepup.triggered`
- `remember_me.session.invalidated`

**KPIs:**

| Métrica | Objetivo | Alerta |
|:--------|:---------|:-------|
| Remember Me adoption | > 60% | < 40% → Mejorar UX prompt |
| Biometric success rate | > 95% | < 85% → Revisar configuración |
| Fallback to password | < 5% | > 15% → Investigar |

---

## 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

### TACs Flutter (Cross-Platform)

```
[ ] TAC-7.1-FLUTTER: Remember Me DEBE ser opt-in, NUNCA habilitado por default.

[ ] TAC-7.2-FLUTTER: NUNCA almacenar contraseña, solo refresh tokens.

[ ] TAC-7.3-FLUTTER: Acceso a token DEBE requerir biometría o PIN de app.

[ ] TAC-7.4-FLUTTER: DEBE existir opción clara de desactivar y limpiar datos.

[ ] TAC-7.5-FLUTTER: Tras 3 fallos de biometría → fallback a login tradicional.

[ ] TAC-7.6-FLUTTER: Operaciones sensibles (> $500, cambio de pago) DEBEN
    requerir step-up authentication.

[ ] TAC-7.7-FLUTTER: Refresh token DEBE tener TTL máximo de 30 días.
```

### TACs Android

```
[ ] TAC-7.8-ANDROID: DEBE usar BiometricPrompt con Class 2+ para autenticación.

[ ] TAC-7.9-ANDROID: Cambio en enrolled biometrics DEBE invalidar tokens.

[ ] TAC-7.10-ANDROID: SecureStorage DEBE usar EncryptedSharedPreferences.

[ ] TAC-7.11-ANDROID: DEBE ofrecer fallback a PIN/Pattern si biometría
    no disponible.
```

### TACs iOS

```
[ ] TAC-7.12-IOS: DEBE usar LAContext con Face ID/Touch ID.

[ ] TAC-7.13-IOS: Keychain DEBE configurarse con `biometryCurrentSet` para
    invalidar si biometría cambia.

[ ] TAC-7.14-IOS: Info.plist DEBE incluir NSFaceIDUsageDescription con texto claro.

[ ] TAC-7.15-IOS: Token NO DEBE sincronizarse a iCloud
    (whenPasscodeSetThisDeviceOnly).
```

### TACs Backend (Referencia)

```
[ ] TAC-7.16-BACKEND: "Cerrar sesión en todos dispositivos" DEBE invalidar
    todos los refresh tokens del usuario.

[ ] TAC-7.17-BACKEND: DEBE soportar múltiples refresh tokens (multi-device).

[ ] TAC-7.18-BACKEND: Warning de seguridad si Remember Me en dispositivo
    compartido detectado.
```

---

## 6. Estrategia de Pruebas (Shift-Left)

### Stack de Testing
- **Unit:** `flutter_test`, `bloc_test` para RememberMeCubit
- **Integration:** `integration_test` en dispositivo físico (biometría)
- **E2E:** `patrol` para flujos completos

### Escenarios Críticos Obligatorios

| # | Escenario | Qué Validar | Plataforma | Tipo |
|:-:|:----------|:------------|:-----------|:-----|
| 1 | **Activar → cerrar → reabrir** | Pide biometría, no password | Flutter | E2E |
| 2 | **Token expirado** | Biometría OK pero refresh falla → login tradicional | Flutter | Integration |
| 3 | **Desactivar Remember Me** | Tokens limpiados, siguiente apertura pide login | Flutter | Integration |
| 4 | **Cambio de huella/Face ID** | Tokens invalidados, requiere login | Android + iOS | Security |
| 5 | **3 fallos de biometría** | Fallback a login tradicional | Flutter | Integration |

---

## Referencias

- [OWASP Mobile Security - Authentication](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Android BiometricPrompt](https://developer.android.com/reference/android/hardware/biometrics/BiometricPrompt)
- [iOS LocalAuthentication](https://developer.apple.com/documentation/localauthentication)
- [Baymard Institute - Cart Abandonment Statistics](https://baymard.com/lists/cart-abandonment-rate)

---

*Anterior: [Session Hijacking](caso-06-session-hijacking-wifi.md) | Siguiente: [MFA SIM Swapping](caso-08-mfa-sim-swapping.md)*
