# Maestría Empresarial Móvil
## Guía Definitiva de Arquitectura Flutter para Banca y E-commerce

---

# TABLA DE CONTENIDOS (100 CASOS)

---

## Capítulo 1: Seguridad Bancaria y Gestión de Identidad (Casos 1-10)

| Caso | Título |
|:----:|:-------|
| 1 | **El Token que Nunca Expira:** Cómo una Sesión Zombie Costó $2.4M en Fraude |
| 2 | **Biometría Falsificada:** Cuando Face ID No Es Suficiente para Aprobar un Crédito |
| 3 | **El Man-in-the-Middle Silencioso:** Certificate Pinning en Transferencias Interbancarias |
| 4 | **Rooted pero Confiable:** Detectar Dispositivos Comprometidos sin Bloquear Usuarios Legítimos |
| 5 | **Keyloggers en el Teclado Virtual:** Protegiendo PINs en Apps de Banca Móvil |
| 6 | **Session Hijacking en WiFi Público:** El Caso del Café que Vació Cuentas |
| 7 | **El Dilema del Remember Me:** Persistencia Segura de Credenciales en E-commerce |
| 8 | **OTP Interceptado:** Implementando MFA Resistente a SIM Swapping |
| 9 | **PCI-DSS en el Bolsillo:** Tokenización de Tarjetas sin Tocar Datos Sensibles |
| 10 | **El Empleado Deshonesto:** Auditoría y Trazabilidad de Acciones en Apps Internas |

---

## Capítulo 2: Gestión de Estado Compleja y Consistencia de Datos (Casos 11-20)

| Caso | Título |
|:----:|:-------|
| 11 | **El Carrito Fantasma:** Sincronizar Estado entre 5 Pestañas y 3 Dispositivos |
| 12 | **Race Condition en el Checkout:** Cuando Dos Hilos Debitan la Misma Cuenta |
| 13 | **Undo Infinito:** Implementando Historial de Acciones en un Editor de Documentos Financieros |
| 14 | **El Formulario de 47 Campos:** Validación Reactiva en Onboarding Bancario |
| 15 | **Optimistic UI que Mintió:** Revertir Estados Cuando el Backend Rechaza |
| 16 | **Estado Global vs Local:** El Dilema del Saldo en Tiempo Real |
| 17 | **Wizard de 12 Pasos:** Persistir Progreso de Solicitud de Crédito Hipotecario |
| 18 | **El Dashboard que Colapsó:** Orquestar 15 Streams de Datos Simultáneos |
| 19 | **Conflicto de Merge en el Cliente:** Resolver Ediciones Concurrentes Offline |
| 20 | **State Machine Financiera:** Modelar el Ciclo de Vida de una Transferencia SPEI/ACH |

---

## Capítulo 3: Optimización de Rendimiento y Rendering (Casos 21-30)

| Caso | Título |
|:----:|:-------|
| 21 | **La Lista de 100,000 Transacciones:** Scroll Infinito sin Memory Leaks |
| 22 | **60 FPS o Muerte:** Animaciones Complejas en Pantallas de Trading |
| 23 | **El Catálogo Pesado:** Lazy Loading de Imágenes en MercadoLibre Scale |
| 24 | **Build Methods Costosos:** Identificar y Eliminar Rebuilds Innecesarios |
| 25 | **El Widget que Congeló el Hilo Principal:** Cómputo Pesado con Isolates |
| 26 | **Memoria que No Regresa:** Hunting Memory Leaks en Sesiones Largas |
| 27 | **Cold Start de 8 Segundos:** Optimizar Tiempo de Arranque en Apps Bancarias |
| 28 | **El Shader Compilation Jank:** Pre-warming de Shaders en Producción |
| 29 | **Tree Shaking Agresivo:** Reducir 40% del Bundle Size en Apps Enterprise |
| 30 | **Battery Drain Silencioso:** Optimizar Background Tasks sin Matar la Batería |

---

## Capítulo 4: Estrategias Offline-First y Sincronización (Casos 31-40)

| Caso | Título |
|:----:|:-------|
| 31 | **Vender en el Metro:** Cola de Transacciones Offline para POS Móvil |
| 32 | **Conflicto de Inventario:** Dos Vendedores, Un Producto, Cero Stock |
| 33 | **El Delta Sync Inteligente:** Sincronizar Solo lo Necesario en Redes 2G |
| 34 | **CRDT para Carritos:** Resolución Automática de Conflictos sin Servidor |
| 35 | **Background Sync Prohibido:** Estrategias cuando iOS Mata tu Proceso |
| 36 | **Cache Invalidation Hell:** Cuándo Confiar y Cuándo Descartar Datos Locales |
| 37 | **El Pedido Duplicado:** Idempotencia en Retry de Operaciones Críticas |
| 38 | **Version Vectors en Banca:** Tracking de Cambios Distribuidos |
| 39 | **Offline Login Seguro:** Autenticar sin Conexión sin Comprometer Seguridad |
| 40 | **El Sync que Tardó 3 Días:** Estrategia de Reconciliación Masiva Post-Desastre |

---

## Capítulo 5: Networking Avanzado e Integración de APIs (Casos 41-50)

| Caso | Título |
|:----:|:-------|
| 41 | **BFF: El Guardián del Móvil:** Orquestar 7 Microservicios en Una Llamada |
| 42 | **GraphQL Subscriptions en Trading:** Updates en Tiempo Real de Precios |
| 43 | **El Timeout que Quebró el Banco:** Configurar Retry Policies Inteligentes |
| 44 | **Circuit Breaker Móvil:** Proteger la App cuando el Backend Agoniza |
| 45 | **Request Deduplication:** Evitar Doble Cobro por Doble Tap |
| 46 | **Cursor vs Offset Pagination:** Navegar Millones de Productos Eficientemente |
| 47 | **Rate Limiting Graceful:** Degradar Funcionalidad sin Crashear |
| 48 | **API Versioning Hell:** Soportar 3 Versiones de API en Producción |
| 49 | **gRPC en Flutter:** Comunicación de Alta Performance para Fintech |
| 50 | **WebSocket Resiliente:** Mantener Conexión Viva en Chat de Soporte Bancario |

---

## Capítulo 6: Arquitectura Modular y Micro-frontends (Casos 51-60)

| Caso | Título |
|:----:|:-------|
| 51 | **Super App Architecture:** Cómo WeChat y Rappi Integran Mini-Apps |
| 52 | **Feature Flags en Banca:** Lanzar Funciones a 1% de Usuarios sin Deploy |
| 53 | **El Módulo que Pesaba 40MB:** Lazy Loading de Features por Demanda |
| 54 | **Dependency Injection at Scale:** GetIt vs Riverpod vs Injectable |
| 55 | **Comunicación entre Features:** Event Bus vs BLoC-to-BLoC |
| 56 | **Shared Kernel:** Extraer Código Común sin Crear un Monolito |
| 57 | **Plugin Architecture:** Permitir que Terceros Extiendan tu App Bancaria |
| 58 | **Mono-Repo con Melos:** Gestionar 15 Packages sin Perder la Cordura |
| 59 | **Build Time de 45 Minutos:** Estrategias de Compilación Incremental |
| 60 | **A/B Testing Arquitectónico:** Servir Diferentes UIs desde el Mismo Código |

---

## Capítulo 7: Integración Nativa y Platform Channels (Casos 61-70)

| Caso | Título |
|:----:|:-------|
| 61 | **El SDK Nativo Obligatorio:** Integrar Jumio KYC via Platform Channels |
| 62 | **Camera Custom para Cheques:** Captura Optimizada con APIs Nativas |
| 63 | **Background Location Legal:** Tracking de Entregas Cumpliendo Políticas |
| 64 | **Push Notifications Ricas:** Acciones Inline en iOS y Android |
| 65 | **Deep Linking Bancario:** Abrir la App en la Pantalla Correcta desde Email |
| 66 | **In-App Purchases Auditables:** Suscripciones con Receipt Validation |
| 67 | **El Sensor que iOS No Expone:** Acceder a Hardware via FFI |
| 68 | **PiP para Video Banking:** Picture-in-Picture en Videollamadas |
| 69 | **Widgets Nativos en Home:** Mostrar Saldo en Widget de iOS/Android |
| 70 | **Siri Shortcuts para Pagos:** "Hey Siri, Paga mi Tarjeta" |

---

## Capítulo 8: DevOps, CI/CD y Estrategia de Release (Casos 71-80)

| Caso | Título |
|:----:|:-------|
| 71 | **El Deploy del Viernes:** Por Qué Falló y Cómo Prevenirlo con Feature Flags |
| 72 | **Code Signing Nightmare:** Automatizar Certificados en Equipos de 50 Devs |
| 73 | **Flavors para 5 Bancos:** Una Codebase, Múltiples White-Labels |
| 74 | **El Hotfix de Emergencia:** Rollback Strategy cuando App Store Tarda 24h |
| 75 | **Coverage del 80%:** Estrategia Realista de Testing en CI/CD |
| 76 | **Canary Release Móvil:** Lanzar a 5% de Usuarios y Monitorear |
| 77 | **El Build que Rompió Producción:** Implementar Staged Rollouts |
| 78 | **Secrets Management:** No Más API Keys en el Repositorio |
| 79 | **App Size Budget:** Alertar Cuando el Bundle Supera 100MB |
| 80 | **Release Train Quincenal:** Coordinar 4 Squads sin Conflictos |

---

## Capítulo 9: Hardware, IoT y Biometría (Casos 81-90)

| Caso | Título |
|:----:|:-------|
| 81 | **NFC Tap-to-Pay:** Implementar Pagos Contactless Propietarios |
| 82 | **BLE para Beacons Bancarios:** Ofertas Personalizadas en Sucursales |
| 83 | **QR Dinámico Seguro:** Generar Códigos de Pago con Expiración |
| 84 | **El Wearable que Autoriza:** Apple Watch como Segundo Factor |
| 85 | **Liveness Detection:** Evitar Fotos de Fotos en Verificación Facial |
| 86 | **Voice Biometrics:** "Mi Voz es mi Contraseña" para Banca Telefónica |
| 87 | **Fingerprint Fallback:** Qué Hacer Cuando el Sensor Falla |
| 88 | **Geofencing Inteligente:** Bloquear Transacciones Fuera de Zona |
| 89 | **Acelerómetro Anti-Fraude:** Detectar Comportamiento de Bot |
| 90 | **Thermal Camera Integration:** Detección de Fiebre en Apps de Salud |

---

## Capítulo 10: Migración de Legacy y Gestión de Deuda Técnica (Casos 91-100)

| Caso | Título |
|:----:|:-------|
| 91 | **Strangler Fig Pattern:** Migrar App Nativa de 8 Años sin Big Bang |
| 92 | **El Monolito de 500K Líneas:** Estrategia de Modularización Gradual |
| 93 | **Bridge Pattern:** Mantener Kotlin y Flutter en Coexistencia |
| 94 | **Data Migration Nocturna:** Mover 2M de Usuarios sin Downtime |
| 95 | **API Compatibility Layer:** Soportar Clientes v1, v2 y v3 Simultáneamente |
| 96 | **Testing de Regresión Legacy:** Crear Tests para Código sin Tests |
| 97 | **El Tercero que Cerró:** Reemplazar SDK Deprecated en 30 Días |
| 98 | **Deuda Técnica Cuantificada:** Métricas para Convencer al Negocio |
| 99 | **Feature Parity Dashboard:** Tracking de Migración Funcionalidad por Funcionalidad |
| 100 | **El Día que Apagamos el Legacy:** Checklist de Decommissioning Seguro |

---

# CAPÍTULO 1: SEGURIDAD BANCARIA Y GESTIÓN DE IDENTIDAD

---

## Caso 1: El Token que Nunca Expira — Cómo una Sesión Zombie Costó $2.4M en Fraude

