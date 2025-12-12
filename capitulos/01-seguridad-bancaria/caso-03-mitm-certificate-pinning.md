# Caso 3: El Man-in-the-Middle Silencioso
## Certificate Pinning en Transferencias Interbancarias

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | certificate pinning, SSL pinning, MITM attack, transferencia interbancaria, SPEI, ACH, seguridad en tránsito |
| **Patrón Técnico** | Certificate Pinning, Public Key Pinning, Trust on First Use (TOFU) |
| **Stack Seleccionado** | Dio + dio_http2_adapter + certificados hardcodeados + BLoC (TransferBloc) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero realizar transferencias interbancarias desde la app móvil con la confianza de que mis datos están protegidos en tránsito."*

El problema real: HTTPS estándar confía en cualquier Certificate Authority (CA) instalado en el dispositivo. Un atacante que instala un certificado malicioso puede interceptar todo el tráfico "cifrado".

### Evidencia de Industria

**Caso DigiNotar 2011:** Una CA holandesa fue comprometida, emitiendo certificados fraudulentos para google.com, permitiendo ataques MITM masivos en Irán.

**Operación Socialist (GCHQ):** Documentos filtrados revelaron uso de proxies MITM con certificados válidos para interceptar tráfico de apps bancarias.

**Estudio académico (2019):** El 73% de las apps bancarias Android no implementaban certificate pinning correctamente.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Transferencias fraudulentas interceptadas, robo de credenciales |
| **Regulatorio** | Incumplimiento PCI-DSS Req. 4.1 (cifrado en tránsito) |
| **Legal** | Responsabilidad del banco si no tomó medidas "razonables" |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Rol | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:----|:-----------------------|:----------------------------------|
| **Junior** | HTTPS estándar con Dio, confiar en CAs del sistema | **FALLA:** Vulnerable a cualquier CA comprometida. Proxy (Charles, mitmproxy) + certificado instalado ve todo en texto plano. |
| **Senior** | Certificate pinning con `SecurityContext` y certificado en assets | **MEJORA:** Bloquea CAs desconocidas. PERO: si certificado expira, app muere. Sin actualización dinámica. |
| **Architect** | **Multi-layer pinning:** 1) Pin de Public Key (no certificado), 2) Backup pins para rollover, 3) Fallback con alertas, 4) Actualización dinámica via config firmada, 5) Detección de proxy | **ENTERPRISE:** Resistente a rotación de certs, detecta intercepción, alerta al SOC, cumple PCI-DSS. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Bloquear certificados no autorizados incluso de CAs "confiables". Sobrevivir rotación de certs con pin de Public Key (SPKI). Detectar proxies de debug. Actualizar pins sin release via config firmada. Backup pins para rollover. |
| **Restricciones Duras (NO permite)** | **Debugging:** Certificate pinning dificulta debug (deshabilitar en debug builds). **Proxies corporativos:** Usuarios en redes con proxy SSL legítimo serán bloqueados. **Expiración:** Si todos los pins expiran, app inoperable. **iOS ATS:** Puede conflictuar. |
| **Criterio de Selección** | Se usa **Public Key pinning** sobre certificate pinning porque las claves públicas sobreviven renovaciones si se usa misma key pair. Se usa **BLoC** para TransferBloc porque transferencias tienen flujo complejo con múltiples eventos y necesitan auditabilidad. |

---

## 4. Manos a la Obra: Estrategia de Implementación

### Fase 1: Diseño — Estrategia de Pins

**Arquitectura de Pins:**
- Primary Pin: Certificado de producción actual (SHA-256 de SPKI en Base64)
- Backup Pin: Próximo certificado (pre-staged para rotación)
- Emergency Pin: Clave de DR (offline, en HSM)
- Fallback Chain: Primary → Backup → Emergency → FAIL + ALERT

**Dominios a Pinear:**
- `api.bank.com`
- `auth.bank.com`
- `transfers.bank.com`

**Estructura de Carpetas:**
```
lib/core/
├── network/
│   ├── certificate_pinner.dart
│   ├── pinning_config.dart
│   ├── ssl_pinning_interceptor.dart
│   └── proxy_detector.dart
└── security/
    └── remote_config_validator.dart
```

### Fase 2: Implementación — Detalles Técnicos

**PinningConfig:**
- Lista de pins hardcodeados como baseline (SHA-256 de SPKI)
- Obtener pins con: `openssl s_client -connect api.bank.com:443 | openssl x509 -pubkey -noout | openssl pkey -pubin -outform der | openssl dgst -sha256 -binary | base64`
- Lista de dominios a pinear
- `maxPinAge`: Tiempo máximo sin validar pins remotos (30 días)

