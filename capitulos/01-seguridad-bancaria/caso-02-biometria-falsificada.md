# Caso 2: Biometría Falsificada
## Cuando Face ID No Es Suficiente para Aprobar un Crédito

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | autenticación biométrica, Face ID, Touch ID, aprobación de crédito, fraude de identidad, liveness detection |
| **Patrón Técnico** | Multi-Layer Biometric Authentication, Step-up Authentication, Liveness Detection |
| **Stack Seleccionado** | local_auth + platform channels para SDK nativo (Facetec/iProov) + Cubit (BiometricCubit) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero aprobar mi solicitud de crédito de $50,000 usando Face ID para no tener que ir a una sucursal."*

El problema: Face ID/Touch ID del sistema operativo solo verifica que "un rostro/huella registrado en el dispositivo" coincide. NO verifica que sea el rostro del titular de la cuenta bancaria, ni que sea una persona viva frente a la cámara.

### Evidencia de Industria

**Caso Deepfake Banking Fraud (2023):** Un grupo criminal en Hong Kong usó deepfakes para engañar sistemas de verificación facial en múltiples bancos, obteniendo préstamos por más de $25 millones.

**Ataque de Presentation Attack (Spoofing):** En 2021, investigadores demostraron que el 20% de los sistemas de reconocimiento facial en apps bancarias europeas podían ser engañados con una foto impresa.

**Regulación PSD2/SCA:** La directiva requiere Strong Customer Authentication para transacciones > €30. La biometría DEBE incluir protección anti-spoofing según estándares EBA.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Pérdidas por fraude en préstamos ($50K-500K por incidente) |
| **Regulatorio** | Incumplimiento KYC/AML: multas de $1M-100M |
| **Reputacional** | Un caso viral de deepfake fraud destruye confianza digital |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | `local_auth` para Face ID/Touch ID, si pasa → aprobado | **INADECUADO:** Solo verifica biometría del dispositivo. No verifica que sea el titular. No detecta fotos/videos. Cualquier persona con acceso al dispositivo puede aprobar. |
| **ACEPTABLE** | `local_auth` como primer factor + captura de selfie + envío a backend para comparación con foto de documento | **CUMPLE MÍNIMOS:** Agrega verificación de identidad. Pero: vulnerable a fotos impresas, latencia alta, dependencia de conectividad. |
| **ENTERPRISE** | **Autenticación en capas:** 1) `local_auth` gate inicial, 2) SDK certificado Liveness (Facetec/iProov) via Platform Channels, 3) Comparación 1:1 contra KYC, 4) Challenge-Response, 5) Cubit para orquestar | **ÓPTIMO PARA BANCA:** Detecta spoofing 2D/3D. Certificación iBeta/NIST. Evidencia auditable con firma criptográfica. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Detección de ataques 2D (fotos, pantallas) y 3D (máscaras). Verificación de liveness con challenge-response. Comparación 1:1 selfie vs documento. FaceMap encriptado para auditoría. Funciona con cámara frontal estándar. Cumple ISO 30107-3 y NIST FRVT. |
| **Restricciones Duras (NO permite)** | **local_auth SOLO:** No distingue entre titular y otra persona del dispositivo. **Sin red:** Algunos SDKs requieren validación server-side. **Condiciones extremas:** Liveness falla con luz < 50 lux, lentes muy oscuros, barbas > 50% rostro. **Simulador:** Biometría no disponible. |
| **Criterio de Selección** | Se usa **Cubit** en lugar de BLoC porque el flujo es secuencial sin eventos complejos (captura→análisis→resultado). Cubit reduce boilerplate. Se usan **Platform Channels** para SDK nativo porque Facetec/iProov solo existen en Kotlin/Swift, procesamiento de imagen más eficiente nativo. |

---

## 4. Manos a la Obra: Estrategia de Implementación

### Justificación del Plan

La estrategia de implementación se deriva del análisis del caso Deepfake Banking Fraud y los requisitos de PSD2/SCA:

1. **Verificación del dispositivo ≠ verificación de identidad** → Necesitamos capa adicional de Liveness Detection
2. **Ataques de presentación evolucionan** → Requerimos SDK con certificación ISO 30107-3
3. **Evidencia para auditoría y disputas** → Implementamos captura de audit trail firmado
4. **Balance UX vs Seguridad** → Flujo escalonado según nivel de riesgo de la operación

