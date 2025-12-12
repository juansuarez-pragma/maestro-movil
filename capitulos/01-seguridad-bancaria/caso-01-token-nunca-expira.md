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

| Rol | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:----|:-----------------------|:----------------------------------|
| **Junior** | `SharedPreferences` para guardar el token, sin expiración, refresh manual cuando falla una request | **FALLA:** Tokens en texto plano accesibles via backup de Android, sin rotación automática, vulnerable a replay attacks. No detecta dispositivos rooteados. Un atacante con acceso físico al dispositivo extrae el token en 30 segundos con `adb backup`. |
| **Senior** | `flutter_secure_storage` + token con TTL de 15 min + refresh token con TTL de 7 días + interceptor Dio para auto-refresh | **MEJORA:** Cifrado AES-256 en Keychain/Keystore, automatización del refresh, pero carece de: rotación de refresh tokens, detección de anomalías, logout remoto. Un token robado tiene ventana de 7 días. |
| **Architect** | `flutter_secure_storage` + **Token Rotation** (nuevo refresh token en cada uso) + **BLoC para estado de auth** + Device Binding + Anomaly Detection + Remote Revocation via FCM | **ENTERPRISE:** Cada refresh genera nuevo refresh token e invalida el anterior. Vinculación dispositivo-token. Detección de uso simultáneo en múltiples dispositivos. Capacidad de logout remoto instantáneo. Cumple PSD2/SCA. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Almacenamiento cifrado con hardware security module (HSM) en dispositivos compatibles. Rotación automática de refresh tokens sin fricción de usuario. Invalidación remota de sesiones específicas o todas las sesiones. Binding de token a device fingerprint (ANDROID_ID + modelo + instalación única). Detección de jailbreak/root antes de permitir operaciones sensibles. |
| **Restricciones Duras (NO permite)** | **iOS Keychain:** Accesible después de primer unlock del dispositivo (no "always available"). **Android Keystore:** Requiere API 23+ para cifrado respaldado por hardware. **Offline:** No puede validar revocación de tokens sin conexión (requiere estrategia de TTL corto). **Backup:** flutter_secure_storage marca datos como excluded from backup pero el usuario puede tener herramientas de extracción. |
| **Criterio de Selección** | Se usa **flutter_secure_storage** sobre Hive/SharedPreferences porque: 1) Utiliza Keychain (iOS) y EncryptedSharedPreferences/Keystore (Android) respaldados por hardware cuando disponible. 2) Los tokens son datos de alta sensibilidad que requieren cifrado at-rest por regulación. 3) Hive cifrado requiere gestionar la key de cifrado (chicken-egg problem). Se usa **BLoC** sobre Riverpod/Provider para AuthState porque: auditabilidad de transiciones (login→authenticated→refreshing→error), logging de eventos para forensics, y separación clara de lógica de negocio. |

---

## 4. Manos a la Obra: Estrategia de Implementación

### Fase 1: Diseño — Contratos y Arquitectura

**Estructura de Carpetas Recomendada:**