### 0. Metadata para Indexación (AI-Tags)
* **Palabras Clave de Negocio:** refresh token, sesión persistente, fraude bancario, logout automático, robo de credenciales
* **Patrón Técnico:** Token Rotation, Sliding Session, Secure Token Storage
* **Stack Seleccionado:** flutter_secure_storage + Dio Interceptors + BLoC (AuthBloc)
* **Nivel de Criticidad:** Alto

### 1. Planteamiento del Problema (El "Trigger")

**Escenario de Negocio:**
> *"Como usuario de banca móvil, quiero mantener mi sesión iniciada para no tener que autenticarme cada vez que abro la app."*

Esta historia de usuario, aparentemente inocente, esconde uno de los vectores de ataque más explotados en aplicaciones financieras: la gestión inadecuada del ciclo de vida de tokens.

**Evidencia de Industria:**

**Caso Revolut 2022:** En septiembre de 2022, Revolut sufrió una brecha que expuso datos de 50,000+ usuarios. El ataque de ingeniería social fue posible parcialmente porque tokens de sesión comprometidos no fueron invalidados oportunamente. La compañía tardó horas en detectar sesiones anómalas porque carecían de mecanismos robustos de rotación y revocación.

**Caso Capital One 2019:** Aunque el vector principal fue una misconfiguration de WAF, la investigación reveló que tokens de servicio tenían lifetimes excesivamente largos, ampliando la ventana de explotación.

**Estadística crítica:** Según el reporte de Verizon DBIR 2023, el 74% de las brechas financieras involucran credenciales comprometidas, y el tiempo promedio de detección de sesiones fraudulentas es de 207 días.

**Riesgos:**

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Fraude directo ($2.4M promedio en incidentes de session hijacking según IBM Cost of Data Breach 2023), multas regulatorias (hasta 4% de revenue global bajo GDPR, sanciones de CNBV/Banco Central) |
| **Reputacional** | Pérdida de confianza, churn de clientes (estudios muestran 65% de usuarios abandonan servicios financieros tras brecha de seguridad) |
| **Técnico** | Deuda de seguridad acumulada, necesidad de reescritura completa del módulo de autenticación, auditorías SOC 2 fallidas |

### 2. Matriz de Soluciones y Selección de Herramientas

| Rol | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:----|:-----------------------|:----------------------------------|
| **Junior** | `SharedPreferences` para guardar el token, sin expiración, refresh manual cuando falla una request | **FALLA:** Tokens en texto plano accesibles via backup de Android, sin rotación automática, vulnerable a replay attacks. No detecta dispositivos rooteados. Un atacante con acceso físico al dispositivo extrae el token en 30 segundos con `adb backup`. |
| **Senior** | `flutter_secure_storage` + token con TTL de 15 min + refresh token con TTL de 7 días + interceptor Dio para auto-refresh | **MEJORA:** Cifrado AES-256 en Keychain/Keystore, automatización del refresh, pero carece de: rotación de refresh tokens, detección de anomalías, logout remoto. Un token robado tiene ventana de 7 días. |
| **Architect** | `flutter_secure_storage` + **Token Rotation** (nuevo refresh token en cada uso) + **BLoC para estado de auth** + Device Binding + Anomaly Detection + Remote Revocation via FCM | **ENTERPRISE:** Cada refresh genera nuevo refresh token e invalida el anterior. Vinculación dispositivo-token. Detección de uso simultáneo en múltiples dispositivos. Capacidad de logout remoto instantáneo. Cumple PSD2/SCA. |

### 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Almacenamiento cifrado con hardware security module (HSM) en dispositivos compatibles. Rotación automática de refresh tokens sin fricción de usuario. Invalidación remota de sesiones específicas o todas las sesiones. Binding de token a device fingerprint (ANDROID_ID + modelo + instalación única). Detección de jailbreak/root antes de permitir operaciones sensibles. |
| **Restricciones Duras (NO permite)** | **iOS Keychain:** Accesible después de primer unlock del dispositivo (no "always available"). **Android Keystore:** Requiere API 23+ para cifrado respaldado por hardware. **Offline:** No puede validar revocación de tokens sin conexión (requiere estrategia de TTL corto). **Backup:** flutter_secure_storage marca datos como excluded from backup pero el usuario puede tener herramientas de extracción. |
| **Criterio de Selección** | Se usa **flutter_secure_storage** sobre Hive/SharedPreferences porque: 1) Utiliza Keychain (iOS) y EncryptedSharedPreferences/Keystore (Android) respaldados por hardware cuando disponible. 2) Los tokens son datos de alta sensibilidad que requieren cifrado at-rest por regulación. 3) Hive cifrado requiere gestionar la key de cifrado (chicken-egg problem). Se usa **BLoC** sobre Riverpod/Provider para AuthState porque: auditabilidad de transiciones (login→authenticated→refreshing→error), logging de eventos para forensics, y separación clara de lógica de negocio. |

### 4. Manos a la Obra: Estrategia de Implementación

#### Fase 1: Diseño — Contratos y Arquitectura

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

#### Fase 2: Implementación — Detalles Técnicos

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

#### Fase 3: Observability — Métricas y Alertas

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

### 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

- [ ] **TAC-1.1:** Los tokens DEBEN almacenarse exclusivamente usando `flutter_secure_storage` con configuración de seguridad máxima para cada plataforma.
- [ ] **TAC-1.2:** El access token DEBE tener TTL máximo de 15 minutos. El refresh token DEBE tener TTL máximo de 7 días.
- [ ] **TAC-1.3:** DEBE implementarse Token Rotation: cada refresh exitoso genera nuevo refresh token e invalida el anterior.
- [ ] **TAC-1.4:** El sistema DEBE detectar y rechazar tokens de familia ya revocada (token reuse attack), forzando logout completo.
- [ ] **TAC-1.5:** El interceptor de Dio DEBE realizar refresh proactivo cuando access token tenga < 30 segundos de vida.
- [ ] **TAC-1.6:** DEBE existir mecanismo de logout remoto via FCM/Push que invalide sesiones en < 5 segundos.
- [ ] **TAC-1.7:** Cada evento de autenticación DEBE generar log de auditoría con: timestamp, device_fingerprint, ip_address, resultado, session_family_id.
- [ ] **TAC-1.8:** El tiempo de respuesta del endpoint `/auth/refresh` NO DEBE superar 500ms en p95.
- [ ] **TAC-1.9:** DEBE implementarse device binding: tokens vinculados a fingerprint de dispositivo.
- [ ] **TAC-1.10:** La aplicación DEBE limpiar TODOS los tokens locales cuando detecta dispositivo rooteado/jailbroken.

### 6. Estrategia de Pruebas (Shift-Left)

**Stack de Testing:**
- **Unit:** `flutter_test`, `bloc_test`, `mocktail`
- **Integration:** `integration_test`, `mockito`
- **E2E:** `patrol` para flujos completos con UI

**Escenarios Críticos Obligatorios:**

| # | Escenario | Qué Validar | Tipo |
|:-:|:----------|:------------|:-----|
| 1 | **Refresh token expirado en background** | Al volver a foreground: detectar expiración, limpiar storage, redirigir a login SIN crash. No intentar refresh infinito. | Integration |
| 2 | **Token reuse attack** | Cuando backend responde `token_family_revoked`: limpiar tokens, enviar evento seguridad, mostrar mensaje claro, NO permitir retry. | Unit + Integration |
| 3 | **Múltiples requests con token expirado** | Solo UNA request de refresh debe ejecutarse. Las demás esperan. Si falla, todas fallan consistentemente. | Unit (QueuedInterceptor) |

---

## Caso 2: Biometría Falsificada — Cuando Face ID No Es Suficiente para Aprobar un Crédito

### 0. Metadata para Indexación (AI-Tags)
* **Palabras Clave de Negocio:** autenticación biométrica, Face ID, Touch ID, aprobación de crédito, fraude de identidad, liveness detection
* **Patrón Técnico:** Multi-Layer Biometric Authentication, Step-up Authentication, Liveness Detection
* **Stack Seleccionado:** local_auth + platform channels para SDK nativo (Facetec/iProov) + Cubit (BiometricCubit)
* **Nivel de Criticidad:** Alto

### 1. Planteamiento del Problema (El "Trigger")

**Escenario de Negocio:**
> *"Como usuario, quiero aprobar mi solicitud de crédito de $50,000 usando Face ID para no tener que ir a una sucursal."*

El problema: Face ID/Touch ID del sistema operativo solo verifica que "un rostro/huella registrado en el dispositivo" coincide. NO verifica que sea el rostro del titular de la cuenta bancaria, ni que sea una persona viva frente a la cámara.

**Evidencia de Industria:**

**Caso Deepfake Banking Fraud (2023):** Un grupo criminal en Hong Kong usó deepfakes para engañar sistemas de verificación facial en múltiples bancos, obteniendo préstamos por más de $25 millones. Los atacantes usaron fotos de redes sociales para generar videos sintéticos que pasaron verificación de "selfie".

**Ataque de Presentation Attack (Spoofing):** En 2021, investigadores de seguridad demostraron que el 20% de los sistemas de reconocimiento facial en apps bancarias europeas podían ser engañados con una foto impresa de alta resolución.

**Regulación PSD2/SCA:** La directiva PSD2 requiere Strong Customer Authentication (SCA) para transacciones que superen €30. La autenticación biométrica DEBE incluir protección contra ataques de presentación según estándares EBA.

**Riesgos:**

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Pérdidas por fraude en préstamos ($50K-500K por incidente), chargebacks, provisiones por cartera vencida |
| **Regulatorio** | Incumplimiento de KYC/AML puede resultar en multas de $1M-100M |
| **Reputacional** | Un caso viral de "deepfake loan fraud" destruye confianza en canal digital |

### 2. Matriz de Soluciones y Selección de Herramientas

| Rol | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:----|:-----------------------|:----------------------------------|
| **Junior** | `local_auth` para Face ID/Touch ID, si pasa → aprobado | **FALLA CRÍTICA:** Solo verifica biometría del dispositivo. No verifica que sea el titular. No detecta fotos/videos. Cualquier persona con acceso al dispositivo puede aprobar. |
| **Senior** | `local_auth` como primer factor + captura de selfie + envío a backend para comparación con foto de documento | **MEJORA PARCIAL:** Agrega verificación de identidad. Pero: vulnerable a fotos impresas, latencia alta, dependencia de conectividad. |
| **Architect** | **Autenticación en capas:** 1) `local_auth` gate inicial, 2) SDK certificado Liveness (Facetec/iProov) via Platform Channels, 3) Comparación 1:1 contra KYC, 4) Challenge-Response, 5) Cubit para orquestar | **ENTERPRISE:** Detecta spoofing 2D/3D. Certificación iBeta/NIST. Evidencia auditable con firma criptográfica. |

### 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Detección de ataques 2D (fotos, pantallas) y 3D (máscaras). Verificación de liveness con challenge-response. Comparación 1:1 selfie vs documento. FaceMap encriptado para auditoría. Funciona con cámara frontal estándar. Cumple ISO 30107-3 y NIST FRVT. |
| **Restricciones Duras (NO permite)** | **local_auth SOLO:** No distingue entre titular y otra persona del dispositivo. **Sin red:** Algunos SDKs requieren validación server-side. **Condiciones extremas:** Liveness falla con luz < 50 lux, lentes muy oscuros, barbas > 50% rostro. **Simulador:** Biometría no disponible en simulador. |
| **Criterio de Selección** | Se usa **Cubit** en lugar de BLoC porque el flujo es secuencial sin eventos complejos (captura→análisis→resultado). Cubit reduce boilerplate. Se usan **Platform Channels** para SDK nativo porque Facetec/iProov solo existen en Kotlin/Swift, procesamiento de imagen más eficiente nativo. |

### 4. Manos a la Obra: Estrategia de Implementación

#### Fase 1: Diseño — Arquitectura de Autenticación en Capas

**Flujo de Autenticación Alto Riesgo:**
1. `local_auth` → Gate rápido (< 1 seg)
2. Liveness Detection → SDK nativo Facetec
3. Face Match vs KYC → API de comparación server-side
4. Challenge Response → Para montos > umbral
5. Final Approval + Audit → Generar evidencia firmada