La arquitectura usa Cubit por simplicidad del flujo secuencial, con Platform Channels para aprovechar SDKs nativos certificados.

---

### Fase 1: Diseño — Arquitectura de Autenticación en Capas

**Flujo de Autenticación Alto Riesgo:**
```
local_auth → Liveness SDK → Face Match API → Challenge (opcional) → Aprobación
   (1 seg)     (5-10 seg)      (2-3 seg)        (5 seg)
```

**Estructura de Carpetas:**
```
lib/features/biometric_auth/
├── data/
│   └── datasources/
│       ├── biometric_local_datasource.dart
│       ├── liveness_native_datasource.dart
│       └── face_match_remote_datasource.dart
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

---

### Fase 2: Implementación por Plataforma

#### 2.1 Flutter (Cross-Platform) — Capa de Aplicación

**BiometricAuthState:**
- Enum de pasos: `initial`, `checkingCapability`, `requestingBiometric`, `capturingLiveness`, `processingLiveness`, `matchingFace`, `waitingChallenge`, `success`, `failure`
- Campos: `errorMessage`, `errorType`, `livenessConfidence`, `faceMatchScore`, `auditToken`

**BiometricAuthCubit - Flujo Principal:**
1. `checkDeviceCapability`: Verificar `canCheckBiometrics`, `isDeviceSupported`
2. `performDeviceBiometric`: `local_auth.authenticate` con `biometricOnly: true`
3. `performLivenessCheck`: Invocar Platform Channel al SDK nativo
4. `performFaceMatch`: Enviar `faceMap` a API, umbral ≥ 0.85
5. `performChallenge`: Solo para montos > $10,000

**LivenessNativeDataSource (Platform Channel):**
- MethodChannel: `com.bank.app/liveness`
- Parámetros: timeout 30s, minConfidence 0.9, enableAudit true
- Retorna: `isAlive`, `confidence`, `faceMap`, `auditImage`, `wasSpoofAttempt`

---

#### 2.2 Android — Integración SDK Nativo

**Configuración Gradle:**
- Agregar repositorio Maven del SDK (Facetec/iProov)
- Configurar ProGuard rules para el SDK
- Permisos: `CAMERA`, `INTERNET`

**Platform Channel Handler (Kotlin):**
- Implementar `MethodChannel` handler en `MainActivity`
- Inicializar SDK con license key
- Manejar lifecycle de cámara
- Retornar resultados via `result.success()`

**Consideraciones Android:**
- Camera2 API para mejor rendimiento
- Verificar que no hay overlay de otras apps (seguridad)
- Manejar rotación de pantalla durante captura

---

#### 2.3 iOS — Integración SDK Nativo

**Configuración Podfile:**
- Agregar pod del SDK de liveness
- Configurar build settings para frameworks

**Platform Channel Handler (Swift):**
- Implementar handler en `AppDelegate`
- Configurar `AVCaptureSession` para cámara
- Manejar permisos de cámara con `NSCameraUsageDescription`

**Consideraciones iOS:**
- Info.plist: `NSCameraUsageDescription` requerido
- App Transport Security: permitir dominio del SDK
- Funciona con TrueDepth camera (Face ID devices) y cámara estándar

---

### Fase 3: Observability — Métricas y KPIs

**Métricas Críticas:**
- Funnel: `auth.started` → `device.passed` → `liveness.passed` → `match.passed` → `completed`
- Seguridad: `spoofing_detected`, `face_not_matched`
- Performance: `liveness_latency_ms`, `total_auth_time_ms`

**KPIs de Éxito:**

| Métrica | Objetivo | Alerta |
|:--------|:---------|:-------|
| Tasa de completación | > 85% | < 70% → Investigar UX |
| Spoofing detection rate | > 99% | < 95% → Crítico |
| Tiempo total auth | < 15 seg | > 25 seg → Optimizar |

---

## 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

### TACs Flutter (Cross-Platform)

```
[ ] TAC-2.1-FLUTTER: Para operaciones de alto riesgo (préstamos, transferencias > $5,000),
    local_auth NO ES SUFICIENTE. Se requiere liveness adicional.

