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

#### Tabla de Componentes y Responsabilidades

| Capa | Componente | Responsabilidad |
|:-----|:-----------|:----------------|
| **Data** | AuthLocalDataSource | Persistencia segura de tokens en almacenamiento cifrado del dispositivo |
| **Data** | AuthRemoteDataSource | Comunicación con endpoints de autenticación del backend |
| **Data** | TokenPairModel | Modelo de datos para serialización/deserialización de tokens |
| **Data** | DeviceInfoModel | Modelo para información de fingerprint del dispositivo |
| **Data** | AuthRepositoryImpl | Implementación concreta que coordina datasources |
| **Domain** | AuthSession | Entidad de dominio que representa una sesión autenticada |
| **Domain** | AuthRepository | Contrato abstracto del repositorio de autenticación |
| **Domain** | LoginUseCase | Caso de uso para flujo de login |
| **Domain** | RefreshTokenUseCase | Caso de uso para rotación de tokens |
| **Domain** | LogoutUseCase | Caso de uso para cierre de sesión |
| **Presentation** | AuthBloc | Gestor de estado de autenticación con auditoría de transiciones |
| **Network** | AuthInterceptor | Inyección automática de access token en requests |
| **Network** | TokenRefreshInterceptor | Manejo de expiración y refresh automático |

#### Contrato de Datos: Token Response (Backend → App)

| Atributo | Tipo | Descripción | Reglas de Validación |
|:---------|:-----|:------------|:---------------------|
| `access_token` | String (JWT) | Token de acceso para autorizar requests | TTL máximo: 15 minutos. Formato: JWT firmado. |
| `refresh_token` | String (Opaco) | Token para obtener nuevos access tokens | TTL máximo: 7 días. Single-use (rotación). |
| `token_family_id` | UUID | Identificador de familia para detectar reuso | Generado en login inicial. Compartido por toda la cadena de refreshes. |
| `device_bound` | Boolean | Indica si el token está vinculado al dispositivo | Si `true`, backend valida device fingerprint. |
| `expires_in` | Integer | Segundos hasta expiración del access token | Usado para refresh proactivo. |

#### Diagrama Sugerido: Flujo de Token Rotation

```
[DIAGRAMA DE SECUENCIA]

Título: Token Rotation con Detección de Reuso

Participantes:
- App (Flutter)
- TokenRefreshInterceptor
- Backend /auth/refresh
- Token Store (DB)

Flujo Principal:
1. App → Interceptor: Request con access_token expirado
2. Interceptor: Detecta 401 o TTL < 30s
3. Interceptor → Backend: POST /auth/refresh {refresh_token, device_fingerprint}
4. Backend → Token Store: Verificar refresh_token válido y no usado
5. Token Store → Backend: Token válido, familia activa
6. Backend → Token Store: Invalidar refresh_token anterior
7. Backend → Token Store: Generar nuevo refresh_token en misma familia
8. Backend → Interceptor: {new_access_token, new_refresh_token}
9. Interceptor → App: Reintentar request original con nuevo token

Flujo Alternativo (Token Reuse Attack):
4b. Token Store → Backend: refresh_token YA FUE USADO
5b. Backend → Token Store: INVALIDAR TODA LA FAMILIA
6b. Backend → Interceptor: 403 {error: "token_family_revoked"}
7b. Interceptor → App: Forzar logout, limpiar storage
```

---

### Fase 2: Implementación por Plataforma

#### 2.1 Flutter (Cross-Platform) — Capa de Aplicación

##### Tabla de Definición: AuthLocalDataSource

| Método | Entrada | Salida | Lógica de Negocio |
|:-------|:--------|:-------|:------------------|
| `saveTokenPair` | TokenPair | void | 1. Validar que tokens no sean nulos. 2. Guardar en secure storage con key prefixada. 3. Guardar timestamp de almacenamiento. |
| `getTokenPair` | - | TokenPair? | 1. Leer de secure storage. 2. Si no existe, retornar null. 3. Si existe, deserializar y retornar. |
| `clearTokens` | - | void | 1. Eliminar access_token. 2. Eliminar refresh_token. 3. Eliminar metadata asociada. |
| `getDeviceFingerprint` | - | String | 1. Obtener identificadores de plataforma. 2. Concatenar valores. 3. Aplicar hash SHA-256. |

##### Tabla de Estados: AuthBloc

| Estado | Descripción | Datos Asociados | Transiciones Válidas |
|:-------|:------------|:----------------|:---------------------|
| `AuthInitial` | Estado inicial, sin verificación | Ninguno | → AuthLoading |
| `AuthLoading` | Verificando credenciales o refrescando | Ninguno | → AuthAuthenticated, AuthUnauthenticated, AuthFailure |
| `AuthAuthenticated` | Sesión activa válida | session, authenticatedAt | → AuthLoading (refresh), AuthUnauthenticated (logout) |
| `AuthUnauthenticated` | Sin sesión activa | reason (opcional) | → AuthLoading (login) |
| `AuthFailure` | Error en operación de auth | errorMessage, errorCode | → AuthLoading (retry), AuthUnauthenticated |

##### Tabla de Eventos: AuthBloc