**Estructura de Carpetas:**
```
lib/features/biometric_auth/
├── data/
│   └── datasources/
│       ├── biometric_local_datasource.dart   # local_auth wrapper
│       ├── liveness_native_datasource.dart   # Platform Channel
│       └── face_match_remote_datasource.dart # API call
├── domain/
│   ├── entities/
│   │   ├── biometric_result.dart
│   │   └── liveness_evidence.dart
│   └── usecases/
│       ├── check_device_biometrics.dart
│       ├── perform_liveness_check.dart
│       └── verify_face_match.dart
└── presentation/
    ├── cubit/
    │   ├── biometric_auth_cubit.dart
    │   └── biometric_auth_state.dart
    └── widgets/
        ├── biometric_prompt.dart
        └── liveness_camera_overlay.dart
```

#### Fase 2: Implementación — Detalles Técnicos

**BiometricAuthState:**
- Enum de pasos: `initial`, `checkingCapability`, `requestingBiometric`, `capturingLiveness`, `processingLiveness`, `matchingFace`, `waitingChallenge`, `success`, `failure`
- Campos: `errorMessage`, `errorType`, `livenessConfidence`, `faceMatchScore`, `auditToken`, `challengeInstruction`
- Helper `isProcessing` para UI loading states

**BiometricAuthCubit - Flujo Principal:**
1. `checkDeviceCapability`: Verificar `canCheckBiometrics`, `isDeviceSupported`, `getAvailableBiometrics`
2. `performDeviceBiometric`: `local_auth.authenticate` con `biometricOnly: true`
3. `performLivenessCheck`: Invocar Platform Channel al SDK nativo
4. `performFaceMatch`: Enviar `faceMap` a API, umbral de confianza ≥ 0.85
5. `performChallenge`: Solo para montos > $10,000

**LivenessNativeDataSource (Platform Channel):**
- MethodChannel: `com.bank.app/liveness`
- Método `startLivenessCheck`: timeout 30s, minConfidence 0.9, enableAudit true
- Retorna: `isAlive`, `confidence`, `faceMap` (encriptado), `auditImage`, `wasSpoofAttempt`, `spoofType`

**Código Nativo iOS/Android:**
- Integrar SDK Facetec/iProov siguiendo documentación oficial
- Manejar lifecycle de cámara y permisos
- Retornar resultados via FlutterResult

#### Fase 3: Observability — Métricas y KPIs

**Métricas Críticas:**
- Funnel: `auth.started` → `device.passed` → `local.passed` → `liveness.passed` → `match.passed` → `completed`
- Seguridad: `spoofing_detected`, `face_not_matched`
- Performance: `liveness_latency_ms`, `match_latency_ms`, `total_auth_time_ms`

**KPIs de Éxito:**

| Métrica | Objetivo | Alerta |
|:--------|:---------|:-------|
| Tasa de completación | > 85% | < 70% → Investigar UX |
| Liveness false rejection | < 5% | > 10% → Ajustar umbral |
| Spoofing detection rate | > 99% | < 95% → Crítico |
| Tiempo total auth | < 15 seg | > 25 seg → Optimizar |

### 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

- [ ] **TAC-2.1:** Para operaciones de alto riesgo (préstamos, transferencias > $5,000), `local_auth` NO ES SUFICIENTE. Se requiere liveness adicional.
- [ ] **TAC-2.2:** El SDK de Liveness DEBE tener certificación ISO 30107-3 Level 2 o superior.
- [ ] **TAC-2.3:** El sistema DEBE detectar y rechazar ataques de presentación 2D con tasa > 99%.
- [ ] **TAC-2.4:** El score de face match contra documento KYC DEBE ser ≥ 0.85.
- [ ] **TAC-2.5:** Cada verificación exitosa DEBE generar audit token firmado criptográficamente.
- [ ] **TAC-2.6:** El tiempo total del flujo NO DEBE exceder 20 segundos en p95.
- [ ] **TAC-2.7:** Ante spoofing detectado: bloquear operación, generar alerta, NO revelar detección al usuario.
- [ ] **TAC-2.8:** El flujo DEBE funcionar en modo degradado (solo local_auth) para operaciones de bajo riesgo sin conectividad.
- [ ] **TAC-2.9:** DEBE existir fallback a verificación humana (videollamada) tras 3 intentos fallidos.
- [ ] **TAC-2.10:** Los datos biométricos (faceMap) NUNCA se almacenan localmente sin cifrado, TTL < 5 minutos.

### 6. Estrategia de Pruebas (Shift-Left)

**Stack de Testing:**
- **Unit:** `flutter_test`, `mocktail` para Cubit
- **Integration:** `integration_test` + dispositivo físico
- **E2E:** `maestro` para flujos visuales
- **Security:** Tests de penetración con fotos/videos (lab especializado)

**Escenarios Críticos Obligatorios:**

| # | Escenario | Qué Validar | Tipo |
|:-:|:----------|:------------|:-----|
| 1 | **Ataque de foto impresa** | Sistema DEBE detectar spoofing y rechazar con `spoofType: "2d_photo"`. NO dar pistas al atacante. | Security (Manual) |
| 2 | **Usuario sin biometría enrollada** | Fallo graceful en paso 1, mensaje claro, ofrecer alternativa (PIN + OTP). No crashear. | Integration |
| 3 | **Timeout de red en face match** | Liveness local OK → timeout en match → reintentar 2x con backoff, luego "intentar más tarde" SIN perder progreso. | Integration |

---

## Caso 3: El Man-in-the-Middle Silencioso — Certificate Pinning en Transferencias Interbancarias

### 0. Metadata para Indexación (AI-Tags)
* **Palabras Clave de Negocio:** certificate pinning, SSL pinning, MITM attack, transferencia interbancaria, SPEI, ACH, seguridad en tránsito
* **Patrón Técnico:** Certificate Pinning, Public Key Pinning, Trust on First Use (TOFU)
* **Stack Seleccionado:** Dio + dio_http2_adapter + certificados hardcodeados + BLoC (TransferBloc)
* **Nivel de Criticidad:** Alto

### 1. Planteamiento del Problema (El "Trigger")

**Escenario de Negocio:**
> *"Como usuario, quiero realizar transferencias interbancarias desde la app móvil con la confianza de que mis datos están protegidos en tránsito."*

El problema real: HTTPS estándar confía en cualquier Certificate Authority (CA) instalado en el dispositivo. Un atacante que instala un certificado malicioso puede interceptar todo el tráfico "cifrado".

**Evidencia de Industria:**

**Caso DigiNotar 2011:** Una CA holandesa fue comprometida, emitiendo certificados fraudulentos para google.com, permitiendo ataques MITM masivos en Irán.

**Operación Socialist (GCHQ):** Documentos filtrados revelaron uso de proxies MITM con certificados válidos para interceptar tráfico de apps bancarias.

**Estudio académico (2019):** El 73% de las apps bancarias Android no implementaban certificate pinning correctamente.

**Riesgos:**

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Transferencias fraudulentas interceptadas, robo de credenciales |
| **Regulatorio** | Incumplimiento PCI-DSS Req. 4.1 (cifrado en tránsito) |
| **Legal** | Responsabilidad del banco si no tomó medidas "razonables" |

### 2. Matriz de Soluciones y Selección de Herramientas

| Rol | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:----|:-----------------------|:----------------------------------|
| **Junior** | HTTPS estándar con Dio, confiar en CAs del sistema | **FALLA:** Vulnerable a cualquier CA comprometida. Proxy (Charles, mitmproxy) + certificado instalado ve todo en texto plano. |
| **Senior** | Certificate pinning con `SecurityContext` y certificado en assets | **MEJORA:** Bloquea CAs desconocidas. PERO: si certificado expira, app muere. Sin actualización dinámica. |
| **Architect** | **Multi-layer pinning:** 1) Pin de Public Key (no certificado), 2) Backup pins para rollover, 3) Fallback con alertas, 4) Actualización dinámica via config firmada, 5) Detección de proxy | **ENTERPRISE:** Resistente a rotación de certs, detecta intercepción, alerta al SOC, cumple PCI-DSS. |

### 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Bloquear certificados no autorizados incluso de CAs "confiables". Sobrevivir rotación de certs con pin de Public Key (SPKI). Detectar proxies de debug. Actualizar pins sin release via config firmada. Backup pins para rollover. |
| **Restricciones Duras (NO permite)** | **Debugging:** Certificate pinning dificulta debug (deshabilitar en debug builds). **Proxies corporativos:** Usuarios en redes con proxy SSL legítimo serán bloqueados. **Expiración:** Si todos los pins expiran, app inoperable. **iOS ATS:** Puede conflictuar. |
| **Criterio de Selección** | Se usa **Public Key pinning** sobre certificate pinning porque las claves públicas sobreviven renovaciones si se usa misma key pair. Se usa **BLoC** para TransferBloc porque transferencias tienen flujo complejo con múltiples eventos y necesitan auditabilidad. |

### 4. Manos a la Obra: Estrategia de Implementación

#### Fase 1: Diseño — Estrategia de Pins

**Arquitectura de Pins:**
- Primary Pin: Certificado de producción actual (SHA-256 de SPKI en Base64)
- Backup Pin: Próximo certificado (pre-staged para rotación)
- Emergency Pin: Clave de DR (offline, en HSM)
- Fallback Chain: Primary → Backup → Emergency → FAIL + ALERT

**Dominios a Pinear:**
- `api.bank.com`
- `auth.bank.com`
- `transfers.bank.com`

**Estructura de Carpetas:**
```
lib/core/
├── network/
│   ├── certificate_pinner.dart
│   ├── pinning_config.dart
│   ├── ssl_pinning_interceptor.dart
│   └── proxy_detector.dart
└── security/
    └── remote_config_validator.dart
```

#### Fase 2: Implementación — Detalles Técnicos

**PinningConfig:**
- Lista de pins hardcodeados como baseline (SHA-256 de SPKI)
- Obtener pins con: `openssl s_client -connect api.bank.com:443 | openssl x509 -pubkey -noout | openssl pkey -pubin -outform der | openssl dgst -sha256 -binary | base64`
- Lista de dominios a pinear
- `maxPinAge`: Tiempo máximo sin validar pins remotos (30 días)

**CertificatePinner:**
- `validateCertificate(X509Certificate cert, String host)`: Validar contra pins
- `_shouldPinDomain(host)`: Solo validar dominios configurados
- `_extractPublicKeyHash(cert)`: Extraer SPKI, calcular SHA-256, encodear Base64
- `_logPinningViolation`: Log para análisis forense (timestamp, host, receivedPin, expectedPins, cert info)

**SSLPinningInterceptor (Dio):**
- En `onRequest`: Detectar proxy con `ProxyDetector`, bloquear endpoints críticos si proxy detectado
- En `onError`: Detectar errores SSL/TLS, transformar a mensaje amigable
- Configurar `IOHttpClientAdapter` con `badCertificateCallback` que siempre rechaza

**ProxyDetector:**
- Verificar variables de entorno (`http_proxy`, `HTTP_PROXY`, `https_proxy`, `HTTPS_PROXY`)
- Verificar VPN activa (requiere platform channel)
- Método `isDebugMode()` para detectar herramientas de reversing

**RemotePinConfig:**
- Obtener pins de servidor con config firmada
- Verificar firma ECDSA/RSA con clave pública hardcodeada
- Cache en `flutter_secure_storage` con TTL
- Fallback a pins hardcodeados si falla red

#### Fase 3: Observability — Métricas y Alertas

**Métricas Críticas:**
- `ssl.pinning.success` / `ssl.pinning.failure`
- `ssl.proxy.detected`
- `ssl.cert.bad`
- `ssl.config.tampered`
- `ssl.handshake.latency_ms`

**Alertas:**

| Evento | Severidad | Acción |
|:-------|:----------|:-------|
| `pinningFailure` con pin desconocido | P1 Critical | Despertar equipo seguridad |
| `proxyDetected` en endpoints críticos | P2 High | Investigar horario laboral |
| `configTampered` | P1 Critical | Posible ataque infraestructura |

