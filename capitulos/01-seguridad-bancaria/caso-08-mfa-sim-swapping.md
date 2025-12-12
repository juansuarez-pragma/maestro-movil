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

## 4. Manos a la Obra: Estrategia de Implementación

### Justificación del Plan

La estrategia se deriva de las recomendaciones NIST y los casos de SIM swapping documentados:

1. **SMS es el eslabón débil** → No como método primario
2. **Push MFA combina seguridad y UX** → Método preferido
3. **Offline es necesario** → TOTP como backup
4. **Recovery es crítico** → Códigos backup y proceso verificado

La arquitectura usa BLoC para gestionar el flujo complejo de MFA, con firebase_messaging para Push y package `otp` para TOTP.

---

### Fase 1: Diseño — Jerarquía de Métodos MFA

**Prioridad de Métodos:**
1. **Push MFA** (mejor UX + seguridad): Notificación push con detalles de operación, aprobación con biometría
2. **TOTP** (fallback offline): Códigos de 6 dígitos cada 30 segundos, compatible con Google Authenticator
3. **Email OTP** (fallback sin dispositivo): Para recovery o dispositivo nuevo
4. **SMS** (último recurso): Solo con verificación adicional (pregunta secreta, callback)

**Flujo Push MFA:**
1. Usuario inicia operación sensible
2. Backend genera challenge y envía push
3. App muestra detalles: "Transferencia de $500 a Juan Pérez"
4. Usuario aprueba con biometría
5. App firma respuesta con device key
6. Backend verifica firma y aprueba operación

**Estructura de Carpetas:**
```
lib/features/mfa/
├── data/
│   ├── datasources/
│   │   ├── push_mfa_datasource.dart
│   │   ├── totp_datasource.dart
│   │   └── sms_otp_datasource.dart
│   └── repositories/
│       └── mfa_repository_impl.dart
├── domain/
│   ├── entities/
│   │   ├── mfa_challenge.dart
│   │   └── mfa_method.dart
│   └── usecases/
│       ├── request_mfa.dart
│       ├── verify_totp.dart
│       └── approve_push_mfa.dart
└── presentation/
    ├── bloc/
    │   ├── mfa_bloc.dart
    │   ├── mfa_event.dart
    │   └── mfa_state.dart
    └── widgets/
        ├── push_approval_sheet.dart
        └── totp_input_widget.dart
```

---

### Fase 2: Implementación por Plataforma

#### 2.1 Flutter (Cross-Platform) — Capa de Aplicación

**MFABloc Estados:**
- `MFAInitial`: Sin challenge activo
- `MFAPushSent`: Push enviado, esperando interacción
- `MFAPushWaiting`: Usuario vio push, decidiendo
- `MFAPushApproved`: Usuario aprobó con biometría
- `MFATOTPRequested`: Esperando código TOTP
- `MFATOTPVerifying`: Verificando código
- `MFASuccess`: MFA completado exitosamente
- `MFAFailure`: MFA falló (timeout, código incorrecto, denegado)

**PushMFAService:**
- Registrar dispositivo con FCM token en backend
- Almacenar `device_mfa_key` (keypair) en flutter_secure_storage
- Verificar firma de push recibido (evitar push spoofing)
- Firmar respuesta de aprobación con private key
- Mostrar detalles de operación antes de aprobar

**TOTPService:**
- Usar package `otp` para generación RFC 6238
- Seed almacenado en flutter_secure_storage
- Aceptar código ± 1 ventana (30 segundos de tolerancia)
- Rate limiting: máximo 5 intentos por minuto

**BackupCodesService:**
- Generar 10 códigos de backup durante setup
- Cada código es single-use
- Almacenados hasheados en backend
- Mostrar una vez, usuario debe guardar

---

#### 2.2 Android — Integración Nativa

**Firebase Cloud Messaging:**
- Configurar `firebase_messaging` para recibir push
- Data message (no notification) para control total
- High priority para entrega inmediata
- Manejar push en foreground y background

**Push Security Android:**
- Verificar que push viene de servidor legítimo (firma)
- No mostrar detalles sensibles en notification tray
- Usar `EncryptedSharedPreferences` para device_mfa_key

**Consideraciones Android:**
- FCM requiere Google Play Services
- Para dispositivos Huawei: integrar HMS Push como alternativa
- Battery optimization puede afectar entrega de push

---

#### 2.3 iOS — Integración Nativa

**Apple Push Notification Service:**
- Configurar APNs certificates en Apple Developer
- Usar `firebase_messaging` que abstrae APNs
- Configurar `content-available: 1` para silent push si necesario

**Push Security iOS:**
- UNNotificationServiceExtension para procesar antes de mostrar
- Keychain para almacenar device_mfa_key
- Rich notifications con acciones inline

**Consideraciones iOS:**
- Requiere provisioning profile con Push capability
- Background modes: "Remote notifications" habilitado
- Push no garantizado en Low Power Mode extremo

