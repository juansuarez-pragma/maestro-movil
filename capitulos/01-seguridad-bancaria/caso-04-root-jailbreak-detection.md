# Caso 4: Rooted pero Confiable
## Detectar Dispositivos Comprometidos sin Bloquear Usuarios Legítimos

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | root detection, jailbreak detection, dispositivo comprometido, fraude, seguridad móvil, SafetyNet, Play Integrity |
| **Patrón Técnico** | Device Attestation, Risk-Based Authentication, Layered Security |
| **Stack Seleccionado** | flutter_jailbreak_detection + Platform Channels (Play Integrity/App Attest) + Riverpod (DeviceSecurityProvider) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como oficial de seguridad del banco, quiero detectar dispositivos rooteados/jailbroken para evaluar el riesgo, pero sin bloquear usuarios legítimos que usan custom ROMs por privacidad."*

El dilema: Bloquear 100% de dispositivos rooteados genera falsos positivos (desarrolladores, usuarios privacy-conscious). No detectarlos abre la puerta a malware y hooking.

### Evidencia de Industria

**Caso Cerberus Banking Trojan (2020-2023):** Malware Android que requiere root. Intercepta SMS OTP, captura credenciales via overlay attacks, extrae datos de apps bancarias. Ha afectado bancos en España, México, Brasil.

**Magisk "MagiskHide" (ahora Zygisk):** Permite ocultar root de apps específicas. Usuarios legítimos la usan, pero también atacantes para evadir detección.

**Estudio Promon (2022):** El 50% de las apps bancarias top 100 pueden tener protecciones bypaseadas en dispositivos rooteados con Frida/Xposed.

**Estadística clave:** Solo ~2% de dispositivos están rooteados, pero representan ~30% de intentos de fraude en apps bancarias.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Fraude vía malware en dispositivos rooteados, ATO |
| **False Positives** | Bloquear usuarios legítimos genera churn |
| **Regulatorio** | Algunos reguladores exigen detección de dispositivos comprometidos |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | `flutter_jailbreak_detection` → si detecta root → bloquear app | **FALLA:** Falsos positivos, fácilmente bypasseable con Magisk Hide, no distingue riesgo real, UX hostil. |
| **ACEPTABLE** | Múltiples checks (su binary, build tags) + warning pero permitir con funcionalidad reducida | **MEJORA:** Reduce falsos positivos. PERO: aún bypasseable, sin verificación server-side, atacante puede parchear binario. |
| **ENTERPRISE** | **Device Attestation multi-capa:** 1) Checks locales, 2) Play Integrity/SafetyNet (Android) y DeviceCheck/App Attest (iOS) server-side, 3) Risk scoring dinámico, 4) Respuesta adaptativa | **ÓPTIMO PARA BANCA:** Verificación criptográfica server-side. Difícil bypassear. Políticas granulares por operación. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Detectar root/jailbreak con múltiples técnicas. Verificación criptográfica server-side (Play Integrity). Risk scoring continuo. Respuestas granulares (permitir lectura, bloquear escritura). Detectar hooking frameworks (Frida, Xposed). |
| **Restricciones Duras (NO permite)** | **100% infalible:** Siempre existirán bypasses con suficiente esfuerzo. **Offline:** Play Integrity requiere conexión. **Emuladores:** Detectados como riesgo, afecta devs. **Play Services:** Play Integrity requiere Google Play Services. |
| **Criterio de Selección** | Se usa **Riverpod** con `DeviceSecurityProvider` porque: estado de seguridad se consulta desde múltiples features, Riverpod permite dependency injection limpio, fácil testing con overrides. Play Integrity sobre SafetyNet porque SafetyNet está deprecated. |

---

## 4. Manos a la Obra: Estrategia de Implementación

### Justificación del Plan

La estrategia de implementación se deriva del análisis de Cerberus Trojan y las estadísticas de fraude en dispositivos rooteados:

1. **Checks locales son bypasseables** → Necesitamos verificación criptográfica server-side
2. **No todos los rooteados son maliciosos** → Implementamos risk scoring adaptativo
3. **Atacantes usan Frida/Xposed** → Detección activa de hooking frameworks
4. **Balance UX vs Seguridad** → Respuestas granulares por nivel de riesgo y tipo de operación

