# Caso 8: OTP Interceptado
## Implementando MFA Resistente a SIM Swapping

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | OTP, SMS, SIM swapping, MFA, autenticación multifactor, TOTP, push notification |
| **Patrón Técnico** | TOTP, Push-based MFA, Hardware Tokens |
| **Stack Seleccionado** | otp (TOTP) + firebase_messaging (Push MFA) + BLoC (MFABloc) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero recibir un código de verificación para aprobar transacciones importantes."*

Problema con SMS: SIM swapping permite a atacantes tomar control del número telefónico.

### Evidencia de Industria

**Caso Twitter/Jack Dorsey (2019):** CEO víctima de SIM swap a pesar de MFA con SMS.

**Estadística FBI:** Ataques SIM swapping aumentaron 400% (2018-2021), pérdidas de $68M en 2021.

**NIST SP 800-63B:** Desaconseja SMS como único factor desde 2016.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Account takeover completo |
| **Regulatorio** | NIST y reguladores desaconsejan SMS |
| **Técnico** | SMS tiene latencia, puede no llegar |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Rol | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:----|:-----------------------|:----------------------------------|
| **Junior** | SMS OTP via Twilio | **FALLA:** SIM swap, SS7 attacks, interception. Costo. |
| **Senior** | TOTP (Google Authenticator) como alternativa | **MEJORA:** No depende de carrier, offline. PERO: seed debe protegerse, otra app. |
| **Architect** | **MFA Híbrido:** Push primario, TOTP backup, SMS solo recovery con friction | **ENTERPRISE:** Push más seguro y mejor UX. TOTP offline. SMS excepcional. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Push MFA con aprobación biométrica. TOTP estándar RFC 6238. Múltiples dispositivos MFA. Códigos backup. |
| **Restricciones Duras (NO permite)** | **Sin internet:** Push no funciona (TOTP sí). **Dispositivo perdido:** Necesita recovery robusto. **Push delivery:** No garantizado inmediato. |
| **Criterio de Selección** | Se usa **BLoC** porque flujo MFA tiene múltiples estados y transiciones complejas (requesting→sent→waiting→verifying). |

---

## 4. Manos a la Obra: Estrategia de Implementación

### Fase 1: Jerarquía de Métodos

**Prioridad:**
1. **Push MFA** (mejor UX + seguridad)
2. **TOTP** (fallback offline)
3. **Email OTP** (fallback sin dispositivo)
4. **SMS** (último recurso + verificación adicional)

**Flujo Push MFA:**
1. Usuario inicia operación
2. Backend envía push
3. App muestra detalles de operación
4. Usuario aprueba con biometría
5. Confirmación firmada al backend

### Fase 2: Implementación

**MFABloc Estados:**
- `MFAPushSent`, `MFAPushWaiting`, `MFAPushApproved`
- `MFATOTPRequested`, `MFATOTPVerifying`
- `MFASuccess`, `MFAFailure`

**PushMFAService:**
- Registrar dispositivo con FCM token
- Almacenar device_mfa_key en secure storage
- Verificar firma de push
- Firmar respuesta antes de enviar

**TOTPService:**
- Package `otp` para RFC 6238
- Seed en flutter_secure_storage
- Aceptar código ± 1 ventana (30 seg tolerancia)

---

## 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

```
[ ] TAC-8.1: SMS NO DEBE ser método primario para operaciones financieras.
[ ] TAC-8.2: DEBE ofrecerse Push MFA con aprobación biométrica.
[ ] TAC-8.3: DEBE ofrecerse TOTP como alternativa offline.
[ ] TAC-8.4: SMS como fallback DEBE requerir verificación adicional.
[ ] TAC-8.5: DEBEN generarse códigos backup durante setup.
[ ] TAC-8.6: Seed TOTP DEBE almacenarse cifrado.
[ ] TAC-8.7: Push DEBE incluir detalles de operación.
[ ] TAC-8.8: Rate limiting: máximo 5 intentos/hora.
[ ] TAC-8.9: Cambio de método DEBE requerir método actual + contraseña.
[ ] TAC-8.10: DEBE existir recovery si pierde todos los factores.
```

---

## 6. Estrategia de Pruebas (Shift-Left)

| # | Escenario | Qué Validar | Tipo |
|:-:|:----------|:------------|:-----|
| 1 | **Push desde otro dispositivo** | Solo dispositivo registrado puede aprobar | Security |
| 2 | **TOTP expirado** | Código > 60 seg rechazado, 30 seg aceptado | Unit |
| 3 | **Todos los métodos fallidos** | Cuenta bloqueada, recovery manual | E2E |

---

*Anterior: [Remember Me](caso-07-remember-me-seguro.md) | Siguiente: [PCI-DSS Tokenización](caso-09-pci-dss-tokenizacion.md)*
