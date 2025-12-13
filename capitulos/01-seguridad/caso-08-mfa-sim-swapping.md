# Caso 8: OTP Interceptado
## Implementando [MFA](#term-mfa "Autenticación multifactor; combina algo que sabes, tienes o eres.") Resistente a [SIM Swapping](#term-sim-swapping "Robo o duplicado de SIM para recibir OTP/SMS y tomar control de cuentas.")

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | OTP, SMS, SIM swapping, MFA, autenticación multifactor, [TOTP](#term-totp "OTP basado en tiempo (RFC 6238), menos dependiente de la red móvil."), push notification |
| **Patrón Técnico** | TOTP (RFC 6238), Push-based MFA, Hardware Tokens |
| **Stack Seleccionado** | otp (TOTP) + firebase_messaging ([Push MFA](#term-push-mfa "Aprobación de login mediante notificación push con contexto de riesgo.")) + BLoC (MFABloc) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- SMS como segundo factor es vulnerable a SIM swapping, SS7, malware de SMS y call forwarding.
- Sin canal alterno (push/TOTP), la UX fuerza SMS, exponiendo a fraudes de ATO.
- Seeds TOTP mal protegidos o sincronizaciones inseguras abren puerta a clonación.

### Escenario de Negocio

> *"Como usuario, quiero recibir un código de verificación para aprobar transacciones importantes, pero me preocupa la seguridad de los SMS."*

- Caso: SIM swap en carrier, [OTP SMS](#term-otp-sms "One-Time Password enviado por SMS; vulnerable a SIM swap y malware de SMS.") redirigido, se aprueban transferencias.
- Caso: usuario sin datos (offline) necesita alternativa (TOTP) para no bloquear operaciones legítimas.

### Incidentes reportados
- **Twitter/Jack Dorsey (2019):** SIM swap permitió secuestro de cuenta a pesar de MFA SMS.
- **Coinbase (2021):** SIM swapping afectó usuarios con 2FA SMS habilitado; se comprometieron fondos.
- **FBI IC3 (2021):** Ataques de SIM swap aumentaron 400% 2018–2021; pérdidas reportadas de $68M en 2021.
- **NIST SP 800-63B (2017):** Desaconseja SMS como único segundo factor por vulnerabilidades conocidas.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| FBI IC3 (2021) | Reportes EEUU | +400% ataques SIM swap 2018–2021; $68M pérdidas 2021. |
| Verizon DBIR (2023) | Global | Abuso de credenciales/sesiones continúa en top de vectores; SMS OTP vulnerable. |
| NowSecure State of Mobile App Security (2024) | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; MFA débil/SMS-only frecuente. |

**Resumen global**
- SIM swap en crecimiento (FBI: +400% en 3 años), con pérdidas significativas.
- SMS-only sigue siendo un punto débil recurrente en apps móviles (MASVS).
- Necesario canal primario seguro (push) y backup offline (TOTP) para balancear seguridad/UX.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Account takeover completo, transferencias fraudulentas |
| **Regulatorio** | NIST y reguladores desaconsejan SMS, PSD2 requiere SCA |
| **Técnico** | SMS tiene latencia variable, puede no llegar, depende de carrier |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | SMS OTP como único segundo factor | **INADECUADO:** Vulnerable a SIM swap, SS7 attacks, interception. Costo por SMS. Latencia. Dependencia de carrier. |
| **ACEPTABLE** | TOTP (Google Authenticator) como alternativa a SMS | **CUMPLE MÍNIMOS:** No depende de carrier, funciona offline, estándar RFC 6238. PERO: seed debe protegerse, requiere otra app, pérdida de dispositivo = pérdida de acceso. |
| **ENTERPRISE** | **MFA Híbrido:** Push MFA primario con aprobación in-app, TOTP como backup offline, SMS solo como último recurso con verificación adicional | **ÓPTIMO PARA BANCA:** Push es más seguro y mejor UX. TOTP para offline. SMS excepcional con fricción extra. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Push MFA con aprobación biométrica en la misma app. TOTP estándar RFC 6238 compatible con authenticators. Múltiples dispositivos MFA registrados. Códigos backup para recovery. Transacciones firmadas criptográficamente. |
| **Restricciones Duras (NO permite)** | **Sin internet:** Push no funciona, TOTP sí. **Dispositivo perdido:** Necesita recovery robusto. **Push delivery:** No garantizado inmediato (APNs/FCM pueden demorar). **Usuarios no técnicos:** TOTP puede confundir. |
| **Criterio de Selección** | Se usa **BLoC** porque flujo MFA tiene múltiples estados y transiciones complejas (requesting→sent→waiting→verifying→success/failure). Eventos claros permiten auditoría del flujo. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | BLoC transita estados MFA y maneja fallback push/TOTP | Equipo móvil, CI |
| Integration (CI) | Push MFA → biometría/confirmación; fallback TOTP válido; SMS solo como emergencia con fricción | Móvil/Backend, CI + staging |
| Seguridad manual | SIM swap emulado: SMS interceptado no permite aprobar; push requiere dispositivo legítimo | Seguridad/QE, dispositivos reales |
| Observabilidad | Eventos `mfa.*` con canal, motivo y resultado | Móvil/SRE, CI |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Canal primario | Push MFA con biometría/aprobación contextual | Mejor UX y seguridad |
| Backup offline | TOTP RFC 6238; backup codes para recovery | Disponibilidad sin red |
| SMS | Solo último recurso; agregar fricción (pregunta adicional, cooldown) | Reduce abuso de SIM swap |
| Cambio de dispositivo | Revalidar MFA y regenerar seeds; invalidar registros anteriores | Minimiza clonación |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Registro de factores | Limitar factores activos por usuario; registrar device_id/token para push | Reduce superficie |
| Revocación | Permitir revocar factor (push/TOTP) desde backend; bloquear SMS si hay sospecha | Respuesta a incidentes |
| Telemetría | Auditar canal usado, localización, IP/ASN; alertar anomalías | Apoya detección de fraude |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | SMS OTP es vulnerable a SIM swap; sin canal alterno seguro se expone ATO. |
| Opciones evaluadas | SMS-only; TOTP-only; Push + TOTP + SMS de emergencia. |
| Decisión | Push como primario (con biometría), TOTP como backup offline, SMS solo emergencia con fricción. |
| Consecuencias | Mayor complejidad de orquestación; necesita backend para registro/rotación de factores; depende de FCM/APNS. |
| Riesgos aceptados | Retrasos de push; usuarios sin biometría usan PIN/app; gestión de recuperación aumenta soporte. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Uso de SMS como canal | < 5% de MFA | Alerta si sube | Minimiza exposición a SIM swap |
| ATO vía SIM swap | ↓ ≥ 70% | Alerta si no baja tras rollout | Reducción de fraude |
| TOTP/adopción backup | > 60% usuarios activos MFA | Alerta si adopción baja | Disponibilidad offline |
| Tiempo de aprobación MFA | p95 < 5 s (push) | Warning si se acerca | UX ágil |
| Errores/latencia SMS | Monitorear; alertar si falla/latencia sube | Control de último recurso |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-sim-swapping"></a>SIM Swapping | Robo o duplicado de SIM para recibir OTP/SMS y tomar control de cuentas. |
| <a id="term-mfa"></a>MFA | Autenticación multifactor; combina algo que sabes, tienes o eres. |
| <a id="term-otp-sms"></a>OTP SMS | One-Time Password enviado por SMS; vulnerable a SIM swap y malware de SMS. |
| <a id="term-totp"></a>TOTP | OTP basado en tiempo (RFC 6238), menos dependiente de la red móvil. |
| <a id="term-push-mfa"></a>Push MFA | Aprobación de login mediante notificación push con contexto de riesgo. |
| <a id="term-webauthn-passkeys"></a>WebAuthn/Passkeys | Autenticación sin contraseña usando llaves FIDO2 y biometría del dispositivo. |
| <a id="term-backup-codes"></a>Backup Codes | Códigos de recuperación para acceso offline o pérdida de dispositivo principal. |

---

## Referencias

- [NIST SP 800-63B - Authentication Guidelines](https://pages.nist.gov/800-63-3/sp800-63b.html)
- [RFC 6238 - TOTP Algorithm](https://tools.ietf.org/html/rfc6238)
- [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging)
- [FIDO Alliance - Passwordless Authentication](https://fidoalliance.org/)
- [FBI IC3 Report 2021](https://www.ic3.gov/Media/PDF/AnnualReport/2021_IC3Report.pdf)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
- [Verizon DBIR 2023](https://www.verizon.com/business/resources/reports/dbir/)

---

*Anterior: [Remember Me](caso-07-remember-me-seguro.md) | Siguiente: [PCI-DSS Tokenización](caso-09-pci-dss-tokenizacion.md)*
