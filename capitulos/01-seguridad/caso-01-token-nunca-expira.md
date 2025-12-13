# Caso 1: El Token que Nunca Expira
## Cómo una Sesión Zombie Costó $2.4M en Fraude

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | refresh token, sesión persistente, fraude bancario, logout automático, robo de credenciales |
| **Patrón Técnico** | <a href="#glosario-de-terminos-clave" title="Refresh nuevo en cada uso e invalida el anterior">[Token Rotation](#term-token-rotation "Emisión de un nuevo refresh token en cada uso e invalidación del anterior para evitar reuso.")</a>, Sliding Session, Secure Token Storage |
| **Stack Seleccionado** | flutter_secure_storage + Dio Interceptors + BLoC (AuthBloc) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Tokens de acceso y refresh con [TTL](#term-ttl "Tiempo de vida del token; en esta solución: access <15m, refresh ≤7d.") extendido y sin rotación generaron sesiones persistentes (“sesiones zombis”) imposibles de revocar de forma selectiva.
- Falta de device binding y de revocación remota impidió cortar uso indebido de credenciales comprometidas, elevando riesgo de fraude y exposición regulatoria.

### Escenario de negocio que originó el problema
> *"Como usuario de banca móvil, quiero mantener mi sesión iniciada para no tener que autenticarme cada vez que abro la app."*

**Incidente detonante (resumen):**
- Usuario pierde su teléfono; el atacante mantiene la sesión porque el token no expira ni rota.
- Se ejecutan transferencias fraudulentas durante la noche (> $2.4M).
- Sin trazabilidad ni rotación, el banco no puede auditar ni cerrar la “sesión zombi”, disparando reclamos, churn y riesgo PSD2/SCA.
- Requerimiento de negocio: conservar la conveniencia (sesión persistente) pero con controles que eliminen esta ventana de fraude.

### Incidentes reportados

**Revolut 2022:** brecha que expuso datos de 50,000+ usuarios. Tokens comprometidos no se invalidaron a tiempo; el equipo tardó horas en detectar sesiones anómalas por falta de rotación/revocación. Impacto: exfiltración de datos, reclamos masivos.

**Capital One 2019:** misconfiguración de WAF y tokens de servicio con lifetimes largos ampliaron la ventana de explotación. Impacto: mayor superficie de ataque y exposición de datos.

**Dato crítico (Verizon DBIR 2023):** 74% de brechas financieras involucran credenciales comprometidas; tiempo promedio de detección de sesiones fraudulentas: 207 días.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Estudio académico UCI (2021) | 50 apps de banca global | 36% mantenían sesiones > 30 días; 12% sin rotación de refresh tras login. |
| PentestPartners (2022) | 12 apps bancarias UK/EU | 42% tenían refresh tokens sin expiración efectiva; 25% permitían reuse tras logout. |
| NowSecure State of Mobile App Security (2024) | 1,000+ apps iOS/Android (US/EU) | 85% fallan ≥1 control MASVS; gestión de sesiones aparece en el top 3 de fallos recurrentes. |
| Kaspersky Mobile Threats (2023) | LATAM y APAC | LATAM y APAC concentran mayor actividad de malware bancario (SharkBot/Anubis) enfocado en robo de tokens/cookies. |

**Resumen global de hallazgos (fuentes arriba)**
- Entre 30% y 40% de apps bancarias evaluadas presentan sesiones largas o sin rotación de refresh (UCI 2021, PentestPartners 2022).
- Al menos 25% permiten reuse de refresh tras logout, creando “sesiones zombi” (PentestPartners 2022).
- 85% de apps móviles analizadas fallan en ≥1 control MASVS; sesión/gestión de tokens es uno de los 3 fallos más comunes (NowSecure 2024).
- LATAM y APAC muestran mayor actividad de malware de robo de tokens (SharkBot/Anubis), elevando el riesgo regional (Kaspersky 2023).

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
| **ACEPTABLE** | `flutter_secure_storage` + token con <a href="#glosario-de-terminos-clave" title="Tiempo de vida del token">TTL</a> de 15 min + refresh token con TTL de 7 días + interceptor Dio para auto-refresh | **CUMPLE MÍNIMOS:** Cifrado AES-256 en Keychain/Keystore, automatización del refresh, pero carece de: rotación de refresh tokens, detección de anomalías, logout remoto. Un token robado tiene ventana de 7 días. |
| **ENTERPRISE** | `flutter_secure_storage` + **<a href="#glosario-de-terminos-clave" title="Refresh nuevo en cada uso e invalida el anterior">Token Rotation</a>** (nuevo refresh token en cada uso) + **BLoC para estado de auth** + <a href="#glosario-de-terminos-clave" title="Asociación de sesión a fingerprint de dispositivo">[Device Binding](#term-device-binding "Asociación del token a un fingerprint de dispositivo (ANDROID_ID + modelo + instalación hasheados).")</a> + Anomaly Detection + <a href="#glosario-de-terminos-clave" title="Invalidación de sesiones activas desde backend via push">Remote Revocation</a> via FCM | **ÓPTIMO PARA BANCA:** Cada refresh genera nuevo refresh token e invalida el anterior. Vinculación dispositivo-token. Detección de uso simultáneo en múltiples dispositivos. Capacidad de logout remoto instantáneo. Cumple PSD2/SCA. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

**Esta sección describe la solución ENTERPRISE**: <a href="#glosario-de-terminos-clave" title="Emite un refresh nuevo en cada uso e invalida el anterior">Token Rotation</a> single-use, <a href="#glosario-de-terminos-clave" title="Asocia la sesión a un fingerprint del dispositivo">Device Binding</a> y <a href="#glosario-de-terminos-clave" title="Invalidación de sesiones activas desde backend via push">[Revocación Remota](#term-revocacion-remota "Invalida sesiones activas desde backend y notifica por push para limpiar storage local.")</a> para minimizar la ventana de explotación y ofrecer trazabilidad forense.

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Interceptor serializa refresh; BLoC sin carreras | Equipo móvil, CI |
| Integration (CI) | Respuesta `token_family_revoked` → limpiar storage, login; reintentos ≤ 1 | Equipo móvil/backend, CI |
| Seguridad manual | Extracción backup con `allowBackup=false`; root/JB bloquea operaciones sensibles | Seguridad/QE, dispositivos reales |
| Observabilidad (CI) | Eventos `auth.*` con atributos (trace_id, device_hash, family_id, app_version) | Equipo móvil, CI |

### 3.2 Capacidades clave
- Almacenamiento cifrado con <a href="#glosario-de-terminos-clave" title="Hardware Security Module, cifrado respaldado por hardware">[HSM](#term-hsm "Hardware Security Module; provee cifrado respaldado por hardware (Keystore/Keychain).")</a> cuando está disponible, excluido de backups.
- Refresh automático single-use (<a href="#glosario-de-terminos-clave" title="Refresh nuevo en cada uso, invalida el anterior">Token Rotation</a>) orquestado por interceptor + BLoC auditables.
- Binding de sesión a fingerprint de dispositivo (ANDROID_ID + modelo + instalación → hash SHA-256) para evitar replay cross-device.
- Logout remoto vía push (FCM/APNS) con limpieza inmediata de storage y cierre de familia de tokens (<a href="#glosario-de-terminos-clave" title="Identificador que agrupa la cadena de refresh">token_family_id</a>).
- Registro estructurado de eventos (login, refresh, token_reuse, remote_logout) para detección de anomalías.

### 3.3 Límites y mitigaciones

| Límite / Riesgo | Mitigación |
|:----------------|:-----------|
| Sin conectividad no se puede verificar revocación | <a href="#glosario-de-terminos-clave" title="Tiempo de vida del token">TTL</a> corto para access (<15m) y refresh (≤7d); bloqueo de operaciones sensibles offline |
| Android < API 23 sin <a href="#glosario-de-terminos-clave" title="Hardware Security Module, cifrado respaldado por hardware">HSM</a> | Cifrado software; monitorear porcentaje de usuarios legacy y plan de desuso |
| Keychain `first_unlock_this_device` no disponible antes del primer unlock | Posponer operaciones críticas hasta primer unlock o reintentar con UX clara |
| Dispositivos rooteados/jailbreak | Bloquear operaciones sensibles al detectar <a href="#glosario-de-terminos-clave" title="Chequeos de integridad y permisos elevados">Root/JB</a>; alertar seguridad |
| Reuso de refresh robado | <a href="#glosario-de-terminos-clave" title="Refresh nuevo en cada uso e invalida el anterior">Token Rotation</a> single-use + invalidación de toda la familia ante reuse |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Tokens comprometidos con ventana larga y sin detección de reuse. |
| Opciones evaluadas | Sliding sessions sin rotación; rotation single-use con device binding; sesiones cortas sin refresh. |
| Decisión | Rotation single-use + device binding + revocación remota + BLoC auditable. |
| Consecuencias | Complejidad en backend (familia/invalidación); UX depende de conectividad para revocación. |
| Riesgos aceptados | Funcionalidad limitada offline; dependencia de push para logout remoto. |

### 3.5 Dependencias y supuestos
- Backend debe emitir <a href="#glosario-de-terminos-clave" title="Identificador que agrupa la cadena de refresh">token_family_id</a> y soportar invalidación de familia completa.
- Interceptor controla concurrencia de refresh (cola) y expone eventos a BLoC para auditoría.
- Politica UX: si remote logout llega, se fuerza limpieza de storage y se muestra motivo claro.

### 3.6 Decisiones arquitectónicas (vs alternativas)

| Opción elegida | Por qué se eligió | Alternativa descartada | Por qué no |
|:---------------|:------------------|:-----------------------|:-----------|
| flutter_secure_storage (Keychain/Keystore) | Cifrado at-rest respaldado por hardware; evita gestionar claves maestras; alineado con [OWASP MASVS](https://owasp.org/www-project-mobile-security-testing-guide/) y cumplimiento regulatorio. | Hive/SharedPreferences con cifrado propio | Requiere manejar key de cifrado (chicken-egg), mayor superficie de error, sin <a href="#glosario-de-terminos-clave" title="Hardware Security Module, cifrado respaldado por hardware">HSM</a>. |
| <a href="#glosario-de-terminos-clave" title="Refresh nuevo en cada uso e invalida el anterior">Token Rotation</a> (refresh single-use) | Cierra ventana de reuso; permite invalidar familias completas ante reuse, recomendado por [Auth0 Rotation](https://auth0.com/docs/secure/tokens/refresh-tokens/refresh-token-rotation). | Sliding session sin rotation | Refresh robado sigue siendo válido hasta expirar; no detecta reuse. |
| BLoC para auth | Estados y eventos auditables; fácil instrumentación con logging/tracing; separación clara de UI/negocio. | Provider/Riverpod para auth | Menor trazabilidad de transiciones; patrones más acoplados a UI. |
| <a href="#glosario-de-terminos-clave" title="Asociación de sesión a fingerprint de dispositivo">Device Binding</a> (fingerprint hash) | Bloquea replay cross-device; combina identificadores estables del dispositivo. | Binding por IP/Geo | IP móvil es volátil, genera falsos positivos; no previene replay en mismo país/ISP. |
| <a href="#glosario-de-terminos-clave" title="Serializa refreshes para evitar concurrencia">[Queued Interceptor](#term-queued-interceptor "Patrón para serializar refresh de tokens y evitar múltiples refreshes concurrentes.")</a> para refresh | Serializa refresh y evita storms de refresh concurrentes; reduce condiciones de carrera. | Refresh ad-hoc por request | Multiplica llamadas /refresh y riesgo de inconsistencias en storage. |

### 3.7 NFRs y objetivos medibles
| Categoría | Objetivo | Cómo se mide |
|:----------|:---------|:-------------|
| Seguridad | 0 incidentes de token reuse/día | Métrica `auth.security.token_reuse` + alerta crítica inmediata |
| Rendimiento | p95 `/auth/refresh` < 500 ms (warning ≥ 650 ms, crítica ≥ 800 ms) | APM backend + trazas en interceptor |
| Confiabilidad | Éxito refresh > 99.5% rolling 7d | Métrica `auth.token.refresh_success_rate` con alerta si baja de 99.5% 5 min |
| UX | Reintentos de refresh ≤ 1 antes de pedir login | Contador de reintentos en interceptor; alerta si promedio > 1 |

### 3.8 Contratos de API (backend)
| Endpoint | Request | Response | Errores/Notas |
|:---------|:--------|:---------|:--------------|
| `POST /auth/login` | `{username, password, device_fingerprint}` | `{access_token, refresh_token, token_family_id, expires_in}` | 401 credenciales; 423 dispositivo bloqueado |
| `POST /auth/refresh` | `{refresh_token, device_fingerprint}` | `{new_access_token, new_refresh_token, token_family_id, expires_in}` | 401 inválido; 403 `token_family_revoked`; 429 rate-limit; rate-limit sugerido 5 req/min y `Retry-After` |

### 3.9 Diagrama de flujo (texto)
```
App -> Interceptor: Request
Interceptor: ¿access expira en <30s? sí -> refrescar
Interceptor -> Backend /auth/refresh {refresh_token, fingerprint}
Backend: valida familia, invalida refresh anterior, emite nuevo par
Backend -> Interceptor: {access, refresh}
Interceptor: reintenta request original
Backend -> FCM/APNS: push de logout remoto
App -> Interceptor: recibe push, limpia storage y fuerza AuthLogout
```

### 3.10 Riesgo → Control → Métrica
| Riesgo | Control | Métrica/Alerta |
|:-------|:--------|:---------------|
| Refresh robado reutilizado | Rotation single-use + invalidar familia | `auth.security.token_reuse` alerta crítica |
| Refresh storm concurrente | Queued Interceptor | Número de refresh concurrentes (esperado 1) |
| Root/JB extracción | Detección root/JB y bloqueo | Evento `auth.security.root_detected` |
| Revocación tardía | TTL corto y push de logout | `auth.session.expired` p95 < 15m |

### 3.11 UX y política offline — tabla rápida
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Operaciones sin conectividad | Bloquear operaciones sensibles; mostrar motivo y CTA para reconectar | Evita ejecución sin revocación posible |
| Skew de reloj | Tolerancia ±2 min; si excede, guiar a sincronizar hora | Previene falsos 401/expiraciones |
| Reintentos de refresh | Un reintento por error de red; luego pedir login | Limita loops y mejora trazabilidad |

### 3.12 Notas de seguridad TLS y claves — tabla sintética
| Tema | Requisito | Notas |
|:-----|:----------|:------|
| Pinning | 2–3 pines SPKI activos, rotación cada 90 días, fail-closed salvo ventana planificada | Soportar doble pin en cliente |
| Skew de reloj | ±2 min en expiración/validación | Evita falsos 401 |
| Claves backend | JWKS con `kid`, RS256/ES256, rotación con solapamiento | Revocar familias asociadas a claves comprometidas |

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Reuse de token | 0 incidentes/día | Crítica si > 0 | Fraude contenido y cumplimiento regulatorio |
| `/auth/refresh` p95 | < 500 ms | Warning ≥ 650 ms; crítica ≥ 800 ms | UX fluida y sin timeouts |
| Éxito de refresh | > 99.5% (rolling 7d) | Alerta si < 99.5% por 5 min | Sesiones confiables y estables |
| Tickets “sesión perdida” | ↓ 30% en 4 semanas | Alertar si no baja tras el release | Menos soporte y mejor NPS |
| Sesiones activas > 30d | < 5% | Alerta si ≥ 5% | Ventana de exposición acotada |
| Reuse de refresh por device_id | < 0.5% | Alerta si ≥ 0.5% | Detección temprana de reuse |
| Incidentes con token válido (IP/ASN anómala) | < 5 por 1M sesiones | Alerta si ≥ 5 | Anomalías contenidas |
| Device swap con mismo token | < 0.1% | Alerta si ≥ 0.1% | Replay cross-device mitigado |
| Duración de sesión | p95 < 24h; p99 < 7d | Alerta si supera metas | Ventana de ataque limitada |

<a id="glosario-de-terminos-clave"></a>
## Glosario de Términos Clave

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-token-rotation"></a><a href="#glosario-de-terminos-clave" title="Refresh nuevo en cada uso e invalida el anterior">Token Rotation</a> | Emisión de un nuevo refresh token en cada uso e invalidación del anterior para evitar reuso. |
| <a id="term-token-family"></a><a href="#glosario-de-terminos-clave" title="Identificador que agrupa la cadena de refresh">Token Family</a> | Identificador (`token_family_id`) que agrupa la cadena de refresh; permite revocar toda la familia ante reuse. |
| <a id="term-device-binding"></a><a href="#glosario-de-terminos-clave" title="Asociación de token a fingerprint de dispositivo">Device Binding</a> | Asociación del token a un fingerprint de dispositivo (ANDROID_ID + modelo + instalación hasheados). |
| <a id="term-refresh-token-single-use"></a><a href="#glosario-de-terminos-clave" title="Refresh se consume una sola vez">Refresh Token Single-Use</a> | Política que marca un refresh token como consumido tras usarse una vez. |
| <a id="term-revocacion-remota"></a><a href="#glosario-de-terminos-clave" title="Invalidación de sesiones activas desde backend via push">Revocación Remota</a> | Invalida sesiones activas desde backend y notifica por push para limpiar storage local. |
| <a id="term-hsm"></a><a href="#glosario-de-terminos-clave" title="Hardware Security Module">HSM</a> | Hardware Security Module; provee cifrado respaldado por hardware (Keystore/Keychain). |
| <a id="term-ttl"></a><a href="#glosario-de-terminos-clave" title="Tiempo de vida del token">TTL</a> | Tiempo de vida del token; en esta solución: access <15m, refresh ≤7d. |
| <a id="term-queued-interceptor"></a><a href="#glosario-de-terminos-clave" title="Serializa refreshes para evitar concurrencia">Queued Interceptor</a> | Patrón para serializar refresh de tokens y evitar múltiples refreshes concurrentes. |
| <a id="term-root-jailbreak-detection"></a><a href="#glosario-de-terminos-clave" title="Chequeos de integridad y permisos elevados">Root/Jailbreak Detection</a> | Controles para bloquear operaciones sensibles en dispositivos comprometidos. |

## Referencias

- [OWASP Mobile Security Testing Guide - Session Management](https://owasp.org/www-project-mobile-security-testing-guide/)
- [RFC 6749 - OAuth 2.0](https://tools.ietf.org/html/rfc6749)
- [Auth0 - Refresh Token Rotation](https://auth0.com/docs/secure/tokens/refresh-tokens/refresh-token-rotation)
- [Android Keystore System](https://developer.android.com/training/articles/keystore)
- [iOS Keychain Services](https://developer.apple.com/documentation/security/keychain_services)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
- [PentestPartners - Banking app security review 2022](https://www.pentestpartners.com/security-blog/banking-app-security-review-2022/)
- [Kaspersky - Mobile Threats 2023](https://securelist.com/mobile-malware-evolution-2023/111742/)
- [Verizon DBIR 2023](https://www.verizon.com/business/resources/reports/dbir/)