### 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

- [ ] **TAC-3.1:** DEBE implementarse Public Key Pinning para todos los endpoints críticos (auth, transfers, payments).
- [ ] **TAC-3.2:** DEBE mantener mínimo 2 backup pins para soportar rotación sin disrupciones.
- [ ] **TAC-3.3:** Ante fallo de pinning, conexión DEBE rechazarse. NO fallback a conexión no pineada.
- [ ] **TAC-3.4:** Cada fallo de pinning DEBE generar alerta con: host, pin recibido, pins esperados, timestamp, device_id.
- [ ] **TAC-3.5:** DEBE existir mecanismo de actualización de pins via config remota firmada criptográficamente.
- [ ] **TAC-3.6:** DEBE detectar proxy y bloquear operaciones críticas cuando se detecte.
- [ ] **TAC-3.7:** En builds DEBUG, pinning puede deshabilitarse. En RELEASE, DEBE estar siempre activo.
- [ ] **TAC-3.8:** Pins hardcodeados DEBEN rotarse mínimo cada 6 meses.
- [ ] **TAC-3.9:** DEBE existir runbook para rotación de emergencia de certificados.
- [ ] **TAC-3.10:** Métricas de pinning DEBEN monitorearse con alertas automáticas.

### 6. Estrategia de Pruebas (Shift-Left)

**Stack de Testing:**
- **Unit:** `flutter_test`, `mocktail` para CertificatePinner
- **Integration:** `integration_test` con servidor mock
- **Security:** Pruebas manuales con mitmproxy, Charles Proxy, Burp Suite

**Escenarios Críticos Obligatorios:**

| # | Escenario | Qué Validar | Tipo |
|:-:|:----------|:------------|:-----|
| 1 | **MITM con Charles Proxy** | Configurar Charles, instalar certificado. Requests a endpoints pineados DEBEN fallar. | Security (Manual) |
| 2 | **Rotación de certificado** | Agregar nuevo pin a backup, rotar cert en servidor. App DEBE seguir funcionando con backup pin. | Integration |
| 3 | **Todos los pins expirados** | Ningún pin válido. App DEBE mostrar error claro, enviar alerta crítica, NO conectar sin pinning. | Unit + Integration |

---

## Caso 4: Rooted pero Confiable — Detectar Dispositivos Comprometidos sin Bloquear Usuarios Legítimos

### 0. Metadata para Indexación (AI-Tags)
* **Palabras Clave de Negocio:** root detection, jailbreak detection, dispositivo comprometido, fraude, seguridad móvil, SafetyNet, Play Integrity
* **Patrón Técnico:** Device Attestation, Risk-Based Authentication, Layered Security
* **Stack Seleccionado:** flutter_jailbreak_detection + Platform Channels (SafetyNet/Play Integrity) + Riverpod (DeviceSecurityProvider)
* **Nivel de Criticidad:** Alto

### 1. Planteamiento del Problema (El "Trigger")

**Escenario de Negocio:**
> *"Como oficial de seguridad del banco, quiero detectar dispositivos rooteados/jailbroken para evaluar el riesgo, pero sin bloquear usuarios legítimos que usan custom ROMs por privacidad."*

El dilema: Bloquear 100% de dispositivos rooteados genera falsos positivos (desarrolladores, usuarios privacy-conscious). No detectarlos abre la puerta a malware y hooking.

**Evidencia de Industria:**

**Caso Cerberus Banking Trojan (2020-2023):** Malware Android que requiere root. Intercepta SMS OTP, captura credenciales via overlay attacks, extrae datos de apps bancarias. Ha afectado bancos en España, México, Brasil.

**Magisk "MagiskHide" (ahora Zygisk):** Permite ocultar root de apps específicas. Usuarios legítimos la usan, pero también atacantes para evadir detección.

**Estudio Promon (2022):** El 50% de las apps bancarias top 100 pueden tener protecciones bypaseadas en dispositivos rooteados con Frida/Xposed.

**Estadística clave:** Solo ~2% de dispositivos están rooteados, pero representan ~30% de intentos de fraude en apps bancarias.

**Riesgos:**

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Fraude vía malware en dispositivos rooteados, ATO |
| **False Positives** | Bloquear usuarios legítimos genera churn |
| **Regulatory** | Algunos reguladores exigen detección de dispositivos comprometidos |

### 2. Matriz de Soluciones y Selección de Herramientas

| Rol | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:----|:-----------------------|:----------------------------------|
| **Junior** | `flutter_jailbreak_detection` → si detecta root → bloquear app | **FALLA:** Falsos positivos, fácilmente bypasseable con Magisk Hide, no distingue riesgo real, UX hostil. |
| **Senior** | Múltiples checks (su binary, build tags) + warning pero permitir con funcionalidad reducida | **MEJORA:** Reduce falsos positivos. PERO: aún bypasseable, sin verificación server-side, atacante puede parchear binario. |
| **Architect** | **Device Attestation multi-capa:** 1) Checks locales, 2) Play Integrity/SafetyNet (Android) y DeviceCheck/App Attest (iOS) server-side, 3) Risk scoring dinámico, 4) Respuesta adaptativa | **ENTERPRISE:** Verificación criptográfica server-side. Difícil bypassear. Políticas granulares por operación. |

### 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Detectar root/jailbreak con múltiples técnicas. Verificación criptográfica server-side (Play Integrity). Risk scoring continuo. Respuestas granulares (permitir lectura, bloquear escritura). Detectar hooking frameworks (Frida, Xposed). |
| **Restricciones Duras (NO permite)** | **100% infalible:** Siempre existirán bypasses con suficiente esfuerzo. **Offline:** Play Integrity requiere conexión. **Emuladores:** Detectados como riesgo, afecta devs. **Play Services:** Play Integrity requiere Google Play Services. |
| **Criterio de Selección** | Se usa **Riverpod** con `DeviceSecurityProvider` porque: estado de seguridad se consulta desde múltiples features, Riverpod permite dependency injection limpio, fácil testing con overrides. Play Integrity sobre SafetyNet porque SafetyNet está deprecated. |

### 4. Manos a la Obra: Estrategia de Implementación

#### Fase 1: Diseño — Modelo de Riesgo Adaptativo

**Niveles de Riesgo:**
- `secure`: Dispositivo limpio, todas las verificaciones pasaron
- `elevated`: Indicadores sospechosos pero no confirmados
- `high`: Root/jailbreak detectado con checks básicos
- `critical`: Hooking framework activo o attestation fallido

**Matriz de Respuesta por Operación:**

| Operación | secure | elevated | high | critical |
|:----------|:-------|:---------|:-----|:---------|
| Ver saldo | ✓ | ✓ | ✓ | ✓ |
| Transferir < $100 | ✓ | ✓ | ⚠️ OTP | ✗ |
| Transferir > $1000 | ✓ | ⚠️ OTP | ✗ | ✗ |
| Cambiar contraseña | ✓ | ✓ | ⚠️ MFA | ✗ |
| Agregar beneficiario | ✓ | ⚠️ OTP | ✗ | ✗ |

**Estructura de Carpetas:**
```
lib/core/security/
├── device_security/
│   ├── data/
│   │   ├── datasources/
│   │   │   ├── local_root_detection.dart
│   │   │   ├── play_integrity_datasource.dart
│   │   │   └── ios_attestation_datasource.dart
│   │   └── repositories/
│   │       └── device_security_repository_impl.dart
│   ├── domain/
│   │   ├── entities/
│   │   │   ├── device_risk_level.dart
│   │   │   └── security_verdict.dart
│   │   └── usecases/
│   │       └── evaluate_device_security.dart
│   └── presentation/
│       └── providers/
│           └── device_security_provider.dart
```

#### Fase 2: Implementación — Detalles Técnicos

**LocalRootDetection (Checks Locales):**
- Verificar existencia de binarios: `/system/bin/su`, `/system/xbin/su`, `/sbin/su`
- Verificar build tags: `test-keys` en `android.os.Build.TAGS`
- Verificar propiedades: `ro.debuggable`, `ro.secure`
- Verificar apps conocidas: Superuser.apk, Magisk Manager
- Verificar directorios escribibles: `/system`, `/data`
- Detectar hooks: verificar integridad de funciones críticas

**PlayIntegrityDatasource (Android):**
- Usar `play_integrity` package o Platform Channel
- Solicitar integrity token con nonce único por request
- Enviar token a backend para verificación
- Backend valida con Google API y retorna verdict
- Evaluar: `deviceRecognitionVerdict`, `appRecognitionVerdict`, `accountDetails`

**iOSAttestationDatasource:**
- App Attest: `DCAppAttestService.shared.attestKey`
- DeviceCheck: `DCDevice.current.generateToken`
- Ambos requieren validación server-side con Apple API

**DeviceSecurityProvider (Riverpod):**
- `AsyncNotifierProvider` para estado de seguridad
- Evaluar al inicio de app y periódicamente
- Cachear resultado con TTL (5-15 minutos)
- Exponer `riskLevel` y `canPerformOperation(operation)`
- Notificar cambios para re-evaluación de UI

**RiskEvaluator:**
- Combinar resultados de todas las fuentes
- Pesos: Play Integrity (40%), Local Checks (30%), Behavioral (30%)
- Calcular score numérico y mapear a nivel de riesgo
- Considerar historial: múltiples fallos → elevar riesgo

#### Fase 3: Observability — Métricas y Alertas

**Métricas:**
- `device.security.risk_level` (distribución por nivel)
- `device.security.root_detected`
- `device.security.attestation_failed`
- `device.security.operation_blocked`
- `device.security.frida_detected` (CRÍTICO)

**Alertas:**

| Evento | Acción |
|:-------|:-------|
| `frida_detected` | Alerta inmediata a SOC |
| Pico de `attestation_failed` | Investigar posible ataque coordinado |
| Usuario legítimo bloqueado repetidamente | Revisar falsos positivos |

### 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

- [ ] **TAC-4.1:** El sistema DEBE implementar detección de root/jailbreak en múltiples capas (local + server-side attestation).
- [ ] **TAC-4.2:** La respuesta a dispositivos comprometidos DEBE ser adaptativa según tipo de operación, NO binaria.
- [ ] **TAC-4.3:** Para operaciones de alto riesgo (transferencias > $1000), DEBE usarse Play Integrity (Android) o App Attest (iOS).
- [ ] **TAC-4.4:** El resultado de attestation DEBE validarse server-side, NUNCA confiar solo en respuesta local.
- [ ] **TAC-4.5:** El sistema DEBE detectar hooking frameworks activos (Frida, Xposed, Substrate).
- [ ] **TAC-4.6:** Ante detección de Frida/hooking activo, BLOQUEAR todas las operaciones financieras inmediatamente.
- [ ] **TAC-4.7:** El nivel de riesgo DEBE re-evaluarse periódicamente (cada 5-15 minutos) durante sesión activa.
- [ ] **TAC-4.8:** DEBE existir mecanismo de apelación para usuarios bloqueados incorrectamente.
- [ ] **TAC-4.9:** Las métricas de dispositivos comprometidos DEBEN alimentar modelos de fraude.
- [ ] **TAC-4.10:** NUNCA revelar al usuario QUÉ check específico falló (no educar al atacante).

### 6. Estrategia de Pruebas (Shift-Left)

**Stack de Testing:**
- **Unit:** `flutter_test`, `mocktail` para providers
- **Integration:** Dispositivo rooteado real + dispositivo limpio
- **Security:** Tests con Frida, Magisk, Xposed

**Escenarios Críticos Obligatorios:**

| # | Escenario | Qué Validar | Tipo |
|:-:|:----------|:------------|:-----|
| 1 | **Magisk con MagiskHide activo** | Checks locales pueden fallar, pero Play Integrity DEBE detectar. Backend debe recibir riesgo elevado. | Security (Manual) |
| 2 | **Frida attach en runtime** | App DEBE detectar y bloquear operaciones. Alerta a SOC. | Security (Manual) |
| 3 | **Dispositivo limpio sin Play Services** | NO bloquear, usar checks locales. Operaciones de bajo riesgo permitidas. | Integration |