```
lib/
├── core/
│   ├── auth/
│   │   ├── data/
│   │   │   ├── datasources/
│   │   │   │   ├── auth_local_datasource.dart      # Secure Storage
│   │   │   │   └── auth_remote_datasource.dart     # API calls
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

### Fase 2: Implementación — Detalles Técnicos

**AuthLocalDataSource:**
- Configurar `FlutterSecureStorage` con opciones de seguridad máxima
- Android: `encryptedSharedPreferences: true`, `preferencesKeyPrefix` único
- iOS: `KeychainAccessibility.first_unlock_this_device` para balance seguridad/usabilidad
- Implementar métodos: `saveTokenPair`, `getTokenPair`, `clearTokens`, `getDeviceFingerprint`
- Usar `Future.wait` para operaciones paralelas de lectura/escritura

**AuthBloc - Estados y Eventos:**
- Estados: `AuthInitial`, `AuthLoading`, `AuthAuthenticated`, `AuthUnauthenticated`, `AuthFailure`
- Eventos: `AuthCheckRequested`, `AuthLoginRequested`, `AuthTokenRefreshRequested`, `AuthLogoutRequested`, `AuthRemoteLogoutReceived`
- El estado `AuthAuthenticated` debe incluir `session` y `authenticatedAt` para tracking

**TokenRefreshInterceptor (QueuedInterceptor):**
- Extender `QueuedInterceptor` para manejar requests concurrentes durante refresh
- Mantener `Completer` para evitar múltiples refreshes simultáneos
- Refresh proactivo cuando access token tiene < 30 segundos de vida
- En `onError` con 401: intentar refresh y retry de request original
- Excluir rutas de auth del interceptor (`/auth/login`, `/auth/refresh`)

**Device Fingerprint:**
- Combinar: ANDROID_ID (Android) / identifierForVendor (iOS)
- Modelo de dispositivo
- ID de instalación único (generado en primer launch)
- Hash SHA-256 del conjunto para privacidad

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

```
[ ] TAC-1.1: Los tokens DEBEN almacenarse exclusivamente usando flutter_secure_storage
    con configuración de seguridad máxima para cada plataforma.

[ ] TAC-1.2: El access token DEBE tener TTL máximo de 15 minutos.
    El refresh token DEBE tener TTL máximo de 7 días.

[ ] TAC-1.3: DEBE implementarse Token Rotation: cada refresh exitoso genera nuevo
    refresh token e invalida el anterior.

[ ] TAC-1.4: El sistema DEBE detectar y rechazar tokens de familia ya revocada
    (token reuse attack), forzando logout completo.

[ ] TAC-1.5: El interceptor de Dio DEBE realizar refresh proactivo cuando
    access token tenga < 30 segundos de vida.

[ ] TAC-1.6: DEBE existir mecanismo de logout remoto via FCM/Push que invalide
    sesiones en < 5 segundos.

[ ] TAC-1.7: Cada evento de autenticación DEBE generar log de auditoría con:
    timestamp, device_fingerprint, ip_address, resultado, session_family_id.

[ ] TAC-1.8: El tiempo de respuesta del endpoint /auth/refresh NO DEBE superar
    500ms en p95.

[ ] TAC-1.9: DEBE implementarse device binding: tokens vinculados a fingerprint
    de dispositivo.

[ ] TAC-1.10: La aplicación DEBE limpiar TODOS los tokens locales cuando detecta
    dispositivo rooteado/jailbroken.
```

---

## 6. Estrategia de Pruebas (Shift-Left)

### Stack de Testing
- **Unit:** `flutter_test`, `bloc_test`, `mocktail`
- **Integration:** `integration_test`, `mockito`
- **E2E:** `patrol` para flujos completos con UI

### Escenarios Críticos Obligatorios

| # | Escenario | Qué Validar | Tipo |
|:-:|:----------|:------------|:-----|
| 1 | **Refresh token expirado en background** | Al volver a foreground: detectar expiración, limpiar storage, redirigir a login SIN crash. No intentar refresh infinito. | Integration |
| 2 | **Token reuse attack** | Cuando backend responde `token_family_revoked`: limpiar tokens, enviar evento seguridad, mostrar mensaje claro, NO permitir retry. | Unit + Integration |
| 3 | **Múltiples requests con token expirado** | Solo UNA request de refresh debe ejecutarse. Las demás esperan. Si falla, todas fallan consistentemente. | Unit (QueuedInterceptor) |

---

## Referencias

- [OWASP Mobile Security Testing Guide - Session Management](https://owasp.org/www-project-mobile-security-testing-guide/)
- [RFC 6749 - OAuth 2.0](https://tools.ietf.org/html/rfc6749)
- [Auth0 - Refresh Token Rotation](https://auth0.com/docs/secure/tokens/refresh-tokens/refresh-token-rotation)
- [Verizon DBIR 2023](https://www.verizon.com/business/resources/reports/dbir/)

---

*Siguiente caso: [Biometría Falsificada](caso-02-biometria-falsificada.md)*
