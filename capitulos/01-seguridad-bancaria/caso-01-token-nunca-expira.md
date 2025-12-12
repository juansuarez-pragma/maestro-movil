# Caso 1: El Token que Nunca Expira
## Cómo una Sesión Zombie Costó $2.4M en Fraude

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | refresh token, sesión persistente, fraude bancario, logout automático, robo de credenciales |
| **Patrón Técnico** | Token Rotation, Sliding Session, Secure Token Storage |
| **Stack Seleccionado** | flutter_secure_storage + Dio Interceptors + BLoC (AuthBloc) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario de banca móvil, quiero mantener mi sesión iniciada para no tener que autenticarme cada vez que abro la app."*

Esta historia de usuario, aparentemente inocente, esconde uno de los vectores de ataque más explotados en aplicaciones financieras: la gestión inadecuada del ciclo de vida de tokens.

### Evidencia de Industria

**Caso Revolut 2022:** En septiembre de 2022, Revolut sufrió una brecha que expuso datos de 50,000+ usuarios. El ataque de ingeniería social fue posible parcialmente porque tokens de sesión comprometidos no fueron invalidados oportunamente. La compañía tardó horas en detectar sesiones anómalas porque carecían de mecanismos robustos de rotación y revocación.

**Caso Capital One 2019:** Aunque el vector principal fue una misconfiguration de WAF, la investigación reveló que tokens de servicio tenían lifetimes excesivamente largos, ampliando la ventana de explotación.

**Estadística crítica:** Según el reporte de Verizon DBIR 2023, el 74% de las brechas financieras involucran credenciales comprometidas, y el tiempo promedio de detección de sesiones fraudulentas es de 207 días.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Fraude directo ($2.4M promedio en incidentes de session hijacking según IBM Cost of Data Breach 2023), multas regulatorias (hasta 4% de revenue global bajo GDPR, sanciones de CNBV/Banco Central) |
| **Reputacional** | Pérdida de confianza, churn de clientes (estudios muestran 65% de usuarios abandonan servicios financieros tras brecha de seguridad) |
| **Técnico** | Deuda de seguridad acumulada, necesidad de reescritura completa del módulo de autenticación, auditorías SOC 2 fallidas |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | `SharedPreferences` para guardar el token, sin expiración, refresh manual cuando falla una request | **INADECUADO:** Tokens en texto plano accesibles via backup de Android, sin rotación automática, vulnerable a replay attacks. No detecta dispositivos rooteados. Un atacante con acceso físico al dispositivo extrae el token en 30 segundos con `adb backup`. |
| **ACEPTABLE** | `flutter_secure_storage` + token con TTL de 15 min + refresh token con TTL de 7 días + interceptor Dio para auto-refresh | **CUMPLE MÍNIMOS:** Cifrado AES-256 en Keychain/Keystore, automatización del refresh, pero carece de: rotación de refresh tokens, detección de anomalías, logout remoto. Un token robado tiene ventana de 7 días. |
| **ENTERPRISE** | `flutter_secure_storage` + **Token Rotation** (nuevo refresh token en cada uso) + **BLoC para estado de auth** + Device Binding + Anomaly Detection + Remote Revocation via FCM | **ÓPTIMO PARA BANCA:** Cada refresh genera nuevo refresh token e invalida el anterior. Vinculación dispositivo-token. Detección de uso simultáneo en múltiples dispositivos. Capacidad de logout remoto instantáneo. Cumple PSD2/SCA. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Almacenamiento cifrado con hardware security module (HSM) en dispositivos compatibles. Rotación automática de refresh tokens sin fricción de usuario. Invalidación remota de sesiones específicas o todas las sesiones. Binding de token a device fingerprint (ANDROID_ID + modelo + instalación única). Detección de jailbreak/root antes de permitir operaciones sensibles. |
| **Restricciones Duras (NO permite)** | **iOS Keychain:** Accesible después de primer unlock del dispositivo (no "always available"). **Android Keystore:** Requiere API 23+ para cifrado respaldado por hardware. **Offline:** No puede validar revocación de tokens sin conexión (requiere estrategia de TTL corto). **Backup:** flutter_secure_storage marca datos como excluded from backup pero el usuario puede tener herramientas de extracción. |
| **Criterio de Selección** | Se usa **flutter_secure_storage** sobre Hive/SharedPreferences porque: 1) Utiliza Keychain (iOS) y EncryptedSharedPreferences/Keystore (Android) respaldados por hardware cuando disponible. 2) Los tokens son datos de alta sensibilidad que requieren cifrado at-rest por regulación. 3) Hive cifrado requiere gestionar la key de cifrado (chicken-egg problem). Se usa **BLoC** sobre Riverpod/Provider para AuthState porque: auditabilidad de transiciones (login→authenticated→refreshing→error), logging de eventos para forensics, y separación clara de lógica de negocio. |