[ ] TAC-2.2-FLUTTER: El BiometricAuthCubit DEBE orquestar el flujo secuencial:
    device check → local_auth → liveness → face match.

[ ] TAC-2.3-FLUTTER: El score de face match contra documento KYC DEBE ser ≥ 0.85.

[ ] TAC-2.4-FLUTTER: Cada verificación exitosa DEBE generar audit token para evidencia.

[ ] TAC-2.5-FLUTTER: El tiempo total del flujo NO DEBE exceder 20 segundos en p95.

[ ] TAC-2.6-FLUTTER: Ante spoofing detectado: bloquear operación, generar alerta,
    NO revelar detección al usuario (no educar al atacante).
```

### TACs Android

```
[ ] TAC-2.7-ANDROID: El SDK de Liveness DEBE tener certificación ISO 30107-3 Level 2+.

[ ] TAC-2.8-ANDROID: DEBE verificarse que no hay overlay de otras apps durante captura
    (WindowManager.LayoutParams check).

[ ] TAC-2.9-ANDROID: La cámara DEBE usar Camera2 API para mejor rendimiento.

[ ] TAC-2.10-ANDROID: ProGuard rules DEBEN configurarse para no ofuscar clases del SDK.
```

### TACs iOS

```
[ ] TAC-2.11-IOS: Info.plist DEBE incluir NSCameraUsageDescription con texto claro.

[ ] TAC-2.12-IOS: El SDK DEBE funcionar tanto en dispositivos con TrueDepth camera
    como en dispositivos con cámara frontal estándar.

[ ] TAC-2.13-IOS: DEBE manejarse correctamente el permiso de cámara denegado
    con fallback a verificación alternativa.

[ ] TAC-2.14-IOS: App Transport Security DEBE permitir dominios del SDK de liveness.
```

### TACs Backend (Referencia)

```
[ ] TAC-2.15-BACKEND: El endpoint de face match DEBE aceptar faceMap encriptado
    y retornar score de similitud + audit token firmado.

[ ] TAC-2.16-BACKEND: DEBE almacenarse evidencia de cada verificación por 7 años
    para auditorías y disputas.

[ ] TAC-2.17-BACKEND: Si se detecta spoofing, DEBE generarse alerta a equipo de fraude
    en < 5 minutos.
```

---

## 6. Estrategia de Pruebas (Shift-Left)

### Stack de Testing
- **Unit:** `flutter_test`, `mocktail` para Cubit
- **Integration:** `integration_test` + dispositivo físico (biometría no funciona en simulador)
- **E2E:** `maestro` para flujos visuales
- **Security:** Tests de penetración (lab especializado)

### Escenarios Críticos Obligatorios

| # | Escenario | Qué Validar | Plataforma | Tipo |
|:-:|:----------|:------------|:-----------|:-----|
| 1 | **Ataque de foto impresa** | Sistema DEBE detectar spoofing y rechazar. NO dar pistas al atacante. | Android + iOS | Security (Manual) |
| 2 | **Usuario sin biometría enrollada** | Fallo graceful, mensaje claro, ofrecer alternativa (PIN + OTP). | Flutter | Integration |
| 3 | **Timeout de red en face match** | Liveness OK → timeout → reintentar 2x, luego "intentar más tarde". | Flutter | Integration |
| 4 | **Permiso de cámara denegado** | Mostrar explicación clara, botón para ir a Settings. | Android + iOS | Integration |
| 5 | **Luz insuficiente** | SDK debe indicar al usuario que mejore iluminación. | Android + iOS | UX (Manual) |

---

## Referencias

- [ISO/IEC 30107-3:2017 - Biometric presentation attack detection](https://www.iso.org/standard/67381.html)
- [NIST FRVT - Face Recognition Vendor Test](https://www.nist.gov/programs-projects/face-recognition-vendor-test-frvt)
- [iBeta Quality Assurance - PAD Testing](https://www.ibeta.com/pad-testing/)
- [Apple Local Authentication Framework](https://developer.apple.com/documentation/localauthentication)
- [Android BiometricPrompt](https://developer.android.com/reference/android/hardware/biometrics/BiometricPrompt)

---

*Anterior: [Token que Nunca Expira](caso-01-token-nunca-expira.md) | Siguiente: [Certificate Pinning](caso-03-mitm-certificate-pinning.md)*
