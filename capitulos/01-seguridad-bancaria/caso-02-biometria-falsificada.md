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

**Caso Deepfake Banking Fraud (2023):** Un grupo criminal en Hong Kong usó deepfakes para engañar sistemas de verificación facial en múltiples bancos, obteniendo préstamos por más de $25 millones. Los atacantes usaron fotos de redes sociales para generar videos sintéticos que pasaron verificación de "selfie".

**Ataque de Presentation Attack (Spoofing):** En 2021, investigadores de seguridad demostraron que el 20% de los sistemas de reconocimiento facial en apps bancarias europeas podían ser engañados con una foto impresa de alta resolución.

**Regulación PSD2/SCA:** La directiva PSD2 requiere Strong Customer Authentication (SCA) para transacciones que superen €30. La autenticación biométrica DEBE incluir protección contra ataques de presentación según estándares EBA.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Pérdidas por fraude en préstamos ($50K-500K por incidente), chargebacks, provisiones por cartera vencida |
| **Regulatorio** | Incumplimiento de KYC/AML puede resultar en multas de $1M-100M |
| **Reputacional** | Un caso viral de "deepfake loan fraud" destruye confianza en canal digital |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Rol | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:----|:-----------------------|:----------------------------------|
| **Junior** | `local_auth` para Face ID/Touch ID, si pasa → aprobado | **FALLA CRÍTICA:** Solo verifica biometría del dispositivo. No verifica que sea el titular. No detecta fotos/videos. Cualquier persona con acceso al dispositivo puede aprobar. |
| **Senior** | `local_auth` como primer factor + captura de selfie + envío a backend para comparación con foto de documento | **MEJORA PARCIAL:** Agrega verificación de identidad. Pero: vulnerable a fotos impresas, latencia alta, dependencia de conectividad. |
| **Architect** | **Autenticación en capas:** 1) `local_auth` gate inicial, 2) SDK certificado Liveness (Facetec/iProov) via Platform Channels, 3) Comparación 1:1 contra KYC, 4) Challenge-Response, 5) Cubit para orquestar | **ENTERPRISE:** Detecta spoofing 2D/3D. Certificación iBeta/NIST. Evidencia auditable con firma criptográfica. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Detección de ataques 2D (fotos, pantallas) y 3D (máscaras). Verificación de liveness con challenge-response. Comparación 1:1 selfie vs documento. FaceMap encriptado para auditoría. Funciona con cámara frontal estándar. Cumple ISO 30107-3 y NIST FRVT. |
| **Restricciones Duras (NO permite)** | **local_auth SOLO:** No distingue entre titular y otra persona del dispositivo. **Sin red:** Algunos SDKs requieren validación server-side. **Condiciones extremas:** Liveness falla con luz < 50 lux, lentes muy oscuros, barbas > 50% rostro. **Simulador:** Biometría no disponible en simulador. |
| **Criterio de Selección** | Se usa **Cubit** en lugar de BLoC porque el flujo es secuencial sin eventos complejos (captura→análisis→resultado). Cubit reduce boilerplate. Se usan **Platform Channels** para SDK nativo porque Facetec/iProov solo existen en Kotlin/Swift, procesamiento de imagen más eficiente nativo. |

---

## 4. Manos a la Obra: Estrategia de Implementación

### Fase 1: Diseño — Arquitectura de Autenticación en Capas

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

### Fase 2: Implementación — Detalles Técnicos

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

### Fase 3: Observability — Métricas y KPIs

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

---

## 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

```
[ ] TAC-2.1: Para operaciones de alto riesgo (préstamos, transferencias > $5,000),
    local_auth NO ES SUFICIENTE. Se requiere liveness adicional.

[ ] TAC-2.2: El SDK de Liveness DEBE tener certificación ISO 30107-3 Level 2 o superior.

[ ] TAC-2.3: El sistema DEBE detectar y rechazar ataques de presentación 2D con tasa > 99%.

[ ] TAC-2.4: El score de face match contra documento KYC DEBE ser ≥ 0.85.

[ ] TAC-2.5: Cada verificación exitosa DEBE generar audit token firmado criptográficamente.

[ ] TAC-2.6: El tiempo total del flujo NO DEBE exceder 20 segundos en p95.

[ ] TAC-2.7: Ante spoofing detectado: bloquear operación, generar alerta,
    NO revelar detección al usuario.

[ ] TAC-2.8: El flujo DEBE funcionar en modo degradado (solo local_auth) para
    operaciones de bajo riesgo sin conectividad.

[ ] TAC-2.9: DEBE existir fallback a verificación humana (videollamada) tras
    3 intentos fallidos.

[ ] TAC-2.10: Los datos biométricos (faceMap) NUNCA se almacenan localmente sin
    cifrado, TTL < 5 minutos.
```

---

## 6. Estrategia de Pruebas (Shift-Left)

### Stack de Testing
- **Unit:** `flutter_test`, `mocktail` para Cubit
- **Integration:** `integration_test` + dispositivo físico
- **E2E:** `maestro` para flujos visuales
- **Security:** Tests de penetración con fotos/videos (lab especializado)

### Escenarios Críticos Obligatorios

| # | Escenario | Qué Validar | Tipo |
|:-:|:----------|:------------|:-----|
| 1 | **Ataque de foto impresa** | Sistema DEBE detectar spoofing y rechazar con `spoofType: "2d_photo"`. NO dar pistas al atacante. | Security (Manual) |
| 2 | **Usuario sin biometría enrollada** | Fallo graceful en paso 1, mensaje claro, ofrecer alternativa (PIN + OTP). No crashear. | Integration |
| 3 | **Timeout de red en face match** | Liveness local OK → timeout en match → reintentar 2x con backoff, luego "intentar más tarde" SIN perder progreso. | Integration |

---

## Referencias

- [ISO/IEC 30107-3:2017 - Biometric presentation attack detection](https://www.iso.org/standard/67381.html)
- [NIST FRVT - Face Recognition Vendor Test](https://www.nist.gov/programs-projects/face-recognition-vendor-test-frvt)
- [iBeta Quality Assurance - PAD Testing](https://www.ibeta.com/pad-testing/)
- [EBA Guidelines on Strong Customer Authentication](https://www.eba.europa.eu/regulation-and-policy/payment-services-and-electronic-money)

---

*Anterior: [Token que Nunca Expira](caso-01-token-nunca-expira.md) | Siguiente: [Certificate Pinning](caso-03-mitm-certificate-pinning.md)*