La arquitectura usa Riverpod para compartir estado de seguridad entre features, con Platform Channels para APIs nativas de attestation.

---

### Fase 1: Diseño — Modelo de Riesgo Adaptativo

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

**Estructura de Carpetas:**
```
lib/core/security/
├── data/
│   ├── datasources/
│   │   ├── local_root_detection_datasource.dart
│   │   ├── play_integrity_datasource.dart
│   │   └── ios_attestation_datasource.dart
│   └── repositories/
│       └── device_security_repository_impl.dart
├── domain/
│   ├── entities/
│   │   └── device_security_status.dart
│   └── usecases/
│       ├── evaluate_device_security.dart
│       └── check_operation_permission.dart
└── presentation/
    └── providers/
        └── device_security_provider.dart
```

---

### Fase 2: Implementación por Plataforma

#### 2.1 Flutter (Cross-Platform) — Capa de Aplicación

**DeviceSecurityStatus Entity:**
- `riskLevel`: Enum (secure, elevated, high, critical)
- `rootDetected`: Boolean
- `hookingDetected`: Boolean
- `attestationPassed`: Boolean
- `lastEvaluatedAt`: DateTime

**DeviceSecurityProvider (Riverpod):**
- `AsyncNotifierProvider` para estado de seguridad
- Evaluar al inicio de app y periódicamente (cada 5-15 minutos)
- Cachear resultado con TTL
- Exponer `riskLevel` y `canPerformOperation(operation)`

**LocalRootDetection (Checks Cross-Platform):**
- Usar `flutter_jailbreak_detection` como primera capa
- Verificar estado en `init()` de la app
- Combinar con verificación server-side

**OperationPermissionChecker:**
- Dado `riskLevel` y `operationType`, determinar acción
- Retorna: `allowed`, `requiresStepUp`, `blocked`
- Configurable desde remote config

---

#### 2.2 Android — Integración Nativa

**LocalRootDetection Android (Platform Channel):**
- Verificar existencia de binarios: `/system/bin/su`, `/system/xbin/su`, `/sbin/su`
- Verificar build tags: `test-keys` en `android.os.Build.TAGS`
- Verificar propiedades: `ro.debuggable`, `ro.secure`
- Verificar apps conocidas: Superuser.apk, Magisk Manager
- Detectar hooks: verificar integridad de funciones críticas

**PlayIntegrityDatasource:**
- Usar `play_integrity` package o Platform Channel directo
- Solicitar integrity token con nonce único por request
- Enviar token a backend para verificación
- Backend valida con Google API y retorna verdict

**FridaDetector Android:**
- Verificar puertos típicos de Frida (27042)
- Verificar mapas de memoria por strings de Frida
- Verificar existencia de `frida-server` en procesos

**Consideraciones Android:**
- Play Integrity requiere Google Play Services
- Para dispositivos sin Play Services, confiar más en checks locales
- API 23+ requerido para algunas funciones de Keystore

---

#### 2.3 iOS — Integración Nativa

**LocalJailbreakDetection iOS (Platform Channel):**
- Verificar paths de jailbreak: `/Applications/Cydia.app`, `/private/var/lib/apt`
- Intentar escribir fuera de sandbox
- Verificar integridad de funciones del sistema
- Detectar Substrate/Substitute hooks

**iOSAttestationDatasource:**
- App Attest: `DCAppAttestService.shared.attestKey`
- Generar key única por instalación
- Attestar key con Apple servers
- DeviceCheck: `DCDevice.current.generateToken` para verificación adicional
- Ambos requieren validación server-side con Apple API

**FridaDetector iOS:**
- Verificar dylibs cargadas sospechosas
- Verificar integridad de funciones críticas
- Detectar Cycript/Substrate

**Consideraciones iOS:**
- App Attest disponible desde iOS 14+
- Para iOS < 14, confiar en checks locales
- DeviceCheck permite bits persistentes para marcar dispositivos sospechosos

---

### Fase 3: Observability — Métricas y Alertas

**Métricas:**
- `device.security.risk_level` (distribución por nivel)
- `device.security.root_detected`
- `device.security.jailbreak_detected`
- `device.security.attestation_failed`
- `device.security.frida_detected` (CRÍTICO)
- `device.security.operations_blocked`

**Alertas:**

