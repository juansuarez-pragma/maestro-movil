# Caso 7: El Dilema del Remember Me
## Persistencia Segura de Credenciales en E-commerce

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | remember me, persistencia de sesión, login automático, credenciales guardadas, e-commerce |
| **Patrón Técnico** | Secure Credential Storage, Biometric-Gated Access, Device Trust |
| **Stack Seleccionado** | flutter_secure_storage + local_auth + Hive (encrypted) + Cubit (RememberMeCubit) |
| **Nivel de Criticidad** | Medio |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario de e-commerce, quiero que la app recuerde mi sesión para no tener que ingresar mi contraseña cada vez."*

Balance: Fricción cero vs Seguridad. E-commerce tolera más riesgo que banca, pero maneja datos de pago.

### Evidencia de Industria

**Amazon 1-Click:** Balancea conveniencia con seguridad: 1-click para compras, re-auth para cambios de cuenta.

**Estadística Baymard:** 17% de abandonos por checkout largo, incluyendo re-auth forzada.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Compras fraudulentas si dispositivo robado |
| **Usabilidad** | Demasiada fricción = abandono |
| **Compliance** | PCI-DSS requiere protección de credenciales |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Rol | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:----|:-----------------------|:----------------------------------|
| **Junior** | Guardar user/password en SharedPreferences | **FALLA:** Credenciales en texto plano. NUNCA guardar contraseña. |
| **Senior** | Refresh token en secure storage, auto-login | **MEJORA:** No guarda contraseña. PERO: dispositivo desbloqueado = acceso total |
| **Architect** | **Biometric-Gated:** Token cifrado, biometría para desbloquear, step-up para operaciones sensibles | **ENTERPRISE:** Conveniencia + protección biométrica |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Auto-login rápido con biometría. Sin guardar contraseña. Step-up configurable. Revocación remota. |
| **Restricciones Duras (NO permite)** | **Sin biometría:** Debe ofrecer alternativa (PIN). **Biometría comprometida:** Protección nula. **Dispositivo compartido:** No recomendado. |
| **Criterio de Selección** | Se usa **Cubit** porque estado simple (enabled/disabled). **flutter_secure_storage** para token, **Hive encrypted** para preferencias. |

---

## 4. Manos a la Obra: Estrategia de Implementación

### Fase 1: Flujos

**Activación:**
1. Login exitoso → Prompt "¿Guardar sesión?"
2. Acepta → Token en secure storage, biometric-protected
3. Siguiente launch → Biometría → Refresh token → Sesión

**Operaciones Sensibles:**
- Cambiar dirección → Re-auth biométrica
- Agregar pago → Re-auth + OTP
- Compra > $500 → Re-auth biométrica

### Fase 2: Implementación

**RememberMeCubit:**
- Estados: `initial`, `enabled`, `disabled`, `authenticating`
- Métodos: `enable`, `disable`, `authenticateWithBiometric`

**SecureTokenStorage:**
- iOS: `authenticationRequired` en Keychain
- Android: BiometricPrompt antes de acceso

**BiometricGate:**
- Wrap de local_auth
- Fallback: biometría → PIN app → contraseña
- Tracking intentos fallidos

---

## 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

```
[ ] TAC-7.1: Remember Me DEBE ser opt-in, nunca default.
[ ] TAC-7.2: NUNCA almacenar contraseña, solo refresh tokens.
[ ] TAC-7.3: Acceso a token DEBE requerir biometría o PIN.
[ ] TAC-7.4: DEBE existir opción de desactivar y limpiar.
[ ] TAC-7.5: Tras 3 fallos biometría → login tradicional.
[ ] TAC-7.6: Operaciones sensibles (> $500) DEBEN requerir step-up.
[ ] TAC-7.7: "Cerrar sesión en todos dispositivos" DEBE invalidar tokens.
[ ] TAC-7.8: Refresh token TTL máximo 30 días.
[ ] TAC-7.9: Cambio de biometría enrollada → invalidar tokens.
[ ] TAC-7.10: Warning sobre riesgos en dispositivos compartidos.
```

---

## 6. Estrategia de Pruebas (Shift-Left)

| # | Escenario | Qué Validar | Tipo |
|:-:|:----------|:------------|:-----|
| 1 | **Activar → cerrar → reabrir** | Pide biometría, no password | E2E |
| 2 | **Token expirado** | Biometría OK pero refresh falla → login tradicional | Integration |
| 3 | **Desactivar Remember Me** | Tokens limpiados, siguiente apertura pide login | Integration |

---

*Anterior: [Session Hijacking](caso-06-session-hijacking-wifi.md) | Siguiente: [MFA SIM Swapping](caso-08-mfa-sim-swapping.md)*