---

## 4. Manos a la Obra: Estrategia de Implementación

### Justificación del Plan

La estrategia de implementación se deriva directamente de los requisitos de seguridad identificados en la matriz de soluciones nivel **ENTERPRISE**. El análisis de los casos de Revolut y Capital One revela que las brechas ocurrieron por:

1. **Tokens de larga duración sin rotación** → Implementamos Token Rotation
2. **Falta de binding dispositivo-sesión** → Implementamos Device Fingerprinting
3. **Imposibilidad de invalidar sesiones remotamente** → Implementamos Remote Revocation via FCM
4. **Ausencia de detección de anomalías** → Implementamos logging estructurado para análisis

La arquitectura propuesta sigue Clean Architecture para facilitar testing y mantenibilidad, con BLoC como gestor de estado por su capacidad de auditoría de transiciones.

---

### Fase 1: Diseño — Contratos y Arquitectura

**Estructura de Carpetas Recomendada:**

```
lib/
├── core/
│   ├── auth/
│   │   ├── data/
│   │   │   ├── datasources/
│   │   │   │   ├── auth_local_datasource.dart
│   │   │   │   └── auth_remote_datasource.dart
│   │   │   ├── models/
│   │   │   │   ├── token_pair_model.dart
│   │   │   │   └── device_info_model.dart
│   │   │   └── repositories/
│   │   │       └── auth_repository_impl.dart
│   │   ├── domain/
│   │   │   ├── entities/
│   │   │   │   └── auth_session.dart
│   │   │   ├── repositories/
│   │   │   │   └── auth_repository.dart
│   │   │   └── usecases/
│   │   │       ├── login_usecase.dart
│   │   │       ├── refresh_token_usecase.dart
│   │   │       └── logout_usecase.dart
│   │   └── presentation/
│   │       └── bloc/
│   │           ├── auth_bloc.dart
│   │           ├── auth_event.dart
│   │           └── auth_state.dart
│   └── network/
│       └── interceptors/
│           ├── auth_interceptor.dart
│           └── token_refresh_interceptor.dart
```

**Contrato de Token Response (Backend):**
- `access_token`: JWT con TTL de 15 minutos
- `refresh_token`: Token opaco con TTL de 7 días
- `token_family_id`: Identificador de familia para detectar reuso
- `device_bound`: Boolean indicando si está vinculado a dispositivo

---

### Fase 2: Implementación por Plataforma

#### 2.1 Flutter (Cross-Platform) — Capa de Aplicación

**AuthLocalDataSource:**
- Configurar `FlutterSecureStorage` con opciones de seguridad máxima por plataforma
- Implementar métodos: `saveTokenPair`, `getTokenPair`, `clearTokens`, `getDeviceFingerprint`
- Usar `Future.wait` para operaciones paralelas de lectura/escritura

