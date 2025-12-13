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

### Problema detectado (técnico)
- “Remember Me” basado solo en token guardado sin gating permite acceso completo si el dispositivo está desbloqueado o es compartido.
- Sin device binding ni rotación de refresh, el token puede usarse en otro dispositivo (replay) o mantenerse activo indefinidamente.
- Falta de step-up para operaciones sensibles (cambios de dirección, métodos de pago) expone a fraudes de cuenta.

### Escenario de Negocio

> *"Como usuario de e-commerce, quiero que la app recuerde mi sesión para no tener que ingresar mi contraseña cada vez que abro la app."*

Balance crítico: fricción vs seguridad. E-commerce tolera más riesgo que banca, pero maneja datos de pago/direcciones y fraude de ATO afecta reputación y chargebacks.

### Incidentes reportados

- **Uber 2016:** Brecha de 57M usuarios; sesiones persistentes facilitaron acceso no autorizado.
- **Baymard Institute 2023:** 17% de abandonos de carrito por procesos de checkout/re-auth largos.
- **Casos de ATO en retail:** Ataques de credential stuffing aprovechan sesiones persistentes sin revalidación para cambiar direcciones y hacer compras.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Baymard Institute (2023) | Estudios de checkout e-commerce | 17% de abandono por fricción (incluye re-auth forzada). |
| NowSecure State of Mobile App Security (2024) | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; gestión de sesión débil es recurrente. |
| ACFE Fraud Report (2023) | Global | Account Takeover en e-commerce sigue en top de fraudes con costo creciente. |

**Resumen global**
- Fricción alta impacta conversión (checkout): ~17% de abandono por procesos largos.
- Gestión de sesión débil es un hallazgo común en apps móviles (MASVS).
- ATO en retail/e-commerce sigue siendo de los principales tipos de fraude según ACFE.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Cubit de remember me cambia estados y respeta biometría/fallback | Equipo móvil, CI |
| Integration (CI) | Auto-login requiere biometría si está habilitada; sin biometría → pide PIN/app | Móvil/QA, CI + staging |
| Seguridad manual | Token extraído no usable en otro dispositivo (device binding) | Seguridad/QE, dispositivos reales |
| Observabilidad | Eventos `remember.*` con motivo de fallback/biometría/falla | Móvil/SRE, CI |

### 3.2 UX, accesibilidad y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Biometría | Usar biometría para desbloquear sesión si está disponible; fallback a PIN de app | Balance UX/seguridad |
| Fallback | Si el usuario rechaza biometría, habilitar remember con PIN; mostrar riesgos | Transparencia y control |
| Step-up | Requerir re-auth biométrica para cambios de pago/dirección | Protege operaciones sensibles |
| Tokens | No exponer usuario a re-login frecuente; pero forzar login tras revocación remota o reuse detectado | Control de riesgo |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Device binding | Asociar refresh a fingerprint de dispositivo; invalidar si cambia | Evita replay cross-device |
| Rotación de tokens | Refresh rotation single-use; invalidar familia ante reuse | Cierra ventana de reuso |
| Revocación remota | Push para logout remoto y limpieza de storage | Respuesta rápida a fraude |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | “Remember Me” sin gating ni binding expone a ATO y abuso en dispositivos compartidos. |
| Opciones evaluadas | Token sin gating; gating biométrico; gating + binding + step-up. |
| Decisión | Gating biométrico + device binding + rotation + step-up en operaciones sensibles. |
| Consecuencias | Depende de biometría/PIN; más lógica de session management. |
| Riesgos aceptados | Dispositivos sin biometría usan PIN (menos fuerte); dispositivos compartidos no recomendados. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Abandono por fricción | ↓ vs baseline (checkout) | Alertar si no mejora | Mejora conversión |
| ATO por remember me | ↓ ≥ 50% | Alerta si no baja tras rollout | Reducción de fraude |
| Reintentos biometría/PIN | ≤ 1 | Alerta si > 1 | UX fluida |
| Uso de remember con biometría | > 80% de dispositivos compatibles | Alerta si adopción baja | Cobertura de seguridad |
| Reuse de token/fingerprint mismatch | < 0.5% | Alerta si ≥ 0.5% | Detección temprana de replay |
| Tickets “sesión perdida” | ↓ 20% | Alertar si no baja | Soporte controlado |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-remember-me"></a>Remember Me | Opción de sesión persistente sin pedir credenciales en cada inicio. |
| <a id="term-refresh-rotation"></a>Refresh Token Rotation | Generar un refresh nuevo en cada uso e invalidar el anterior. |
| <a id="term-device-binding"></a>Device Binding | Asociar tokens a fingerprint del dispositivo para evitar replay en otro hardware. |
| <a id="term-sliding"></a>Sliding Session | Sesión que extiende expiración con actividad; usada en modelos menos estrictos. |
| <a id="term-remote-logout"></a>Remote Logout | Capacidad de invalidar tokens y limpiar sesión vía push desde backend. |
| <a id="term-tacs"></a>TACs | Criterios de aceptación técnicos para validar controles de seguridad. |

---

## Referencias

- [OWASP Mobile Security - Authentication](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Android BiometricPrompt](https://developer.android.com/reference/android/hardware/biometrics/BiometricPrompt)
- [iOS LocalAuthentication](https://developer.apple.com/documentation/localauthentication)
- [Baymard Institute - Cart Abandonment Statistics](https://baymard.com/lists/cart-abandonment-rate)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
- [ACFE Fraud Report 2023](https://acfepublic.s3-us-west-2.amazonaws.com/2023-Report-to-the-Nations.pdf)

---

*Anterior: [Session Hijacking](caso-06-session-hijacking-wifi.md) | Siguiente: [MFA SIM Swapping](caso-08-mfa-sim-swapping.md)*