---

## Caso 5: Keyloggers en el Teclado Virtual — Protegiendo PINs en Apps de Banca Móvil

### 0. Metadata para Indexación (AI-Tags)
* **Palabras Clave de Negocio:** PIN seguro, teclado virtual, keylogger, entrada segura, banca móvil, scramble pad
* **Patrón Técnico:** Secure Input, Custom Keyboard, Screen Recording Protection
* **Stack Seleccionado:** Custom Widget (SecurePinPad) + Platform Channels (FLAG_SECURE) + Provider (PinEntryProvider)
* **Nivel de Criticidad:** Alto

### 1. Planteamiento del Problema (El "Trigger")

**Escenario de Negocio:**
> *"Como usuario, quiero ingresar mi PIN de 6 dígitos para autorizar transferencias, con la certeza de que ningún malware puede capturarlo."*

El problema: Teclados de terceros (Gboard, SwiftKey) pueden tener permisos para enviar datos a servidores externos. Además, malware con permisos de accesibilidad puede leer inputs. Screen recording captura posición de toques.

**Evidencia de Industria:**

**Caso ai.type Keyboard (2017):** Este teclado popular filtró datos de 31 millones de usuarios, incluyendo textos escritos. Aunque fue removido, demuestra el riesgo de teclados de terceros.

**EventBot Malware (2020):** Troyano bancario que abusaba servicios de accesibilidad Android para capturar credenciales de 200+ apps bancarias, incluyendo PINs ingresados en teclados propietarios mal implementados.

**Estadística:** Según Kaspersky, el 29% del malware bancario móvil usa técnicas de keylogging o screen capture.

**Riesgos:**

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | PIN comprometido = cuenta comprometida. Fraude directo. |
| **Compliance** | PCI-DSS requiere protección de datos de autenticación |
| **Reputacional** | "Malware roba PINs de app de [Banco]" → crisis de PR |

### 2. Matriz de Soluciones y Selección de Herramientas

| Rol | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:----|:-----------------------|:----------------------------------|
| **Junior** | `TextField` con `obscureText: true` y teclado del sistema | **FALLA:** Teclado de terceros puede capturar. Screen recording captura posiciones. Accesibilidad services leen el campo. |
| **Senior** | Custom `PinPad` widget con botones propios, sin usar teclado del sistema | **MEJORA:** Evita teclados de terceros. PERO: posición de botones fija permite inferir PIN por screen recording. No protege contra overlay attacks. |
| **Architect** | **SecurePinPad:** 1) Teclado custom con posiciones aleatorizadas (scramble), 2) FLAG_SECURE para prevenir screenshots/recording, 3) Detección de overlay/accesibilidad maliciosa, 4) Timeout de entrada | **ENTERPRISE:** Resistente a keyloggers, screen capture, overlays. Cumple PCI-DSS. |

### 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Evitar teclados de terceros completamente. Aleatorizar posición de dígitos en cada aparición. Prevenir screenshots y screen recording. Detectar ventanas overlay sospechosas. Timeout de entrada para evitar shoulder surfing extendido. |
| **Restricciones Duras (NO permite)** | **Accesibilidad:** Si usuario tiene accessibility service malicioso con permiso de "observe text", puede leer el PIN aunque usemos custom keyboard. **Root:** En dispositivo rooteado, atacante puede hooker cualquier función. **FLAG_SECURE:** No funciona en screenshots de sistema en algunas ROMs. iOS no tiene equivalente directo. |
| **Criterio de Selección** | Se usa **Provider simple** (`PinEntryProvider`) en lugar de BLoC porque la lógica es mínima (acumular dígitos, validar longitud). No justifica overhead de eventos/estados de BLoC. Custom Widget es necesario porque ningún package existente combina scramble + FLAG_SECURE + overlay detection. |

### 4. Manos a la Obra: Estrategia de Implementación

#### Fase 1: Diseño — Arquitectura del Secure Pin Pad

**Características del SecurePinPad:**
1. Grid de 4x3 con dígitos 0-9, backspace, y confirm
2. Posiciones aleatorizadas en cada montaje del widget
3. FLAG_SECURE activo en la pantalla
4. Detección de overlays antes de mostrar
5. Timeout de 30 segundos de inactividad
6. Máximo 3 intentos antes de bloqueo temporal

**Estructura:**
```
lib/core/ui/secure_input/
├── widgets/
│   ├── secure_pin_pad.dart
│   ├── scrambled_digit_button.dart
│   └── pin_dots_indicator.dart
├── providers/
│   └── pin_entry_provider.dart
├── utils/
│   ├── digit_scrambler.dart
│   └── overlay_detector.dart
└── platform/
    └── secure_flag_handler.dart
```

#### Fase 2: Implementación — Detalles Técnicos

**SecureFlagHandler (Platform Channel):**
- Android: `window.setFlags(WindowManager.LayoutParams.FLAG_SECURE, FLAG_SECURE)`
- Activar en `initState`, desactivar en `dispose`
- iOS: No hay equivalente directo; usar `UITextField.isSecureTextEntry` si es posible, o mostrar warning

**DigitScrambler:**
- Generar permutación aleatoria de [0,1,2,3,4,5,6,7,8,9]
- Usar `Random.secure()` para criptográficamente seguro
- Regenerar en cada build del widget
- Mantener posiciones fijas de backspace y confirm

**OverlayDetector (Android):**
- Verificar `Settings.canDrawOverlays` para otras apps
- Listar ventanas visibles con `AccessibilityService` propio (paradójico pero necesario)
- Si se detecta overlay sospechoso → no mostrar PIN pad, mostrar error

**SecurePinPad Widget:**
- `StatefulWidget` que maneja estado local del PIN
- `GridView.count` con `crossAxisCount: 3`
- Cada `ScrambledDigitButton` recibe el dígito a mostrar según permutación
- Feedback háptico en cada toque (sin feedback visual del dígito)
- Mostrar solo dots (●) para PIN ingresado

**PinEntryProvider:**
- Lista de dígitos ingresados (máx 6)
- Métodos: `addDigit`, `removeLastDigit`, `clear`, `submit`
- Getter `isComplete` cuando length == 6
- Getter `maskedPin` para mostrar en UI

**Timeout y Reintentos:**
- Timer de 30 segundos desde último input
- Si expira → limpiar PIN, mostrar mensaje
- Contador de intentos fallidos persistido en secure storage
- 3 intentos fallidos → bloqueo de 5 minutos

#### Fase 3: Observability — Métricas

**Métricas:**
- `pin.entry.started`
- `pin.entry.completed`
- `pin.entry.timeout`
- `pin.entry.failed` (PIN incorrecto)
- `pin.entry.blocked` (máx intentos)
- `pin.overlay_detected` (ALERTA)
- `pin.entry.duration_ms`

### 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

- [ ] **TAC-5.1:** El PIN DEBE ingresarse mediante teclado custom, NUNCA usando teclado del sistema.
- [ ] **TAC-5.2:** Las posiciones de los dígitos DEBEN aleatorizarse en cada aparición del teclado.
- [ ] **TAC-5.3:** La pantalla de ingreso de PIN DEBE tener FLAG_SECURE activo (Android) para prevenir screenshots.
- [ ] **TAC-5.4:** El sistema DEBE detectar overlays sospechosos y NO mostrar el PIN pad si se detectan.
- [ ] **TAC-5.5:** El PIN ingresado NUNCA debe mostrarse en pantalla, solo representación con dots (●).
- [ ] **TAC-5.6:** DEBE existir timeout de 30 segundos de inactividad que limpie el PIN parcial.
- [ ] **TAC-5.7:** Tras 3 intentos fallidos, DEBE bloquearse el acceso por mínimo 5 minutos.
- [ ] **TAC-5.8:** El PIN NUNCA debe loggearse, ni siquiera en builds de debug.
- [ ] **TAC-5.9:** Cada dígito DEBE tener feedback háptico (vibración) sin feedback visual del valor.
- [ ] **TAC-5.10:** El PIN DEBE transmitirse al backend hasheado con salt, NUNCA en texto plano.

### 6. Estrategia de Pruebas (Shift-Left)

**Stack de Testing:**
- **Unit:** `flutter_test` para scrambler y provider
- **Widget:** `flutter_test` para SecurePinPad
- **Security:** Intentar screenshot/recording, probar con overlay apps

**Escenarios Críticos Obligatorios:**

| # | Escenario | Qué Validar | Tipo |
|:-:|:----------|:------------|:-----|
| 1 | **Screenshot durante entrada de PIN** | En Android con FLAG_SECURE, screenshot debe mostrar pantalla negra o bloquearse. | Security (Manual) |
| 2 | **App con overlay activo** | PIN pad NO debe mostrarse. Mensaje de error sobre "app con overlay detectada". | Integration |
| 3 | **Timeout de entrada** | Tras 30 seg sin input, PIN parcial se limpia, usuario debe reiniciar. | Widget Test |

---

## Caso 6: Session Hijacking en WiFi Público — El Caso del Café que Vació Cuentas

### 0. Metadata para Indexación (AI-Tags)
* **Palabras Clave de Negocio:** session hijacking, WiFi público, red insegura, sidejacking, seguridad de red
* **Patrón Técnico:** Network Security Detection, Session Binding, Additional Verification on Untrusted Networks
* **Stack Seleccionado:** connectivity_plus + Platform Channels (Network Security) + BLoC (NetworkSecurityBloc)
* **Nivel de Criticidad:** Alto

### 1. Planteamiento del Problema (El "Trigger")

**Escenario de Negocio:**
> *"Como usuario, quiero poder revisar mi cuenta bancaria desde el WiFi de la cafetería mientras espero mi café."*

El problema: Redes WiFi públicas son terreno fértil para ataques. ARP spoofing, rogue access points, y SSL stripping permiten a atacantes interceptar tráfico incluso "cifrado".

**Evidencia de Industria:**

**Caso Firesheep (2010):** Herramienta que automatizó session hijacking en redes WiFi públicas, capturando cookies de sesión de Facebook, Twitter, etc. Aunque antiguo, el principio sigue vigente.

**DarkHotel APT:** Grupo de hackers que comprometía redes WiFi de hoteles de lujo para atacar ejecutivos de alto perfil, incluyendo acceso a apps corporativas y bancarias.

**Estudio Kaspersky 2023:** El 25% de los puntos de acceso WiFi públicos no usan ningún cifrado, y otro 25% usa cifrado débil (WEP).

**Riesgos:**

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Session tokens robados permiten account takeover |
| **Reputacional** | "Usuario robado en Starbucks mientras usaba app de [Banco]" |
| **Legal** | Banco puede ser demandado por no advertir sobre riesgos |

### 2. Matriz de Soluciones y Selección de Herramientas

| Rol | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:----|:-----------------------|:----------------------------------|
| **Junior** | Confiar en HTTPS, no hacer nada especial | **FALLA:** SSL stripping, proxies maliciosos con certificados instalados, no protege contra session hijacking si token es robado. |
| **Senior** | Detectar WiFi público y mostrar advertencia al usuario | **MEJORA:** Usuario informado. PERO: muchos ignoran warnings. No agrega protección real. |
| **Architect** | **Defensa en profundidad:** 1) Detectar red insegura, 2) Forzar re-verificación para operaciones sensibles en redes no confiables, 3) Tokens con binding a características de red, 4) Certificate pinning reforzado, 5) VPN sugerida | **ENTERPRISE:** Protección activa, no solo advertencia. Session invalidada si contexto de red cambia drásticamente. |

### 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Detectar tipo de red (WiFi/Cellular/Ethernet). Identificar si WiFi es "segura" (WPA2/3 enterprise) vs "insegura" (abierta, WEP). Requerir step-up authentication en redes no confiables. Vincular session a IP/network fingerprint. |
| **Restricciones Duras (NO permite)** | **Certeza absoluta:** No se puede garantizar que una red "privada" no esté comprometida. **VPN detection:** Detectar si usuario está en VPN es complejo. **iOS limitaciones:** APIs de red más restrictivas que Android. **Usabilidad:** Demasiadas restricciones frustran usuarios legítimos. |
| **Criterio de Selección** | Se usa **BLoC** (`NetworkSecurityBloc`) porque el estado de red puede cambiar en cualquier momento y múltiples partes de la app necesitan reaccionar. BLoC permite broadcast de cambios de estado de red a toda la app. |