**AuthBloc - Estados y Eventos:**
- Estados: `AuthInitial`, `AuthLoading`, `AuthAuthenticated`, `AuthUnauthenticated`, `AuthFailure`
- Eventos: `AuthCheckRequested`, `AuthLoginRequested`, `AuthTokenRefreshRequested`, `AuthLogoutRequested`, `AuthRemoteLogoutReceived`
- El estado `AuthAuthenticated` debe incluir `session` y `authenticatedAt` para tracking

**TokenRefreshInterceptor (QueuedInterceptor):**
- Extender `QueuedInterceptor` de Dio para manejar requests concurrentes durante refresh
- Mantener `Completer` para evitar múltiples refreshes simultáneos
- Refresh proactivo cuando access token tiene < 30 segundos de vida
- En `onError` con 401: intentar refresh y retry de request original
- Excluir rutas de auth del interceptor (`/auth/login`, `/auth/refresh`)

---

#### 2.2 Android — Configuración Nativa

**Configuración de flutter_secure_storage:**
```
AndroidOptions(
  encryptedSharedPreferences: true,
  sharedPreferencesName: 'secure_auth_prefs',
  preferencesKeyPrefix: 'auth_',
)
```

**Device Fingerprint Android:**
- Obtener `ANDROID_ID` via `Settings.Secure`
- Combinar con `Build.MODEL`, `Build.MANUFACTURER`
- Hash SHA-256 del conjunto

**Consideraciones Android:**
- API 23+ requerido para KeyStore respaldado por hardware
- Configurar `android:allowBackup="false"` en AndroidManifest
- EncryptedSharedPreferences usa AES-256-GCM

---

#### 2.3 iOS — Configuración Nativa

**Configuración de flutter_secure_storage:**
```
IOSOptions(
  accessibility: KeychainAccessibility.first_unlock_this_device,
  accountName: 'com.bank.app.auth',
)
```

**Device Fingerprint iOS:**
- Obtener `identifierForVendor` via UIDevice
- Combinar con modelo de dispositivo
- Hash SHA-256 del conjunto

**Consideraciones iOS:**
- Keychain persiste entre reinstalaciones (considerar cleanup)
- `first_unlock_this_device` balancea seguridad y usabilidad
- Datos NO se sincronizan a iCloud con esta configuración

---

### Fase 3: Observability — Métricas y Alertas

**Métricas a enviar a Datadog/Firebase:**
- `auth.login.success` / `auth.login.failure`
- `auth.token.refresh` / `auth.token.refresh_failure`
- `auth.session.expired`
- `auth.logout.remote`
- `auth.security.token_reuse` (CRÍTICO)
- `auth.token.refresh_latency_ms`

**KPIs de Éxito:**

| Métrica | Objetivo | Alerta |
|:--------|:---------|:-------|
| Token refresh success rate | > 99.5% | < 98% → PagerDuty |
| Token refresh latency p95 | < 500ms | > 1000ms → Warning |
| Token reuse incidents | 0 / día | > 0 → Critical |
| Session duration promedio | > 5 min | < 1 min → Investigar |

---

## 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

### TACs Flutter (Cross-Platform)

```
[ ] TAC-1.1-FLUTTER: Los tokens DEBEN almacenarse usando flutter_secure_storage,
    NUNCA SharedPreferences ni Hive sin cifrado.

[ ] TAC-1.2-FLUTTER: El access token DEBE tener TTL máximo de 15 minutos.
    El refresh token DEBE tener TTL máximo de 7 días.

[ ] TAC-1.3-FLUTTER: DEBE implementarse Token Rotation: cada refresh exitoso
    genera nuevo refresh token e invalida el anterior en el backend.

[ ] TAC-1.4-FLUTTER: El sistema DEBE detectar token_family_revoked del backend
    y forzar logout completo limpiando storage local.

[ ] TAC-1.5-FLUTTER: El TokenRefreshInterceptor DEBE usar QueuedInterceptor
    para evitar múltiples refreshes simultáneos.

[ ] TAC-1.6-FLUTTER: DEBE implementarse refresh proactivo cuando access token
    tenga < 30 segundos de vida restante.
```

### TACs Android

