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

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Root/Jailbreak Detection | Conjunto de técnicas para identificar dispositivos comprometidos (su binary, build tags, rutas sospechosas). |
| Play Integrity / SafetyNet | Servicio Android de atestación criptográfica server-side para validar integridad del dispositivo. |
| DeviceCheck / App Attest | Equivalentes en iOS para verificar estado de dispositivo y apps. |
| Risk-Based Authentication | Ajustar controles (permitir/limitar/bloquear) según nivel de riesgo del dispositivo. |
| Hooking Frameworks | Herramientas como Frida/Xposed que interceptan/alteran código en runtime. |
| Attestation Token | Evidencia firmada usada por backend para decidir si permite operaciones sensibles. |

---

## Referencias

- [Google Play Integrity API](https://developer.android.com/google/play/integrity)
- [Apple App Attest](https://developer.apple.com/documentation/devicecheck/establishing_your_app_s_integrity)
- [Apple DeviceCheck](https://developer.apple.com/documentation/devicecheck)
- [OWASP Mobile Security Testing Guide - Anti-Tampering](https://owasp.org/www-project-mobile-security-testing-guide/)

---

*Anterior: [Certificate Pinning](caso-03-mitm-certificate-pinning.md) | Siguiente: [Secure PIN Pad](caso-05-secure-pin-pad.md)*
