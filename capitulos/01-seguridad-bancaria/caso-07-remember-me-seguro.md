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

> *"Como usuario de e-commerce, quiero que la app recuerde mi sesión para no tener que ingresar mi contraseña cada vez que abro la app."*

Balance crítico: Fricción cero vs Seguridad. E-commerce tolera más riesgo que banca, pero maneja datos de pago y direcciones.

### Evidencia de Industria

**Amazon 1-Click Patent:** Amazon patentó y popularizó la compra con un click, demostrando que reducir fricción aumenta conversión. Balancea conveniencia con re-auth para cambios de cuenta.

**Estudio Baymard Institute 2023:** 17% de abandonos de carrito ocurren por proceso de checkout largo, incluyendo re-autenticación forzada.

**Caso Uber (2016):** Brecha expuso datos de 57M usuarios. Sesiones persistentes sin protección adecuada facilitaron el acceso.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Compras fraudulentas si dispositivo robado o prestado |
| **Usabilidad** | Demasiada fricción = abandono de carrito, churn |
| **Compliance** | PCI-DSS requiere protección de credenciales almacenadas |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Guardar user/password en SharedPreferences para auto-login | **INADECUADO:** Credenciales en texto plano. NUNCA guardar contraseña. Backup extrae datos. |
| **ACEPTABLE** | Refresh token en flutter_secure_storage, auto-login sin interacción | **CUMPLE MÍNIMOS:** No guarda contraseña, usa token. PERO: dispositivo desbloqueado = acceso total sin verificación. |
| **ENTERPRISE** | **Biometric-Gated Access:** Token cifrado en secure storage, biometría para desbloquear sesión, step-up para operaciones sensibles | **ÓPTIMO:** Conveniencia + capa de protección biométrica. Cumple balance UX/Seguridad. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Auto-login rápido protegido por biometría. Sin almacenar contraseña en dispositivo. Step-up configurable por tipo de operación. Revocación remota de sesiones. Múltiples dispositivos gestionados. |
| **Restricciones Duras (NO permite)** | **Sin biometría:** Dispositivos sin biometría deben ofrecer alternativa (PIN de app). **Biometría comprometida:** Si alguien enrolla su huella, protección es nula. **Dispositivo compartido:** No recomendado para tablets familiares. **Biometría opcional:** Usuario puede rechazar, debe existir fallback. |
| **Criterio de Selección** | Se usa **Cubit** porque estado de Remember Me es simple (enabled/disabled/authenticating), sin eventos complejos. **flutter_secure_storage** para refresh token, **Hive encrypted** para preferencias y metadata no crítica. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Remember Me | Opción de sesión persistente sin pedir credenciales en cada inicio. |
| Refresh Token Rotation | Generar un refresh nuevo en cada uso e invalidar el anterior. |
| Device Binding | Asociar tokens a fingerprint del dispositivo para evitar replay en otro hardware. |
| Sliding Session | Sesión que extiende expiración con actividad; usada en modelos menos estrictos. |
| Remote Logout | Capacidad de invalidar tokens y limpiar sesión vía push desde backend. |
| TACs | Criterios de aceptación técnicos para validar controles de seguridad. |

---

## Referencias

- [OWASP Mobile Security - Authentication](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Android BiometricPrompt](https://developer.android.com/reference/android/hardware/biometrics/BiometricPrompt)
- [iOS LocalAuthentication](https://developer.apple.com/documentation/localauthentication)
- [Baymard Institute - Cart Abandonment Statistics](https://baymard.com/lists/cart-abandonment-rate)

---

*Anterior: [Session Hijacking](caso-06-session-hijacking-wifi.md) | Siguiente: [MFA SIM Swapping](caso-08-mfa-sim-swapping.md)*