```
[ ] TAC-1.7-ANDROID: flutter_secure_storage DEBE configurarse con
    encryptedSharedPreferences: true (requiere API 23+).

[ ] TAC-1.8-ANDROID: El AndroidManifest DEBE incluir android:allowBackup="false"
    para prevenir extracción de datos via adb backup.

[ ] TAC-1.9-ANDROID: El Device Fingerprint DEBE incluir ANDROID_ID + Build.MODEL
    hasheados con SHA-256.

[ ] TAC-1.10-ANDROID: La app DEBE verificar si el dispositivo está rooteado
    usando checks de /system/bin/su y build tags antes de operaciones sensibles.
```

### TACs iOS

```
[ ] TAC-1.11-IOS: flutter_secure_storage DEBE configurarse con
    KeychainAccessibility.first_unlock_this_device.

[ ] TAC-1.12-IOS: Los tokens NO DEBEN sincronizarse a iCloud
    (configuración por defecto con first_unlock_this_device).

[ ] TAC-1.13-IOS: El Device Fingerprint DEBE usar identifierForVendor + modelo,
    hasheados con SHA-256.

[ ] TAC-1.14-IOS: DEBE considerarse limpiar Keychain en primera instalación
    para evitar tokens huérfanos de instalaciones previas.
```

### TACs Backend (Referencia)

```
[ ] TAC-1.15-BACKEND: El endpoint /auth/refresh DEBE retornar nuevo refresh_token
    en cada llamada exitosa e invalidar el anterior.

[ ] TAC-1.16-BACKEND: DEBE implementarse detección de token reuse: si un refresh
    token ya usado se presenta, invalidar toda la familia de tokens.

[ ] TAC-1.17-BACKEND: DEBE existir endpoint para logout remoto que invalide
    sesiones específicas o todas las sesiones de un usuario.

[ ] TAC-1.18-BACKEND: El tiempo de respuesta de /auth/refresh NO DEBE superar
    500ms en p95.
```

---

## 6. Estrategia de Pruebas (Shift-Left)

### Stack de Testing
- **Unit:** `flutter_test`, `bloc_test`, `mocktail`
- **Integration:** `integration_test`, `mockito`
- **E2E:** `patrol` para flujos completos con UI

### Escenarios Críticos Obligatorios

| # | Escenario | Qué Validar | Plataforma | Tipo |
|:-:|:----------|:------------|:-----------|:-----|
| 1 | **Refresh token expirado en background** | Al volver a foreground: detectar expiración, limpiar storage, redirigir a login SIN crash. No intentar refresh infinito. | Flutter | Integration |
| 2 | **Token reuse attack** | Cuando backend responde `token_family_revoked`: limpiar tokens, enviar evento seguridad, mostrar mensaje claro, NO permitir retry. | Flutter | Unit + Integration |
| 3 | **Múltiples requests con token expirado** | Solo UNA request de refresh debe ejecutarse. Las demás esperan. Si falla, todas fallan consistentemente. | Flutter | Unit |
| 4 | **Extracción via adb backup** | Con allowBackup=false, verificar que tokens NO aparecen en backup extraído. | Android | Security (Manual) |
| 5 | **Persistencia tras reinstalación** | En iOS, verificar comportamiento de Keychain tras desinstalar/reinstalar. Implementar cleanup si necesario. | iOS | Integration |

---

## Referencias

- [OWASP Mobile Security Testing Guide - Session Management](https://owasp.org/www-project-mobile-security-testing-guide/)
- [RFC 6749 - OAuth 2.0](https://tools.ietf.org/html/rfc6749)
- [Auth0 - Refresh Token Rotation](https://auth0.com/docs/secure/tokens/refresh-tokens/refresh-token-rotation)
- [Android Keystore System](https://developer.android.com/training/articles/keystore)
- [iOS Keychain Services](https://developer.apple.com/documentation/security/keychain_services)

---

*Siguiente caso: [Biometría Falsificada](caso-02-biometria-falsificada.md)*