### 4. Manos a la Obra: Estrategia de Implementación

#### Fase 1: Diseño — Modelo de Confianza de Red

**Niveles de Confianza:**
- `trusted`: Red celular, WiFi empresarial conocido, VPN activa
- `moderate`: WiFi doméstico con WPA2/3
- `untrusted`: WiFi público, red abierta, WiFi con captive portal
- `hostile`: Red conocida como comprometida (blacklist)

**Matriz de Acciones por Confianza:**

| Operación | trusted | moderate | untrusted | hostile |
|:----------|:--------|:---------|:----------|:--------|
| Ver saldo | ✓ | ✓ | ✓ | ⚠️ Warning |
| Transferir < $100 | ✓ | ✓ | ⚠️ Re-auth | ✗ |
| Transferir > $1000 | ✓ | ⚠️ OTP | ✗ | ✗ |
| Login | ✓ | ✓ | ✓ + Warning | ⚠️ Warning |

#### Fase 2: Implementación — Detalles Técnicos

**NetworkSecurityBloc:**
- Escuchar cambios de conectividad con `connectivity_plus`
- En cada cambio, evaluar nivel de confianza
- Estados: `NetworkInitial`, `NetworkEvaluating`, `NetworkTrusted`, `NetworkModerate`, `NetworkUntrusted`, `NetworkHostile`
- Eventos: `NetworkChanged`, `VPNStatusChanged`, `ForceReevaluate`

**NetworkTrustEvaluator:**
- Verificar tipo de conexión (WiFi, Cellular, Ethernet)
- Si WiFi: obtener SSID, BSSID, security type via Platform Channel
- Android: `WifiManager.getConnectionInfo()`
- iOS: `NEHotspotNetwork.fetchCurrent()`
- Verificar contra whitelist de redes conocidas (configurables)
- Verificar si hay VPN activa

**SessionNetworkBinding:**
- Al crear session, guardar fingerprint de red (IP pública, ISP, país)
- En cada request, verificar que contexto no cambió drásticamente
- Cambio de país en segundos → probable VPN o session hijacking → invalidar

**Step-up Authentication:**
- Cuando `NetworkUntrusted` y operación sensible → requerir biometría + OTP
- Mostrar modal explicando por qué (educación al usuario)
- Opción de "continuar bajo mi riesgo" para operaciones de muy bajo monto

#### Fase 3: Observability — Métricas

**Métricas:**
- `network.trust_level` (distribución)
- `network.untrusted_operations` (operaciones en redes no confiables)
- `network.stepup_required`
- `network.session_invalidated_context_change`

### 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

- [ ] **TAC-6.1:** La app DEBE detectar y clasificar el nivel de confianza de la red actual (trusted/moderate/untrusted/hostile).
- [ ] **TAC-6.2:** En redes clasificadas como `untrusted`, operaciones financieras > $100 DEBEN requerir autenticación adicional.
- [ ] **TAC-6.3:** En redes clasificadas como `hostile`, DEBEN bloquearse todas las operaciones financieras.
- [ ] **TAC-6.4:** El usuario DEBE ser informado cuando está en una red no confiable con mensaje claro y no alarmista.
- [ ] **TAC-6.5:** DEBE existir mecanismo de session binding a contexto de red para detectar cambios sospechosos.
- [ ] **TAC-6.6:** Cambio drástico de contexto de red (ej: cambio de país) DEBE invalidar la sesión.
- [ ] **TAC-6.7:** La app DEBE sugerir uso de VPN cuando detecta red no confiable.
- [ ] **TAC-6.8:** El certificate pinning DEBE estar activo independientemente del tipo de red.
- [ ] **TAC-6.9:** Los usuarios DEBEN poder configurar redes de confianza (whitelist de SSIDs conocidos).
- [ ] **TAC-6.10:** DEBE existir telemetría de operaciones en redes no confiables para análisis de fraude.

### 6. Estrategia de Pruebas (Shift-Left)

**Escenarios Críticos Obligatorios:**

| # | Escenario | Qué Validar | Tipo |
|:-:|:----------|:------------|:-----|
| 1 | **Conectar a WiFi abierto** | App detecta red untrusted, muestra warning, requiere step-up para transferencias. | Integration |
| 2 | **Cambio de red durante sesión** | Session se re-evalúa, si pasa de trusted a untrusted, mostrar notificación. | Integration |
| 3 | **Cambio de IP/país abrupto** | Simular VPN que cambia país. Session debe invalidarse o requerir re-auth. | Integration |

---

## Caso 7: El Dilema del Remember Me — Persistencia Segura de Credenciales en E-commerce

### 0. Metadata para Indexación (AI-Tags)
* **Palabras Clave de Negocio:** remember me, persistencia de sesión, login automático, credenciales guardadas, e-commerce
* **Patrón Técnico:** Secure Credential Storage, Biometric-Gated Access, Device Trust
* **Stack Seleccionado:** flutter_secure_storage + local_auth + Hive (encrypted) + Cubit (RememberMeCubit)
* **Nivel de Criticidad:** Medio

### 1. Planteamiento del Problema (El "Trigger")

**Escenario de Negocio:**
> *"Como usuario de e-commerce, quiero que la app recuerde mi sesión para no tener que ingresar mi contraseña cada vez que quiero comprar algo."*

El balance: Fricción cero (auto-login) vs Seguridad (re-autenticar siempre). E-commerce tolera más riesgo que banca, pero aún maneja datos de pago.

**Evidencia de Industria:**

**Caso Wish.com (2020):** Brecha donde sesiones persistentes mal implementadas permitieron access token reuse desde dispositivos no autorizados.

**Amazon 1-Click:** Amazon balancea conveniencia con seguridad permitiendo compras de 1-click pero requiriendo re-auth para cambios de cuenta o direcciones de envío nuevas.

**Estadística:** Según Baymard Institute, el 17% de abandonos de carrito se deben a proceso de checkout muy largo, incluyendo re-autenticación forzada.

**Riesgos:**

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Compras fraudulentas si dispositivo es robado/comprometido |
| **Usabilidad** | Demasiada fricción = abandono de carrito |
| **Compliance** | PCI-DSS requiere protección de credenciales almacenadas |

### 2. Matriz de Soluciones y Selección de Herramientas

| Rol | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:----|:-----------------------|:----------------------------------|
| **Junior** | Guardar usuario/contraseña en SharedPreferences, auto-login siempre | **FALLA:** Credenciales en texto plano. Cualquier app con acceso a storage las lee. Nunca guardar contraseña real localmente. |
| **Senior** | Guardar refresh token en secure storage, auto-login al abrir app | **MEJORA:** No guarda contraseña. PERO: si alguien tiene el dispositivo desbloqueado, tiene acceso completo a la cuenta. |
| **Architect** | **Biometric-Gated Remember Me:** 1) Guardar refresh token cifrado, 2) Requerir biometría para desbloquear token, 3) Step-up para operaciones sensibles, 4) Device trust scoring | **ENTERPRISE:** Conveniencia de remember-me + protección de biometría. Incluso con dispositivo robado, atacante necesita biometría. |

### 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Auto-login rápido con un toque (biometría). Sesión persistente sin guardar contraseña. Step-up configurable para operaciones de riesgo. Revocación remota de sesiones recordadas. |
| **Restricciones Duras (NO permite)** | **Sin biometría enrollada:** Debe ofrecer alternativa (PIN de app). **Biometría comprometida:** Si el dispositivo tiene biometría de atacante enrollada, protección nula. **Dispositivo compartido:** No recomendado para tablets familiares. |
| **Criterio de Selección** | Se usa **Cubit** porque el estado de remember-me es simple (enabled/disabled, stored/not-stored). Se usa **flutter_secure_storage** para el token y **Hive encrypted** para preferencias menos sensibles (settings de remember-me). |

### 4. Manos a la Obra: Estrategia de Implementación

#### Fase 1: Diseño — Flujo de Remember Me

**Flujo de Activación:**
1. Usuario hace login exitoso
2. Prompt: "¿Guardar sesión para acceso rápido?"
3. Si acepta → guardar refresh token en secure storage, marcado como biometric-protected
4. Próximo launch → mostrar pantalla de biometría
5. Si pasa → usar refresh token para obtener access token
6. Si falla biometría 3 veces → ir a login tradicional

**Flujo de Operaciones Sensibles:**
- Cambiar dirección de envío → re-auth biométrica
- Agregar método de pago → re-auth + OTP
- Compra > $500 → re-auth biométrica
- Cambiar contraseña → re-auth con contraseña actual

#### Fase 2: Implementación — Detalles Técnicos

**RememberMeCubit:**
- Estados: `initial`, `checking`, `enabled`, `disabled`, `authenticating`, `authenticated`, `failed`
- Métodos: `enableRememberMe()`, `disableRememberMe()`, `authenticateWithBiometric()`, `checkRememberMeStatus()`

**SecureTokenStorage:**
- Usar flutter_secure_storage con opciones de acceso biométrico
- iOS: `KeychainAccessibility.whenPasscodeSetThisDeviceOnly` + `authenticationRequired`
- Android: Usar BiometricPrompt como gate antes de acceder a Keystore

**BiometricGate:**
- Wrap de local_auth con UI consistente
- Configuración de fallback: biometría → PIN de app → contraseña
- Tracking de intentos fallidos

**SessionRecovery:**
- En app launch, verificar si hay token guardado
- Si existe, mostrar BiometricGate
- Si biometría exitosa, hacer silent refresh
- Si token expirado, ir a login tradicional

#### Fase 3: Observability — Métricas

**Métricas:**
- `remember_me.enabled_rate` (% de usuarios que activan)
- `remember_me.biometric_success_rate`
- `remember_me.fallback_to_password`
- `remember_me.session_recovered`

### 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

- [ ] **TAC-7.1:** La funcionalidad Remember Me DEBE ser opt-in, nunca activada por defecto.
- [ ] **TAC-7.2:** NUNCA almacenar contraseña localmente, solo refresh tokens.
- [ ] **TAC-7.3:** El acceso al refresh token almacenado DEBE requerir autenticación biométrica o PIN de app.
- [ ] **TAC-7.4:** DEBE existir opción de desactivar Remember Me y limpiar tokens guardados.
- [ ] **TAC-7.5:** Tras 3 intentos fallidos de biometría, DEBE requerir login tradicional.
- [ ] **TAC-7.6:** Operaciones sensibles (pago > $500, cambio de dirección) DEBEN requerir step-up authentication incluso con sesión activa.
- [ ] **TAC-7.7:** DEBE existir opción de "cerrar sesión en todos los dispositivos" que invalide tokens remotamente.
- [ ] **TAC-7.8:** El refresh token guardado DEBE tener TTL máximo de 30 días.
- [ ] **TAC-7.9:** Si el dispositivo detecta cambio de biometría enrollada, DEBE invalidar tokens guardados.
- [ ] **TAC-7.10:** DEBE mostrarse advertencia sobre riesgos de Remember Me en dispositivos compartidos.

### 6. Estrategia de Pruebas (Shift-Left)

**Escenarios Críticos Obligatorios:**

| # | Escenario | Qué Validar | Tipo |
|:-:|:----------|:------------|:-----|
| 1 | **Activar Remember Me → cerrar app → reabrir** | Debe pedir biometría, no password. Si biometría OK, entrar directo. | E2E |
| 2 | **Token expirado con Remember Me activo** | Biometría exitosa pero refresh falla → ir a login tradicional con mensaje claro. | Integration |
| 3 | **Desactivar Remember Me** | Tokens deben limpiarse de secure storage. Siguiente apertura pide login completo. | Integration |

---