| Evento | Trigger | Handler | Efecto |
|:-------|:--------|:--------|:-------|
| `AuthCheckRequested` | App startup | Verificar tokens en storage | Emitir Authenticated o Unauthenticated |
| `AuthLoginRequested` | Usuario envía credenciales | Llamar backend, guardar tokens | Emitir Authenticated o Failure |
| `AuthTokenRefreshRequested` | Token cerca de expirar | Llamar /refresh con rotation | Actualizar tokens o emitir Unauthenticated |
| `AuthLogoutRequested` | Usuario cierra sesión | Limpiar storage, notificar backend | Emitir Unauthenticated |
| `AuthRemoteLogoutReceived` | Push de invalidación | Limpiar storage inmediatamente | Emitir Unauthenticated con reason |

##### Algoritmo: TokenRefreshInterceptor (QueuedInterceptor)

**Propósito:** Manejar automáticamente la expiración de tokens y evitar múltiples refreshes simultáneos.

**Lógica paso a paso:**

1. **onRequest (antes de cada request):**
   - SI la ruta está en lista de exclusión (`/auth/login`, `/auth/refresh`) → continuar sin modificar
   - SI access_token tiene menos de 30 segundos de vida → ejecutar refresh proactivo
   - Agregar header `Authorization: Bearer {access_token}`

2. **onError (cuando request falla):**
   - SI código de error NO es 401 → propagar error original
   - SI ya hay un refresh en progreso → encolar request actual y esperar
   - SI no hay refresh en progreso:
     - Crear Completer para coordinar requests encoladas
     - Llamar a RefreshTokenUseCase
     - SI refresh exitoso → actualizar tokens, completar Completer, reintentar request
     - SI refresh falla con `token_family_revoked` → emitir AuthRemoteLogoutReceived, propagar error
     - SI refresh falla por otra razón → propagar error, limpiar cola

3. **Manejo de concurrencia:**
   - Mantener referencia a Completer activo
   - Requests concurrentes esperan el mismo Completer
   - Al completar refresh, todos los requests encolados se reintentan

---

#### 2.2 Android — Configuración Nativa

##### Tabla de Configuración: Secure Storage Android

| Parámetro | Valor Requerido | Justificación |
|:----------|:----------------|:--------------|
| `encryptedSharedPreferences` | `true` | Habilita cifrado AES-256-GCM respaldado por Keystore |
| `sharedPreferencesName` | `'secure_auth_prefs'` | Aísla preferencias de auth de otras preferencias |
| `preferencesKeyPrefix` | `'auth_'` | Facilita identificación y limpieza selectiva |
| `keyCipherAlgorithm` | RSA/ECB/PKCS1Padding | Algoritmo para cifrado de clave maestra |
| `storageCipherAlgorithm` | AES/GCM/NoPadding | Algoritmo para cifrado de datos |

##### Tabla de Configuración: AndroidManifest

| Atributo | Valor | Propósito |
|:---------|:------|:----------|
| `android:allowBackup` | `false` | Previene extracción de datos via `adb backup` |
| `android:fullBackupContent` | `false` | Excluye app de Auto Backup |

##### Algoritmo: Device Fingerprint Android

1. Obtener `ANDROID_ID` de `Settings.Secure`
2. Obtener `Build.MODEL` (modelo del dispositivo)
3. Obtener `Build.MANUFACTURER` (fabricante)
4. Concatenar: `{ANDROID_ID}|{MODEL}|{MANUFACTURER}`
5. Aplicar hash SHA-256 al resultado
6. Retornar hash como string hexadecimal

**Consideraciones:**
- API 23+ requerido para KeyStore respaldado por hardware
- En API < 23, usar EncryptedSharedPreferences con cifrado software
- ANDROID_ID es único por combinación app/usuario/dispositivo

---

#### 2.3 iOS — Configuración Nativa

##### Tabla de Configuración: Secure Storage iOS

| Parámetro | Valor Requerido | Justificación |
|:----------|:----------------|:--------------|
| `accessibility` | `first_unlock_this_device` | Accesible después del primer desbloqueo, no sincroniza a iCloud |
| `accountName` | `'com.{empresa}.{app}.auth'` | Identifica la cuenta en Keychain |
| `synchronizable` | `false` (implícito) | No sincroniza a iCloud por la opción de accessibility |

##### Tabla de Valores: Keychain Accessibility

| Valor | Cuándo Accesible | Sincroniza iCloud | Uso Recomendado |
|:------|:-----------------|:------------------|:----------------|
| `passcode` | Solo con passcode activo | No | Máxima seguridad, puede bloquear al usuario |
| `first_unlock` | Después de primer desbloqueo | Sí | Balance seguridad/UX, sincroniza |
| `first_unlock_this_device` | Después de primer desbloqueo | **No** | **RECOMENDADO para tokens bancarios** |
| `always` | Siempre | Sí | Baja seguridad, no usar para auth |
| `always_this_device` | Siempre | No | Baja seguridad, no usar para auth |

##### Algoritmo: Device Fingerprint iOS

1. Obtener `identifierForVendor` de `UIDevice.current`
2. Obtener modelo de dispositivo via `utsname`
3. Concatenar: `{identifierForVendor}|{deviceModel}`
4. Aplicar hash SHA-256 al resultado
5. Retornar hash como string hexadecimal

**Consideraciones:**
- `identifierForVendor` persiste mientras haya al menos una app del vendor instalada
- Keychain persiste entre reinstalaciones (implementar cleanup en primera ejecución si necesario)
- Agregar `NSFaceIDUsageDescription` en Info.plist si se usa biometría

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