---

### Fase 3: Observability — Métricas y Alertas

**Métricas:**
- `mfa.method.used` (distribución: push, totp, sms)
- `mfa.push.sent` / `mfa.push.received` / `mfa.push.approved` / `mfa.push.denied`
- `mfa.totp.attempts` / `mfa.totp.success` / `mfa.totp.failure`
- `mfa.sms.fallback` (cuántos caen a SMS)
- `mfa.completion_time_ms`

**Alertas:**

| Evento | Severidad | Acción |
|:-------|:----------|:-------|
| `mfa.push.denied` múltiples | P2 High | Posible ataque, alertar usuario |
| `mfa.totp.failure` > 3 consecutivos | P3 Medium | Rate limit, posible brute force |
| `mfa.sms.fallback` alto | P3 Medium | Investigar por qué no usan push |

---

## 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

### TACs Flutter (Cross-Platform)

```
[ ] TAC-8.1-FLUTTER: SMS NO DEBE ser método primario para operaciones financieras.

[ ] TAC-8.2-FLUTTER: DEBE ofrecerse Push MFA con aprobación biométrica como
    método preferido.

[ ] TAC-8.3-FLUTTER: DEBE ofrecerse TOTP como alternativa offline.

[ ] TAC-8.4-FLUTTER: Push MFA DEBE mostrar detalles de operación antes de aprobar.

[ ] TAC-8.5-FLUTTER: DEBEN generarse códigos backup durante setup de MFA.

[ ] TAC-8.6-FLUTTER: Seed TOTP DEBE almacenarse cifrado en secure storage.

[ ] TAC-8.7-FLUTTER: Rate limiting: máximo 5 intentos TOTP por minuto.

[ ] TAC-8.8-FLUTTER: Cambio de método MFA DEBE requerir método actual + contraseña.
```

### TACs Android

```
[ ] TAC-8.9-ANDROID: DEBE usar FCM para Push MFA con data messages.

[ ] TAC-8.10-ANDROID: Device MFA key DEBE almacenarse en EncryptedSharedPreferences.

[ ] TAC-8.11-ANDROID: Para dispositivos sin Play Services, DEBE ofrecer
    alternativa (HMS Push o TOTP-only).

[ ] TAC-8.12-ANDROID: Push NO DEBE mostrar detalles sensibles en notification tray.
```

### TACs iOS

```
[ ] TAC-8.13-IOS: DEBE configurar APNs capability en provisioning profile.

[ ] TAC-8.14-IOS: Device MFA key DEBE almacenarse en Keychain.

[ ] TAC-8.15-IOS: DEBE soportar rich notifications con acciones de aprobar/denegar.

[ ] TAC-8.16-IOS: Background modes DEBE incluir "Remote notifications".
```

### TACs Backend (Referencia)

```
[ ] TAC-8.17-BACKEND: SMS como fallback DEBE requerir verificación adicional
    (pregunta secreta, callback).

[ ] TAC-8.18-BACKEND: DEBE existir recovery si pierde todos los factores
    (proceso verificado con ID).

[ ] TAC-8.19-BACKEND: Push challenges DEBEN tener timeout de 2 minutos.

[ ] TAC-8.20-BACKEND: DEBE almacenar backup codes hasheados, no en texto plano.
```

---

## 6. Estrategia de Pruebas (Shift-Left)

### Stack de Testing
- **Unit:** `flutter_test`, `bloc_test` para MFABloc, tests de TOTP generation
- **Integration:** `integration_test` con mock server
- **E2E:** `patrol` con push notifications simuladas

### Escenarios Críticos Obligatorios

| # | Escenario | Qué Validar | Plataforma | Tipo |
|:-:|:----------|:------------|:-----------|:-----|
| 1 | **Push desde otro dispositivo** | Solo dispositivo registrado puede aprobar | Flutter | Security |
| 2 | **TOTP expirado** | Código > 60 seg rechazado, 30 seg aceptado | Flutter | Unit |
| 3 | **5 fallos TOTP** | Rate limiting activo, bloqueo temporal | Flutter | Integration |
| 4 | **Push no llega** | Fallback a TOTP disponible | Flutter | Integration |
| 5 | **Backup code usado** | Funciona una vez, segundo uso falla | Flutter | Integration |

---

## Referencias

- [NIST SP 800-63B - Authentication Guidelines](https://pages.nist.gov/800-63-3/sp800-63b.html)
- [RFC 6238 - TOTP Algorithm](https://tools.ietf.org/html/rfc6238)
- [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging)
- [FIDO Alliance - Passwordless Authentication](https://fidoalliance.org/)

---

*Anterior: [Remember Me](caso-07-remember-me-seguro.md) | Siguiente: [PCI-DSS Tokenización](caso-09-pci-dss-tokenizacion.md)*