## Caso 8: OTP Interceptado — Implementando MFA Resistente a SIM Swapping

### 0. Metadata para Indexación (AI-Tags)
* **Palabras Clave de Negocio:** OTP, SMS, SIM swapping, MFA, autenticación multifactor, TOTP, push notification
* **Patrón Técnico:** TOTP, Push-based MFA, Hardware Tokens
* **Stack Seleccionado:** otp (TOTP) + firebase_messaging (Push MFA) + BLoC (MFABloc)
* **Nivel de Criticidad:** Alto

### 1. Planteamiento del Problema (El "Trigger")

**Escenario de Negocio:**
> *"Como usuario, quiero recibir un código de verificación para aprobar transacciones importantes y proteger mi cuenta."*

El problema con SMS OTP: SIM swapping permite a atacantes tomar control del número telefónico y recibir los SMS destinados a la víctima.

**Evidencia de Industria:**

**Caso Twitter/Jack Dorsey (2019):** El CEO de Twitter fue víctima de SIM swap, permitiendo a atacantes postear desde su cuenta a pesar de tener MFA con SMS.

**Caso Banco de Chile (2021):** Múltiples clientes afectados por SIM swapping, perdiendo acceso a cuentas después de que atacantes convencieran a operadores de telecomunicaciones de transferir sus números.

**Estadística FBI:** Los ataques de SIM swapping aumentaron 400% entre 2018-2021, con pérdidas reportadas de $68 millones solo en 2021.

**NIST SP 800-63B:** Desde 2016, NIST desaconseja SMS como único factor de autenticación para sistemas de alto riesgo.

**Riesgos:**

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Account takeover completo, vaciado de cuentas |
| **Regulatorio** | NIST y reguladores financieros desaconsejan SMS OTP |
| **Técnico** | SMS tiene latencia, puede no llegar, y es fundamentalmente inseguro |

### 2. Matriz de Soluciones y Selección de Herramientas

| Rol | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:----|:-----------------------|:----------------------------------|
| **Junior** | SMS OTP via Twilio/AWS SNS | **FALLA:** Vulnerable a SIM swap, SS7 attacks, y interception. Latencia variable. Costo por mensaje. |
| **Senior** | TOTP (Google Authenticator compatible) como alternativa a SMS | **MEJORA:** No depende de carrier. Offline. PERO: seed debe protegerse, no todos los usuarios quieren otra app. |
| **Architect** | **MFA Híbrido:** 1) Push notification como primario (aprobación con biometría), 2) TOTP como backup, 3) SMS solo para recovery con múltiples verificaciones, 4) Device binding | **ENTERPRISE:** Push es más seguro y mejor UX que OTP. TOTP como fallback no requiere conexión. SMS solo para casos excepcionales con friction adicional. |

### 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Push MFA con aprobación biométrica en el dispositivo registrado. TOTP estándar RFC 6238 compatible con Google/Microsoft Authenticator. Múltiples dispositivos de MFA registrados. Códigos de backup para emergencias. |
| **Restricciones Duras (NO permite)** | **Sin internet:** Push MFA no funciona offline (TOTP sí). **Dispositivo perdido:** Necesita recovery flow robusto. **Usuarios no técnicos:** TOTP puede ser confuso. **Push delivery:** FCM/APNs no garantiza entrega inmediata. |
| **Criterio de Selección** | Se usa **BLoC** (`MFABloc`) porque el flujo de MFA tiene múltiples estados y transiciones complejas (requesting→sent→waiting→verifying→success/failure→retry). Necesitamos auditabilidad de cada paso. |

### 4. Manos a la Obra: Estrategia de Implementación

#### Fase 1: Diseño — Jerarquía de Métodos MFA

**Prioridad de Métodos:**
1. **Push MFA** (mejor UX + seguridad): Notificación push → usuario aprueba con biometría en su dispositivo
2. **TOTP** (fallback offline): Código de 6 dígitos que cambia cada 30 segundos
3. **Email OTP** (fallback si no hay dispositivo): Código enviado al email registrado
4. **SMS OTP** (último recurso): Solo disponible si usuario no tiene otras opciones + requiere verificación adicional

**Flujo de Push MFA:**
1. Usuario inicia operación sensible
2. Backend envía push al dispositivo registrado
3. App muestra prompt de aprobación con detalles de la operación
4. Usuario aprueba con biometría
5. App envía confirmación firmada al backend
6. Backend completa la operación

#### Fase 2: Implementación — Detalles Técnicos

**MFABloc Estados:**
- `MFAInitial`, `MFAMethodsLoading`, `MFAMethodsLoaded`
- `MFAPushSent`, `MFAPushWaiting`, `MFAPushApproved`, `MFAPushDenied`
- `MFATOTPRequested`, `MFATOTPVerifying`
- `MFASuccess`, `MFAFailure`

**PushMFAService:**
- Registrar dispositivo con FCM/APNs token en backend
- Almacenar device_mfa_key en secure storage
- Al recibir push: verificar que viene del backend (firma), mostrar aprobación
- Firmar respuesta con device_mfa_key antes de enviar

**TOTPService:**
- Usar package `otp` para generar códigos RFC 6238
- Seed almacenado en flutter_secure_storage, cifrado
- Setup: generar seed, mostrar QR, usuario escanea con app autenticadora
- Verificación: aceptar código actual ± 1 ventana de tiempo (30 seg tolerancia)

**MFARegistration:**
- Proceso de setup seguro: requiere sesión autenticada + contraseña actual
- Generar códigos de backup (8 códigos de un solo uso)
- Almacenar backup codes hasheados en backend

#### Fase 3: Observability — Métricas

**Métricas:**
- `mfa.method_used` (distribución: push/totp/email/sms)
- `mfa.push.delivery_time_ms`
- `mfa.push.approval_time_ms`
- `mfa.totp.success_rate`
- `mfa.sms.used` (alerta si aumenta)
- `mfa.failure_rate`

### 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

- [ ] **TAC-8.1:** SMS OTP NO DEBE ser el método primario de MFA para operaciones financieras.
- [ ] **TAC-8.2:** DEBE ofrecerse Push MFA como método recomendado con aprobación biométrica.
- [ ] **TAC-8.3:** DEBE ofrecerse TOTP como alternativa que funciona offline.
- [ ] **TAC-8.4:** Si SMS es usado como fallback, DEBE requerir verificación adicional (ej: preguntas de seguridad + documento).
- [ ] **TAC-8.5:** DEBEN generarse códigos de backup durante setup de MFA para recuperación.
- [ ] **TAC-8.6:** El seed de TOTP DEBE almacenarse cifrado en secure storage.
- [ ] **TAC-8.7:** Push MFA DEBE incluir detalles de la operación (monto, destino) para que usuario verifique.
- [ ] **TAC-8.8:** DEBE existir rate limiting: máximo 5 intentos de MFA por hora.
- [ ] **TAC-8.9:** Cambio de método MFA DEBE requerir verificación del método actual + contraseña.
- [ ] **TAC-8.10:** DEBE existir mecanismo de recovery si usuario pierde todos los factores (proceso manual con verificación de identidad).

### 6. Estrategia de Pruebas (Shift-Left)

**Escenarios Críticos Obligatorios:**

| # | Escenario | Qué Validar | Tipo |
|:-:|:----------|:------------|:-----|
| 1 | **Push MFA: aprobar desde otro dispositivo** | Solo el dispositivo registrado puede aprobar. Otro dispositivo del mismo usuario no puede. | Security |
| 2 | **TOTP: código expirado** | Código de hace > 60 segundos debe rechazarse. Código de hace 30 seg debe aceptarse (tolerancia). | Unit |
| 3 | **Todos los métodos fallidos** | Tras agotar intentos de push, totp, y backup codes, cuenta debe bloquearse y requerir recovery manual. | E2E |

---

## Caso 9: PCI-DSS en el Bolsillo — Tokenización de Tarjetas sin Tocar Datos Sensibles

### 0. Metadata para Indexación (AI-Tags)
* **Palabras Clave de Negocio:** PCI-DSS, tokenización, datos de tarjeta, PAN, CVV, procesamiento de pagos, pasarela de pagos
* **Patrón Técnico:** Tokenization, Hosted Fields, PCI Scope Reduction
* **Stack Seleccionado:** Platform Channels (SDKs de Stripe/Adyen) + Provider (PaymentMethodProvider)
* **Nivel de Criticidad:** Alto

### 1. Planteamiento del Problema (El "Trigger")

**Escenario de Negocio:**
> *"Como usuario, quiero guardar mi tarjeta de crédito en la app para compras futuras sin tener que ingresarla cada vez."*

El problema regulatorio: PCI-DSS tiene 12 requisitos con 300+ controles. Si la app toca datos de tarjeta (PAN, CVV, expiry), toda la infraestructura entra en scope de auditoría.

**Evidencia de Industria:**

**Caso Target (2013):** Brecha de 40 millones de tarjetas por almacenar datos de tarjeta en sistemas propios. Costo total: $292 millones. Esto impulsó la adopción masiva de tokenización.

**Caso Heartland Payment Systems (2008):** 130 millones de tarjetas comprometidas. La empresa casi quiebra y tardó años en recuperar certificación PCI.

**Estadística:** Según Verizon PCI Report 2023, solo el 43% de organizaciones mantienen cumplimiento PCI-DSS completo durante todo el año.

**Riesgos:**

| Tipo | Impacto |
|:-----|:--------|
| **Regulatorio** | Multas de $5,000-100,000 por mes de incumplimiento PCI-DSS |
| **Económico** | Costo de auditoría PCI Level 1: $50,000-500,000 anuales |
| **Legal** | Responsabilidad directa si datos de tarjeta son comprometidos |

### 2. Matriz de Soluciones y Selección de Herramientas

| Rol | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:----|:-----------------------|:----------------------------------|
| **Junior** | TextFields para capturar PAN, CVV, Expiry → enviar a backend → guardar en DB | **FALLA CRÍTICA:** App está en full PCI scope. Backend en scope. DB en scope. Auditoría costosa. Multas seguras. |
| **Senior** | Usar SDK de Stripe/Adyen que captura datos y retorna token | **MEJORA:** SDK maneja datos sensibles. App recibe solo token. PERO: implementación ingenua puede filtrar datos en logs o UI. |
| **Architect** | **PCI Scope Reduction completo:** 1) SDK nativo de payment provider via Platform Channels, 2) Nunca loggear nada relacionado con tarjeta, 3) Token almacenado en backend (no local), 4) Re-tokenización periódica, 5) 3DS2 para SCA | **ENTERPRISE:** PCI scope reducido al mínimo (SAQ A-EP o SAQ A). Sin datos sensibles en ningún punto de la infraestructura propia. |

### 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Capturar datos de tarjeta de forma segura sin que pasen por código propio. Obtener token reutilizable para cobros futuros. Mostrar últimos 4 dígitos y marca de tarjeta para identificación. Soportar 3DS2 para autenticación fuerte. |
| **Restricciones Duras (NO permite)** | **Almacenar localmente:** PAN, CVV, expiry date nunca en el dispositivo. **Loggear:** Ningún dato de tarjeta en logs, analytics, crash reports. **Mostrar en UI:** CVV nunca se muestra de vuelta al usuario. **Transmitir a backend propio:** Solo el token, nunca datos crudos. |
| **Criterio de Selección** | Se usa **Platform Channels** con SDKs nativos de Stripe/Adyen porque los SDKs de Flutter de estos providers a veces son wrappers con limitaciones. Los SDKs nativos tienen certificación PCI y son auditados. Se usa **Provider** simple porque el estado de métodos de pago es básico (lista de tokens guardados). |

### 4. Manos a la Obra: Estrategia de Implementación

#### Fase 1: Diseño — Arquitectura de Tokenización

**Flujo de Agregar Tarjeta:**
1. Usuario toca "Agregar tarjeta"
2. App invoca SDK nativo de Stripe/Adyen via Platform Channel
3. SDK presenta su propia UI para capturar datos (o elementos embebidos)
4. SDK envía datos directamente a Stripe/Adyen (nunca pasan por app)
5. Stripe/Adyen retorna token al SDK
6. SDK retorna token a la app via Platform Channel
7. App envía token a backend propio
8. Backend guarda asociación user_id ↔ payment_token

