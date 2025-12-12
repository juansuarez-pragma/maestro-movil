# Caso 1: El Token que Nunca Expira
## Cómo una Sesión Zombie Costó $2.4M en Fraude

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | refresh token, sesión persistente, fraude bancario, logout automático, robo de credenciales |
| **Patrón Técnico** | Token Rotation, Sliding Session, Secure Token Storage |
| **Stack Seleccionado** | flutter_secure_storage + Dio Interceptors + BLoC (AuthBloc) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario de banca móvil, quiero mantener mi sesión iniciada para no tener que autenticarme cada vez que abro la app."*

Esta historia de usuario, aparentemente inocente, esconde uno de los vectores de ataque más explotados en aplicaciones financieras: la gestión inadecuada del ciclo de vida de tokens.

### Evidencia de Industria

**Caso Revolut 2022:** En septiembre de 2022, Revolut sufrió una brecha que expuso datos de 50,000+ usuarios. El ataque de ingeniería social fue posible parcialmente porque tokens de sesión comprometidos no fueron invalidados oportunamente. La compañía tardó horas en detectar sesiones anómalas porque carecían de mecanismos robustos de rotación y revocación.

**Caso Capital One 2019:** Aunque el vector principal fue una misconfiguration de WAF, la investigación reveló que tokens de servicio tenían lifetimes excesivamente largos, ampliando la ventana de explotación.

**Estadística crítica:** Según el reporte de Verizon DBIR 2023, el 74% de las brechas financieras involucran credenciales comprometidas, y el tiempo promedio de detección de sesiones fraudulentas es de 207 días.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Fraude directo ($2.4M promedio en incidentes de session hijacking según IBM Cost of Data Breach 2023), multas regulatorias (hasta 4% de revenue global bajo GDPR, sanciones de CNBV/Banco Central) |
| **Reputacional** | Pérdida de confianza, churn de clientes (estudios muestran 65% de usuarios abandonan servicios financieros tras brecha de seguridad) |
| **Técnico** | Deuda de seguridad acumulada, necesidad de reescritura completa del módulo de autenticación, auditorías SOC 2 fallidas |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | `SharedPreferences` para guardar el token, sin expiración, refresh manual cuando falla una request | **INADECUADO:** Tokens en texto plano accesibles via backup de Android, sin rotación automática, vulnerable a replay attacks. No detecta dispositivos rooteados. Un atacante con acceso físico al dispositivo extrae el token en 30 segundos con `adb backup`. |
| **ACEPTABLE** | `flutter_secure_storage` + token con TTL de 15 min + refresh token con TTL de 7 días + interceptor Dio para auto-refresh | **CUMPLE MÍNIMOS:** Cifrado AES-256 en Keychain/Keystore, automatización del refresh, pero carece de: rotación de refresh tokens, detección de anomalías, logout remoto. Un token robado tiene ventana de 7 días. |
| **ENTERPRISE** | `flutter_secure_storage` + **Token Rotation** (nuevo refresh token en cada uso) + **BLoC para estado de auth** + Device Binding + Anomaly Detection + Remote Revocation via FCM | **ÓPTIMO PARA BANCA:** Cada refresh genera nuevo refresh token e invalida el anterior. Vinculación dispositivo-token. Detección de uso simultáneo en múltiples dispositivos. Capacidad de logout remoto instantáneo. Cumple PSD2/SCA. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

**Esta sección describe la solución ENTERPRISE**: rotación single-use de refresh, device binding y revocación remota para minimizar la ventana de explotación y ofrecer trazabilidad forense.

### 3.1 Capacidades clave
- Almacenamiento cifrado con HSM cuando está disponible, excluido de backups.
- Refresh automático single-use (token rotation) orquestado por interceptor + BLoC auditables.
- Binding de sesión a fingerprint de dispositivo (ANDROID_ID + modelo + instalación → hash SHA-256) para evitar replay cross-device.
- Logout remoto vía push (FCM/APNS) con limpieza inmediata de storage y cierre de familia de tokens (`token_family_id`).
- Registro estructurado de eventos (login, refresh, token_reuse, remote_logout) para detección de anomalías.

### 3.2 Límites y mitigaciones

| Límite / Riesgo | Mitigación |
|:----------------|:-----------|
| Sin conectividad no se puede verificar revocación | TTL corto para access (<15m) y refresh (≤7d); bloqueo de operaciones sensibles offline |
| Android < API 23 sin HSM | Cifrado software; monitorear porcentaje de usuarios legacy y plan de desuso |
| Keychain `first_unlock_this_device` no disponible antes del primer unlock | Posponer operaciones críticas hasta primer unlock o reintentar con UX clara |
| Dispositivos rooteados/jailbreak | Bloquear operaciones sensibles al detectar root/JB; alertar seguridad |
| Reuso de refresh robado | Token rotation single-use + invalidación de toda la familia ante reuse |

### 3.3 Dependencias y supuestos
- Backend debe emitir `token_family_id` y soportar invalidación de familia completa.
- Interceptor controla concurrencia de refresh (cola) y expone eventos a BLoC para auditoría.
- Politica UX: si remote logout llega, se fuerza limpieza de storage y se muestra motivo claro.

### 3.4 Decisiones arquitectónicas (vs alternativas)
- **flutter_secure_storage** sobre Hive/SharedPreferences: evita gestionar claves maestras y cumple cifrado en reposo exigido por banca.
- **Token Rotation** sobre sliding session simple: reduce impacto de robo y permite revocación granular.
- **BLoC** sobre Provider/Riverpod en auth: trazabilidad de eventos y estados para auditoría y observabilidad.
- **Device Binding** sobre binding por IP/geo: más estable para redes móviles y protege contra replays cross-device.

## Referencias

- [OWASP Mobile Security Testing Guide - Session Management](https://owasp.org/www-project-mobile-security-testing-guide/)
- [RFC 6749 - OAuth 2.0](https://tools.ietf.org/html/rfc6749)
- [Auth0 - Refresh Token Rotation](https://auth0.com/docs/secure/tokens/refresh-tokens/refresh-token-rotation)
- [Android Keystore System](https://developer.android.com/training/articles/keystore)
- [iOS Keychain Services](https://developer.apple.com/documentation/security/keychain_services)
