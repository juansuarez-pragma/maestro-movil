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

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Almacenamiento cifrado con hardware security module (HSM) en dispositivos compatibles. Rotación automática de refresh tokens sin fricción de usuario. Invalidación remota de sesiones específicas o todas las sesiones. Binding de token a device fingerprint (ANDROID_ID + modelo + instalación única). Detección de jailbreak/root antes de permitir operaciones sensibles. |
| **Restricciones Duras (NO permite)** | **iOS Keychain:** Accesible después de primer unlock del dispositivo (no "always available"). **Android Keystore:** Requiere API 23+ para cifrado respaldado por hardware. **Offline:** No puede validar revocación de tokens sin conexión (requiere estrategia de TTL corto). **Backup:** flutter_secure_storage marca datos como excluded from backup pero el usuario puede tener herramientas de extracción. |
| **Criterio de Selección** | Se usa **flutter_secure_storage** sobre Hive/SharedPreferences porque: 1) Utiliza Keychain (iOS) y EncryptedSharedPreferences/Keystore (Android) respaldados por hardware cuando disponible. 2) Los tokens son datos de alta sensibilidad que requieren cifrado at-rest por regulación. 3) Hive cifrado requiere gestionar la key de cifrado (chicken-egg problem). Se usa **BLoC** sobre Riverpod/Provider para AuthState porque: auditabilidad de transiciones (login→authenticated→refreshing→error), logging de eventos para forensics, y separación clara de lógica de negocio. |

## Referencias

- [OWASP Mobile Security Testing Guide - Session Management](https://owasp.org/www-project-mobile-security-testing-guide/)
- [RFC 6749 - OAuth 2.0](https://tools.ietf.org/html/rfc6749)
- [Auth0 - Refresh Token Rotation](https://auth0.com/docs/secure/tokens/refresh-tokens/refresh-token-rotation)
- [Android Keystore System](https://developer.android.com/training/articles/keystore)
- [iOS Keychain Services](https://developer.apple.com/documentation/security/keychain_services)