**Flujo de Cobro:**
1. Usuario confirma compra
2. App envía a backend: product_id, quantity, payment_token_id
3. Backend llama a Stripe/Adyen con el token para cobrar
4. Stripe/Adyen procesa y retorna resultado
5. Backend confirma a app

#### Fase 2: Implementación — Detalles Técnicos

**StripeNativeChannel:**
- MethodChannel `com.app/stripe_native`
- Método `createPaymentMethod`: presenta CardField nativo, retorna payment_method_id
- Método `confirmPayment`: para pagos con 3DS2 que requieren autenticación

**PaymentMethodProvider:**
- Lista de tokens guardados (solo metadata: últimos 4 dígitos, marca, alias)
- Métodos: `loadSavedMethods()`, `addPaymentMethod()`, `removePaymentMethod()`, `setDefault()`
- NUNCA almacenar datos sensibles localmente

**Logging Sanitization:**
- Configurar logger para NUNCA capturar campos con nombres: card, pan, cvv, cvc, number, expiry
- Regex para detectar y redactar patrones de tarjeta en cualquier log
- Crash reporting (Sentry/Crashlytics): configurar scrubbing de datos sensibles

**3DS2 Integration:**
- Para SCA (Strong Customer Authentication) en Europa/UK
- SDK maneja el flujo de autenticación 3DS2
- App presenta WebView o native challenge según el SDK

#### Fase 3: Observability — Métricas (Sanitizadas)

**Métricas (sin datos sensibles):**
- `payment.method_added`
- `payment.method_removed`
- `payment.attempt`
- `payment.success`
- `payment.failure` (con código de error genérico, nunca detalles de tarjeta)
- `payment.3ds_challenge_presented`

### 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

- [ ] **TAC-9.1:** Los datos de tarjeta (PAN, CVV, expiry) NUNCA deben pasar por código de la aplicación.
- [ ] **TAC-9.2:** DEBE usarse SDK certificado PCI-DSS del payment provider (Stripe, Adyen, etc.).
- [ ] **TAC-9.3:** El SDK DEBE invocarse via Platform Channels para usar implementación nativa certificada.
- [ ] **TAC-9.4:** El único dato que la app recibe y transmite es el token opaco retornado por el SDK.
- [ ] **TAC-9.5:** NINGÚN dato de tarjeta debe aparecer en logs, analytics, o crash reports.
- [ ] **TAC-9.6:** La UI solo puede mostrar últimos 4 dígitos y marca de la tarjeta, nunca PAN completo.
- [ ] **TAC-9.7:** Los tokens de pago DEBEN almacenarse en backend, NUNCA localmente en el dispositivo.
- [ ] **TAC-9.8:** DEBE implementarse 3DS2 para transacciones que lo requieran (SCA compliance).
- [ ] **TAC-9.9:** DEBE existir proceso de eliminación de método de pago que invalide el token.
- [ ] **TAC-9.10:** Auditoría de seguridad debe confirmar que la app califica para SAQ A-EP o superior.

### 6. Estrategia de Pruebas (Shift-Left)

**Escenarios Críticos Obligatorios:**

| # | Escenario | Qué Validar | Tipo |
|:-:|:----------|:------------|:-----|
| 1 | **Verificar que PAN nunca aparece en logs** | Activar modo debug, agregar tarjeta, buscar en todos los logs. NO debe aparecer ningún número de tarjeta. | Security |
| 2 | **Flujo 3DS2 completo** | Usar tarjeta de prueba que requiere 3DS. Completar challenge. Pago debe procesarse exitosamente. | E2E |
| 3 | **Eliminar método de pago** | Eliminar tarjeta guardada. Intentar cobrar con ese token desde backend. Debe fallar. | Integration |

---

## Caso 10: El Empleado Deshonesto — Auditoría y Trazabilidad de Acciones en Apps Internas

### 0. Metadata para Indexación (AI-Tags)
* **Palabras Clave de Negocio:** auditoría, trazabilidad, log de acciones, compliance, insider threat, app interna, backoffice
* **Patrón Técnico:** Audit Trail, Event Sourcing, Immutable Logs, Chain of Custody
* **Stack Seleccionado:** Dio Interceptors + drift (SQLite) para log local + BLoC (AuditBloc)
* **Nivel de Criticidad:** Alto

### 1. Planteamiento del Problema (El "Trigger")

**Escenario de Negocio:**
> *"Como oficial de cumplimiento, necesito saber exactamente qué empleado realizó qué acción, cuándo, y desde qué dispositivo, para auditorías regulatorias y detección de fraude interno."*

El problema: Las apps internas (backoffice, call center, cajeros) manejan operaciones sensibles. Un empleado malicioso puede abusar accesos. Sin audit trail, es imposible investigar incidentes.

**Evidencia de Industria:**

**Caso Wells Fargo (2016):** Empleados crearon 3.5 millones de cuentas fraudulentas sin autorización de clientes. La falta de auditoría granular permitió que esto continuara por años.

**Caso Société Générale (2008):** Un trader causó pérdidas de €4.9 billones con operaciones no autorizadas. Los controles de auditoría existentes no detectaron las anomalías a tiempo.

**Estadística ACFE:** El 5% del revenue de las organizaciones se pierde por fraude. El 42% de los fraudes ocupacionales son cometidos por empleados.

**Riesgos:**

| Tipo | Impacto |
|:-----|:--------|
| **Regulatorio** | Reguladores bancarios (CNBV, SBS, Banco Central) requieren audit trails |
| **Legal** | Sin evidencia, imposible probar/refutar acusaciones |
| **Operacional** | Fraude interno no detectado puede continuar indefinidamente |

### 2. Matriz de Soluciones y Selección de Herramientas

| Rol | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:----|:-----------------------|:----------------------------------|
| **Junior** | Logs en servidor con timestamp y user_id básicos | **FALLA:** Logs modificables, sin contexto de dispositivo, sin hash de integridad. Empleado con acceso a DB puede borrar evidencia. |
| **Senior** | Logs estructurados con más contexto + envío a sistema centralizado (ELK) | **MEJORA:** Centralización dificulta tampering. PERO: sin firma criptográfica, administrador de ELK puede modificar. Sin log local si hay problemas de red. |
| **Architect** | **Audit Trail Inmutable:** 1) Log local firmado con hash encadenado (mini blockchain), 2) Sync a servidor con verificación de integridad, 3) Almacenamiento write-once (WORM), 4) Contexto completo (acción, usuario, dispositivo, timestamp, hash anterior), 5) BLoC para tracking de eventos | **ENTERPRISE:** Tamper-evident. Evidencia admisible legalmente. Detección de manipulación. Cumple SOX, GDPR, regulaciones bancarias. |

### 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Registrar cada acción con contexto completo. Hash encadenado para detectar manipulación. Sincronización eventual con servidor central. Consulta local para revisiones rápidas. Exportación para auditorías externas. |
| **Restricciones Duras (NO permite)** | **Borrado:** Logs no pueden borrarse, solo archivarse con retención configurable. **Modificación:** Cualquier cambio rompe la cadena de hashes e invalida el log. **Dispositivo offline extendido:** Si no sincroniza en X días, forzar sync o bloquear operaciones. |
| **Criterio de Selección** | Se usa **drift** (SQLite) para log local porque: necesitamos queries para búsqueda local, integridad transaccional, y es más robusto que Hive para este caso. Se usa **BLoC** (`AuditBloc`) porque cada evento de la app que sea auditable debe pasar por un punto central que garantice el registro. |

### 4. Manos a la Obra: Estrategia de Implementación

#### Fase 1: Diseño — Estructura del Audit Event

**Campos del Audit Event:**
- `event_id`: UUID único
- `timestamp`: ISO 8601 con timezone
- `user_id`: ID del empleado
- `session_id`: Sesión activa
- `device_id`: Fingerprint del dispositivo
- `action_type`: Enum (VIEW, CREATE, UPDATE, DELETE, APPROVE, REJECT, LOGIN, LOGOUT, etc.)
- `resource_type`: Qué tipo de recurso (ACCOUNT, TRANSACTION, USER, CONFIG, etc.)
- `resource_id`: ID específico del recurso afectado
- `action_details`: JSON con detalles específicos de la acción
- `ip_address`: IP desde donde se realizó
- `location`: Coordenadas si disponible
- `previous_hash`: Hash del evento anterior (chain)
- `event_hash`: SHA-256 de todo lo anterior

#### Fase 2: Implementación — Detalles Técnicos

**AuditBloc:**
- Cada feature de la app emite eventos de auditoría via `AuditBloc`
- Evento: `AuditEventRequested(actionType, resourceType, resourceId, details)`
- BLoC: 1) Enriquecer con contexto (user, device, timestamp), 2) Calcular hash, 3) Guardar localmente, 4) Encolar para sync

**LocalAuditRepository (drift):**
- Tabla `audit_events` con todos los campos
- Insert-only: no UPDATE, no DELETE
- Índices en timestamp, user_id, action_type para queries
- Método `verifyChain()`: recalcular hashes y verificar integridad

**AuditSyncService:**
- Background job que sincroniza eventos pendientes cada X minutos
- Envía batch de eventos a backend
- Backend verifica hashes y almacena
- Mark as synced localmente solo si backend confirma

**TamperDetection:**
- Al iniciar app: ejecutar `verifyChain()` sobre log local
- Si detecta inconsistencia: alertar al servidor, bloquear operaciones sensibles, registrar incidente

**AuditInterceptor (Dio):**
- Para ciertas APIs (modificación de datos), generar audit event automáticamente
- Capturar request/response metadata

#### Fase 3: Observability — Métricas

**Métricas:**
- `audit.events_logged` (rate)
- `audit.events_synced`
- `audit.sync_lag_ms`
- `audit.chain_verification_failures` (CRÍTICO)
- `audit.storage_size_mb`

### 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

- [ ] **TAC-10.1:** TODA acción de negocio (CRUD en recursos sensibles) DEBE generar un evento de auditoría.
- [ ] **TAC-10.2:** Cada evento DEBE incluir: timestamp, user_id, device_id, action_type, resource_type, resource_id, details.
- [ ] **TAC-10.3:** Los eventos DEBEN encadenarse con hash criptográfico para detectar tampering.
- [ ] **TAC-10.4:** El log local DEBE ser insert-only, SIN capacidad de UPDATE o DELETE.
- [ ] **TAC-10.5:** DEBE existir sincronización con backend central que verifique integridad de la cadena.
- [ ] **TAC-10.6:** Si se detecta manipulación del log local, la app DEBE alertar al backend y bloquear operaciones.
- [ ] **TAC-10.7:** Los logs DEBEN poder exportarse en formato estándar para auditorías externas.
- [ ] **TAC-10.8:** La retención mínima de logs DEBE ser 7 años (regulación bancaria típica).
- [ ] **TAC-10.9:** El backend de auditoría DEBE usar almacenamiento WORM (Write Once Read Many).
- [ ] **TAC-10.10:** DEBE existir capacidad de búsqueda por user_id, fecha, action_type para investigaciones.

### 6. Estrategia de Pruebas (Shift-Left)

**Escenarios Críticos Obligatorios:**

| # | Escenario | Qué Validar | Tipo |
|:-:|:----------|:------------|:-----|
| 1 | **Modificar DB de audit local manualmente** | App debe detectar chain rota al iniciar. Debe alertar y bloquear operaciones sensibles. | Security |
| 2 | **Operación offline → sync** | Realizar acciones sin red. Reconectar. Eventos deben sincronizarse con hashes correctos. | Integration |
| 3 | **Query de auditoría por investigación** | Dado un user_id y rango de fechas, obtener todas las acciones. Resultado debe ser completo y verificable. | Integration |

---

# FIN DEL CAPÍTULO 1

---

*Los capítulos 2-10 continúan con la misma estructura detallada...*