| Evento | Severidad | Acción |
|:-------|:----------|:-------|
| `frida_detected` | P1 Critical | Alerta inmediata a SOC |
| `attestation_failed` masivo | P2 High | Investigar posible ataque |
| `root_detected` + operación sensible | P3 Medium | Log para análisis |

---

## 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

### TACs Flutter (Cross-Platform)

```
[ ] TAC-4.1-FLUTTER: El sistema DEBE implementar detección en múltiples capas
    (local + server-side attestation).

[ ] TAC-4.2-FLUTTER: La respuesta DEBE ser adaptativa según operación, NO binaria.

[ ] TAC-4.3-FLUTTER: DeviceSecurityProvider DEBE exponer riskLevel y
    canPerformOperation(operation).

[ ] TAC-4.4-FLUTTER: Riesgo DEBE re-evaluarse cada 5-15 minutos durante sesión.

[ ] TAC-4.5-FLUTTER: NUNCA revelar QUÉ check falló (no educar al atacante).

[ ] TAC-4.6-FLUTTER: Métricas DEBEN alimentar modelos de fraude.
```

### TACs Android

```
[ ] TAC-4.7-ANDROID: DEBE usarse Play Integrity API para operaciones > $1000.

[ ] TAC-4.8-ANDROID: Attestation DEBE validarse server-side, NUNCA confiar solo
    en respuesta local.

[ ] TAC-4.9-ANDROID: DEBE detectar hooking frameworks activos (Frida, Xposed).

[ ] TAC-4.10-ANDROID: Ante Frida/hooking activo, BLOQUEAR operaciones financieras.

[ ] TAC-4.11-ANDROID: Para dispositivos sin Play Services, usar checks locales
    con funcionalidad reducida.
```

### TACs iOS

```
[ ] TAC-4.12-IOS: DEBE usarse App Attest para verificación criptográfica (iOS 14+).

[ ] TAC-4.13-IOS: DEBE implementarse DeviceCheck para marcar dispositivos sospechosos.

[ ] TAC-4.14-IOS: DEBE detectar Substrate/Substitute hooks activos.

[ ] TAC-4.15-IOS: Para iOS < 14, DEBE implementarse detección equivalente en código.
```

### TACs Backend (Referencia)

```
[ ] TAC-4.16-BACKEND: DEBE validar tokens de Play Integrity con Google API.

[ ] TAC-4.17-BACKEND: DEBE validar attestations de App Attest con Apple API.

[ ] TAC-4.18-BACKEND: DEBE existir mecanismo de apelación para usuarios bloqueados.

[ ] TAC-4.19-BACKEND: Risk scoring DEBE ser configurable sin release de app.
```

---

## 6. Estrategia de Pruebas (Shift-Left)

### Stack de Testing
- **Unit:** `flutter_test`, `mocktail` para providers
- **Integration:** `integration_test` con dispositivo físico
- **Security:** Pruebas en dispositivos rooteados reales, Frida attach tests

### Escenarios Críticos Obligatorios

| # | Escenario | Qué Validar | Plataforma | Tipo |
|:-:|:----------|:------------|:-----------|:-----|
| 1 | **Magisk con MagiskHide** | Checks locales pueden fallar, Play Integrity DEBE detectar. | Android | Security |
| 2 | **Frida attach en runtime** | DEBE detectar y bloquear operaciones. Alerta a SOC. | Android + iOS | Security |
| 3 | **Dispositivo sin Play Services** | NO bloquear, usar checks locales con funcionalidad reducida. | Android | Integration |
| 4 | **App Attest en iOS 14+** | Verificar que attestation funciona correctamente. | iOS | Integration |
| 5 | **Jailbreak con Checkra1n** | DEBE detectar y aplicar políticas de riesgo. | iOS | Security |

---

## Referencias

- [Google Play Integrity API](https://developer.android.com/google/play/integrity)
- [Apple App Attest](https://developer.apple.com/documentation/devicecheck/establishing_your_app_s_integrity)
- [Apple DeviceCheck](https://developer.apple.com/documentation/devicecheck)
- [OWASP Mobile Security Testing Guide - Anti-Tampering](https://owasp.org/www-project-mobile-security-testing-guide/)

---

*Anterior: [Certificate Pinning](caso-03-mitm-certificate-pinning.md) | Siguiente: [Secure PIN Pad](caso-05-secure-pin-pad.md)*