**CertificatePinner:**
- `validateCertificate(X509Certificate cert, String host)`: Validar contra pins
- `_shouldPinDomain(host)`: Solo validar dominios configurados
- `_extractPublicKeyHash(cert)`: Extraer SPKI, calcular SHA-256, encodear Base64
- `_logPinningViolation`: Log para análisis forense (timestamp, host, receivedPin, expectedPins, cert info)

**SSLPinningInterceptor (Dio):**
- En `onRequest`: Detectar proxy con `ProxyDetector`, bloquear endpoints críticos si proxy detectado
- En `onError`: Detectar errores SSL/TLS, transformar a mensaje amigable
- Configurar `IOHttpClientAdapter` con `badCertificateCallback` que siempre rechaza

**ProxyDetector:**
- Verificar variables de entorno (`http_proxy`, `HTTP_PROXY`, `https_proxy`, `HTTPS_PROXY`)
- Verificar VPN activa (requiere platform channel)
- Método `isDebugMode()` para detectar herramientas de reversing

**RemotePinConfig:**
- Obtener pins de servidor con config firmada
- Verificar firma ECDSA/RSA con clave pública hardcodeada
- Cache en `flutter_secure_storage` con TTL
- Fallback a pins hardcodeados si falla red

### Fase 3: Observability — Métricas y Alertas

**Métricas Críticas:**
- `ssl.pinning.success` / `ssl.pinning.failure`
- `ssl.proxy.detected`
- `ssl.cert.bad`
- `ssl.config.tampered`
- `ssl.handshake.latency_ms`

**Alertas:**

| Evento | Severidad | Acción |
|:-------|:----------|:-------|
| `pinningFailure` con pin desconocido | P1 Critical | Despertar equipo seguridad |
| `proxyDetected` en endpoints críticos | P2 High | Investigar horario laboral |
| `configTampered` | P1 Critical | Posible ataque infraestructura |

---

## 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

```
[ ] TAC-3.1: DEBE implementarse Public Key Pinning para todos los endpoints
    críticos (auth, transfers, payments).

[ ] TAC-3.2: DEBE mantener mínimo 2 backup pins para soportar rotación sin
    disrupciones.

[ ] TAC-3.3: Ante fallo de pinning, conexión DEBE rechazarse.
    NO fallback a conexión no pineada.

[ ] TAC-3.4: Cada fallo de pinning DEBE generar alerta con: host, pin recibido,
    pins esperados, timestamp, device_id.

[ ] TAC-3.5: DEBE existir mecanismo de actualización de pins via config remota
    firmada criptográficamente.

[ ] TAC-3.6: DEBE detectar proxy y bloquear operaciones críticas cuando se detecte.

[ ] TAC-3.7: En builds DEBUG, pinning puede deshabilitarse. En RELEASE, DEBE
    estar siempre activo.

[ ] TAC-3.8: Pins hardcodeados DEBEN rotarse mínimo cada 6 meses.

[ ] TAC-3.9: DEBE existir runbook para rotación de emergencia de certificados.

[ ] TAC-3.10: Métricas de pinning DEBEN monitorearse con alertas automáticas.
```

---

## 6. Estrategia de Pruebas (Shift-Left)

### Stack de Testing
- **Unit:** `flutter_test`, `mocktail` para CertificatePinner
- **Integration:** `integration_test` con servidor mock
- **Security:** Pruebas manuales con mitmproxy, Charles Proxy, Burp Suite

### Escenarios Críticos Obligatorios

| # | Escenario | Qué Validar | Tipo |
|:-:|:----------|:------------|:-----|
| 1 | **MITM con Charles Proxy** | Configurar Charles, instalar certificado. Requests a endpoints pineados DEBEN fallar. | Security (Manual) |
| 2 | **Rotación de certificado** | Agregar nuevo pin a backup, rotar cert en servidor. App DEBE seguir funcionando con backup pin. | Integration |
| 3 | **Todos los pins expirados** | Ningún pin válido. App DEBE mostrar error claro, enviar alerta crítica, NO conectar sin pinning. | Unit + Integration |

---

## Referencias

- [OWASP Certificate Pinning Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Pinning_Cheat_Sheet.html)
- [RFC 7469 - HTTP Public Key Pinning (HPKP)](https://tools.ietf.org/html/rfc7469)
- [PCI-DSS Requirement 4.1](https://www.pcisecuritystandards.org/)

---

*Anterior: [Biometría Falsificada](caso-02-biometria-falsificada.md) | Siguiente: [Root/Jailbreak Detection](caso-04-root-jailbreak-detection.md)*
