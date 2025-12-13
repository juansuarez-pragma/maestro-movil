# Caso 4: Rooted pero Confiable
## Detectar Dispositivos Comprometidos sin Bloquear Usuarios Legítimos

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | root detection, jailbreak detection, dispositivo comprometido, fraude, seguridad móvil, SafetyNet, Play Integrity |
| **Patrón Técnico** | Device Attestation, [Risk-Based Authentication](#term-risk-auth "Ajustar controles (permitir/limitar/bloquear) según riesgo del dispositivo."), Layered Security |
| **Stack Seleccionado** | flutter_jailbreak_detection + Platform Channels (Play Integrity/App Attest) + Riverpod (DeviceSecurityProvider) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Detecciones locales de root/jailbreak son triviales de evadir (Magisk/Zygisk, hooking con Frida/Xposed); sin atestación server-side ([Play Integrity](#term-play-integrity), [App Attest](#term-app-attest)) el riesgo se subestima.
- Bloqueo binario a todo dispositivo rooteado genera falsos positivos en devs o usuarios de ROMs seguras; se necesita respuesta graduada (lectura/consulta sí, operaciones sensibles no).
- Sin telemetría de riesgo ni eventos de bypass, el SOC no detecta intentos de evasión ni patrones de fraude ligados a dispositivos comprometidos.

### Escenario de Negocio

> *"Como oficial de seguridad del banco, quiero detectar dispositivos rooteados/jailbroken para evaluar el riesgo, pero sin bloquear usuarios legítimos que usan custom ROMs por privacidad."*

- Caso: cliente premium con ROM personalizada para privacidad es bloqueado al iniciar sesión → churn y ticket crítico.
- Caso opuesto: dispositivo rooteado con malware Cerberus roba OTP y tokens; sin controles server-side, el fraude prospera.

### Incidentes reportados

- **Cerberus Banking Trojan (2020-2023):** Malware Android que requiere root. Intercepta SMS OTP y extrae datos de apps bancarias (España, México, Brasil).
- **MagiskHide/Zygisk:** Ocultamiento de root a apps específicas; usado por atacantes para evadir detecciones superficiales.
- **Promon (2022):** 50% de apps bancarias top 100 pueden ser bypaseadas en root con Frida/Xposed. Rooteados ~2% de base, pero ~30% de intentos de fraude.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Promon (2022) | 100 apps bancarias | 50% podía ser bypaseada en root con Frida/Xposed. |
| NowSecure State of Mobile App Security (2024) | 1,000+ apps móviles (US/EU) | 85% fallan ≥1 control MASVS; integridad del dispositivo es un fallo recurrente. |
| Kaspersky Mobile Threats (2023) | LATAM/APAC | Aumento de malware que exige root para robar OTP/tokens (SharkBot, Anubis). |

**Resumen global**
- La base rooteada es pequeña (~2%), pero concentra ~30% de intentos de fraude.
- La mitad de las apps bancarias evaluadas puede ser evadida en dispositivos comprometidos (Promon 2022).
- Control de integridad es una de las brechas más frecuentes en MASVS (NowSecure 2024).

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
| **Capacidades (SÍ permite)** | Detectar root/jailbreak con múltiples técnicas. Verificación criptográfica server-side ([Play Integrity](#term-play-integrity), [App Attest](#term-app-attest)). Risk scoring continuo. Respuestas granulares (lectura permitida, operaciones sensibles bloqueadas). Detectar hooking frameworks (Frida, Xposed). |
| **Restricciones Duras (NO permite)** | **100% infalible:** Siempre habrá bypasses avanzados. **Offline:** [Play Integrity](#term-play-integrity) requiere conexión. **Emuladores:** Clasificados como alto riesgo; afecta devs. **Dependencias:** Play Integrity depende de Google Play Services. |
| **Criterio de Selección** | Se usa **[Riverpod](#term-riverpod)** porque el estado de seguridad se consume en múltiples features y requiere DI/test fácil. [Play Integrity](#term-play-integrity) sobre SafetyNet por deprecación. Respuesta adaptativa evita falsos positivos masivos. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Provider entrega estados consistentes (secure → degraded → blocked) ante flags simulados | Equipo móvil, CI |
| Integration (CI) | Token de [Play Integrity](#term-play-integrity) inválido → backend degrada operaciones; token válido → operaciones completas | Móvil/Backend, CI + staging |
| Seguridad manual | Bypass con Magisk/Frida se detecta; operaciones sensibles bloqueadas con mensaje claro | Seguridad/QE, dispositivos rooteados |
| Observabilidad | Evento `device.risk` con tipo (root, bypass), score y acción tomada | Móvil/SRE, CI |

### 3.2 UX, accesibilidad y respuesta adaptativa
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Mensajes | Diferenciar consulta permitida vs operaciones bloqueadas; CTA a usar dispositivo no comprometido | Reduce churn y clarifica restricción |
| Consentimiento | Registrar aceptación al continuar en modo degradado (solo consulta) | Evidencia para auditoría |
| Reintentos | Limitar reintentos; mostrar motivo (root, atestación fallida, hooking) | Evita loops e informa al usuario |

### 3.3 Operación y riesgo
- Política de riesgo: root/jailbreak = riesgo alto → solo lectura; integridad válida = operaciones completas.
- Revalidar integridad en operaciones sensibles (transferencias, cambio de límites).
- Alertar al SOC al detectar bypass repetido (>3 intentos/hora).

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Dispositivos rooteados permiten malware/hooking; bloqueo total genera falsos positivos. |
| Opciones evaluadas | Bloqueo binario por root; solo checks locales; atestación server-side con respuesta adaptativa. |
| Decisión | Atestación server-side (Play Integrity/App Attest) + respuesta graduada (consulta vs operaciones) + telemetría. |
| Consecuencias | Dependencia de servicios de atestación y conectividad; mayor complejidad operativa. |
| Riesgos aceptados | Usuarios en ROMs seguras quedan en modo consulta; posibles fallas en baja conectividad. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Fraudes desde dispositivos comprometidos | ↓ ≥ 60% | Alerta si no reduce tras rollout | Beneficio directo de negocio |
| Falsos positivos | Usuarios legítimos bloqueados ≈ 0; modo consulta para casos borderline | Alerta si > 0.5% usuarios activos | Balance UX-seguridad |
| Éxito de atestación | > 98% en dispositivos compliant | Alerta si baja por 5 min | Confiabilidad de flujo |
| Detección de bypass (Magisk/Frida) | Alertas en < 5 s | Eventos > 3/hora disparan alerta SOC | Respuesta rápida a intentos |
| Tickets por bloqueo | ↓ 20% en 4 semanas | Alertar si no baja | Costos de soporte controlados |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-root-detection"></a>Root/Jailbreak Detection | Técnicas para identificar dispositivos comprometidos (su binary, build tags, rutas sospechosas). |
| <a id="term-play-integrity"></a>Play Integrity | Atestación criptográfica Android (reemplaza SafetyNet) para validar integridad del dispositivo. |
| <a id="term-app-attest"></a>App Attest / DeviceCheck | Atestación y signals de integridad en iOS. |
| <a id="term-risk-auth"></a>Risk-Based Authentication | Ajustar controles (permitir/limitar/bloquear) según riesgo del dispositivo. |
| <a id="term-hooking"></a>Hooking Frameworks | Herramientas como Frida/Xposed que interceptan/alteran código en runtime. |
| <a id="term-attestation-token"></a>Attestation Token | Evidencia firmada usada por backend para decidir si permite operaciones sensibles. |
| <a id="term-riverpod"></a>Riverpod | Contenedor de DI/estado reactivo usado para compartir estado de seguridad entre features. |

---

## Referencias

- [Google Play Integrity API](https://developer.android.com/google/play/integrity)
- [Apple App Attest](https://developer.apple.com/documentation/devicecheck/establishing_your_app_s_integrity)
- [Apple DeviceCheck](https://developer.apple.com/documentation/devicecheck)
- [OWASP Mobile Security Testing Guide - Anti-Tampering](https://owasp.org/www-project-mobile-security-testing-guide/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
- [Kaspersky - Mobile Threats 2023](https://securelist.com/mobile-malware-evolution-2023/111742/)
- [Promon Report 2022](https://promon.co/security-news/)

---

*Anterior: [Certificate Pinning](caso-03-mitm-certificate-pinning.md) | Siguiente: [Secure PIN Pad](caso-05-secure-pin-pad.md)*
