# Caso 4: Rooted pero Confiable
## Detectar Dispositivos Comprometidos sin Bloquear Usuarios Legítimos

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | root detection, jailbreak detection, dispositivo comprometido, fraude, seguridad móvil, SafetyNet, Play Integrity |
| **Patrón Técnico** | Device Attestation, Risk-Based Authentication, Layered Security |
| **Stack Seleccionado** | flutter_jailbreak_detection + Platform Channels (SafetyNet/Play Integrity) + Riverpod (DeviceSecurityProvider) |
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
| **Regulatory** | Algunos reguladores exigen detección de dispositivos comprometidos |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Rol | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:----|:-----------------------|:----------------------------------|
| **Junior** | `flutter_jailbreak_detection` → si detecta root → bloquear app | **FALLA:** Falsos positivos, fácilmente bypasseable con Magisk Hide, no distingue riesgo real, UX hostil. |
| **Senior** | Múltiples checks (su binary, build tags) + warning pero permitir con funcionalidad reducida | **MEJORA:** Reduce falsos positivos. PERO: aún bypasseable, sin verificación server-side, atacante puede parchear binario. |
| **Architect** | **Device Attestation multi-capa:** 1) Checks locales, 2) Play Integrity/SafetyNet (Android) y DeviceCheck/App Attest (iOS) server-side, 3) Risk scoring dinámico, 4) Respuesta adaptativa | **ENTERPRISE:** Verificación criptográfica server-side. Difícil bypassear. Políticas granulares por operación. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Detectar root/jailbreak con múltiples técnicas. Verificación criptográfica server-side (Play Integrity). Risk scoring continuo. Respuestas granulares (permitir lectura, bloquear escritura). Detectar hooking frameworks (Frida, Xposed). |
| **Restricciones Duras (NO permite)** | **100% infalible:** Siempre existirán bypasses con suficiente esfuerzo. **Offline:** Play Integrity requiere conexión. **Emuladores:** Detectados como riesgo, afecta devs. **Play Services:** Play Integrity requiere Google Play Services. |
| **Criterio de Selección** | Se usa **Riverpod** con `DeviceSecurityProvider` porque: estado de seguridad se consulta desde múltiples features, Riverpod permite dependency injection limpio, fácil testing con overrides. Play Integrity sobre SafetyNet porque SafetyNet está deprecated. |

---

## 4. Manos a la Obra: Estrategia de Implementación

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

### Fase 2: Implementación — Detalles Técnicos

**LocalRootDetection (Checks Locales):**
- Verificar existencia de binarios: `/system/bin/su`, `/system/xbin/su`, `/sbin/su`
- Verificar build tags: `test-keys` en `android.os.Build.TAGS`
- Verificar propiedades: `ro.debuggable`, `ro.secure`
- Verificar apps conocidas: Superuser.apk, Magisk Manager
- Detectar hooks: verificar integridad de funciones críticas

**PlayIntegrityDatasource (Android):**
- Usar `play_integrity` package o Platform Channel
- Solicitar integrity token con nonce único por request
- Enviar token a backend para verificación
- Backend valida con Google API y retorna verdict

**iOSAttestationDatasource:**
- App Attest: `DCAppAttestService.shared.attestKey`
- DeviceCheck: `DCDevice.current.generateToken`
- Ambos requieren validación server-side con Apple API

**DeviceSecurityProvider (Riverpod):**
- `AsyncNotifierProvider` para estado de seguridad
- Evaluar al inicio de app y periódicamente
- Cachear resultado con TTL (5-15 minutos)
- Exponer `riskLevel` y `canPerformOperation(operation)`

### Fase 3: Observability — Métricas

**Métricas:**
- `device.security.risk_level` (distribución por nivel)
- `device.security.root_detected`
- `device.security.attestation_failed`
- `device.security.frida_detected` (CRÍTICO)

---

## 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

```
[ ] TAC-4.1: El sistema DEBE implementar detección en múltiples capas
    (local + server-side attestation).

[ ] TAC-4.2: La respuesta DEBE ser adaptativa según operación, NO binaria.

[ ] TAC-4.3: Para operaciones > $1000, DEBE usarse Play Integrity (Android)
    o App Attest (iOS).

[ ] TAC-4.4: Attestation DEBE validarse server-side, NUNCA confiar solo en local.

[ ] TAC-4.5: DEBE detectar hooking frameworks activos (Frida, Xposed, Substrate).

[ ] TAC-4.6: Ante Frida/hooking activo, BLOQUEAR operaciones financieras.

[ ] TAC-4.7: Riesgo DEBE re-evaluarse cada 5-15 minutos durante sesión.

[ ] TAC-4.8: DEBE existir mecanismo de apelación para usuarios bloqueados.

[ ] TAC-4.9: Métricas DEBEN alimentar modelos de fraude.

[ ] TAC-4.10: NUNCA revelar QUÉ check falló (no educar al atacante).
```

---

## 6. Estrategia de Pruebas (Shift-Left)

### Escenarios Críticos Obligatorios

| # | Escenario | Qué Validar | Tipo |
|:-:|:----------|:------------|:-----|
| 1 | **Magisk con MagiskHide** | Checks locales pueden fallar, Play Integrity DEBE detectar. | Security |
| 2 | **Frida attach en runtime** | DEBE detectar y bloquear operaciones. Alerta a SOC. | Security |
| 3 | **Dispositivo sin Play Services** | NO bloquear, usar checks locales. Operaciones bajo riesgo permitidas. | Integration |

---

*Anterior: [Certificate Pinning](caso-03-mitm-certificate-pinning.md) | Siguiente: [Secure PIN Pad](caso-05-secure-pin-pad.md)*
