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

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | HTTPS estándar con Dio, confiar en CAs del sistema | **INADECUADO:** Vulnerable a cualquier CA comprometida. Proxy (Charles, mitmproxy) + certificado instalado ve todo en texto plano. |
| **ACEPTABLE** | Certificate pinning con `SecurityContext` y certificado en assets | **CUMPLE MÍNIMOS:** Bloquea CAs desconocidas. PERO: si certificado expira, app muere. Sin actualización dinámica. |
| **ENTERPRISE** | **Multi-layer pinning:** 1) Pin de Public Key (no certificado), 2) Backup pins para rollover, 3) Fallback con alertas, 4) Actualización dinámica via config firmada, 5) Detección de proxy | **ÓPTIMO PARA BANCA:** Resistente a rotación de certs, detecta intercepción, alerta al SOC, cumple PCI-DSS. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Bloquear certificados no autorizados incluso de CAs "confiables". Sobrevivir rotación de certs con pin de Public Key (SPKI). Detectar proxies de debug. Actualizar pins sin release via config firmada. Backup pins para rollover. |
| **Restricciones Duras (NO permite)** | **Debugging:** Certificate pinning dificulta debug (deshabilitar en debug builds). **Proxies corporativos:** Usuarios en redes con proxy SSL legítimo serán bloqueados. **Expiración:** Si todos los pins expiran, app inoperable. **iOS ATS:** Puede conflictuar. |
| **Criterio de Selección** | Se usa **Public Key pinning** sobre certificate pinning porque las claves públicas sobreviven renovaciones si se usa misma key pair. Se usa **BLoC** para TransferBloc porque transferencias tienen flujo complejo con múltiples eventos y necesitan auditabilidad. |

---

## 4. Manos a la Obra: Estrategia de Implementación

### Justificación del Plan

La estrategia se deriva del análisis de DigiNotar y los requisitos de PCI-DSS:

1. **CAs pueden ser comprometidas** → No confiar solo en trust store del sistema
2. **Certificados rotan periódicamente** → Usar Public Key pinning que sobrevive rotaciones
3. **Necesidad de actualizar pins sin release** → Config remota firmada
4. **Detectar ataques activamente** → Logging de violaciones de pinning al SOC

Se implementa en capas para máxima resiliencia: pins hardcodeados como baseline, config remota para actualizaciones, detección de proxy para alertas.

---

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

---

### Fase 2: Implementación por Plataforma

#### 2.1 Flutter (Cross-Platform) — Capa de Aplicación

**PinningConfig:**
- Lista de pins hardcodeados como baseline (SHA-256 de SPKI)
- Obtener pins con: `openssl s_client -connect api.bank.com:443 | openssl x509 -pubkey -noout | openssl pkey -pubin -outform der | openssl dgst -sha256 -binary | base64`
- Lista de dominios a pinear
- `maxPinAge`: Tiempo máximo sin validar pins remotos (30 días)

**CertificatePinner:**
- `validateCertificate(X509Certificate cert, String host)`: Validar contra pins
- `_shouldPinDomain(host)`: Solo validar dominios configurados
- `_extractPublicKeyHash(cert)`: Extraer SPKI, calcular SHA-256, encodear Base64
- `_logPinningViolation`: Log para análisis forense

**SSLPinningInterceptor (Dio):**
- En `onRequest`: Detectar proxy, bloquear endpoints críticos si detectado
- En `onError`: Detectar errores SSL/TLS, transformar a mensaje amigable
- Configurar `IOHttpClientAdapter` con `badCertificateCallback`

**RemotePinConfig:**
- Obtener pins de servidor con config firmada
- Verificar firma ECDSA/RSA con clave pública hardcodeada
- Cache en secure storage con TTL
- Fallback a pins hardcodeados si falla red

---

#### 2.2 Android — Configuración Nativa

**Network Security Config (res/xml/network_security_config.xml):**
- Configurar pins nativos como capa adicional
- Especificar dominios y sus pins
- Configurar expiration dates para forzar actualizaciones

**AndroidManifest.xml:**
```
android:networkSecurityConfig="@xml/network_security_config"
```

**ProxyDetector Android:**
- Verificar `System.getProperty("http.proxyHost")`
- Verificar `ConnectivityManager` para VPN activa
- Verificar `WifiManager` para proxy configurado en WiFi

**Consideraciones Android:**
- Network Security Config funciona desde API 24+
- Para API < 24, implementar pinning en código Dart
- `cleartextTrafficPermitted="false"` para forzar HTTPS

---

#### 2.3 iOS — Configuración Nativa

**Info.plist - App Transport Security:**
```
NSAppTransportSecurity:
  NSPinnedDomains:
    api.bank.com:
      NSIncludesSubdomains: true
      NSPinnedCAIdentities:
        - SPKI-SHA256-BASE64: "..."
```

**ProxyDetector iOS:**
- Verificar `CFNetworkCopySystemProxySettings()`
- Detectar VPN via `NEVPNManager`
- Verificar proxy en WiFi settings

**Consideraciones iOS:**
- ATS pinning disponible desde iOS 14+
- Para iOS < 14, implementar pinning en código Dart
- `NSAllowsArbitraryLoads` debe ser `false` en producción

---

### Fase 3: Observability — Métricas y Alertas

**Métricas Críticas:**
- `ssl.pinning.success` / `ssl.pinning.failure`
- `ssl.proxy.detected`
- `ssl.cert.bad`
- `ssl.config.tampered`

**Alertas:**

| Evento | Severidad | Acción |
|:-------|:----------|:-------|
| `pinningFailure` con pin desconocido | P1 Critical | Despertar equipo seguridad |
| `proxyDetected` en endpoints críticos | P2 High | Investigar horario laboral |
| `configTampered` | P1 Critical | Posible ataque infraestructura |

---

## 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

### TACs Flutter (Cross-Platform)

```
[ ] TAC-3.1-FLUTTER: DEBE implementarse Public Key Pinning para todos los
    endpoints críticos (auth, transfers, payments).

[ ] TAC-3.2-FLUTTER: DEBE mantener mínimo 2 backup pins para soportar rotación.

[ ] TAC-3.3-FLUTTER: Ante fallo de pinning, conexión DEBE rechazarse.
    NO fallback a conexión no pineada.

[ ] TAC-3.4-FLUTTER: Cada fallo de pinning DEBE generar alerta con: host,
    pin recibido, pins esperados, timestamp, device_id.

[ ] TAC-3.5-FLUTTER: DEBE existir mecanismo de actualización de pins via
    config remota firmada criptográficamente.

[ ] TAC-3.6-FLUTTER: En builds DEBUG, pinning puede deshabilitarse.
    En RELEASE, DEBE estar siempre activo.
```

### TACs Android

```
[ ] TAC-3.7-ANDROID: DEBE configurarse Network Security Config con pins
    para todos los dominios críticos (API 24+).

[ ] TAC-3.8-ANDROID: AndroidManifest DEBE incluir
    android:networkSecurityConfig="@xml/network_security_config".

[ ] TAC-3.9-ANDROID: ProxyDetector DEBE verificar http.proxyHost,
    ConnectivityManager VPN, y WifiManager proxy.

[ ] TAC-3.10-ANDROID: cleartextTrafficPermitted DEBE ser false en producción.
```

### TACs iOS

```
[ ] TAC-3.11-IOS: DEBE configurarse NSPinnedDomains en Info.plist para
    dominios críticos (iOS 14+).

[ ] TAC-3.12-IOS: NSAllowsArbitraryLoads DEBE ser false en producción.

[ ] TAC-3.13-IOS: ProxyDetector DEBE verificar CFNetworkCopySystemProxySettings
    y NEVPNManager.

[ ] TAC-3.14-IOS: Para iOS < 14, DEBE implementarse pinning equivalente en Dart.
```

### TACs Backend (Referencia)

```
[ ] TAC-3.15-BACKEND: DEBE existir endpoint para config de pins firmada
    criptográficamente.

[ ] TAC-3.16-BACKEND: Pins hardcodeados DEBEN rotarse mínimo cada 6 meses.

[ ] TAC-3.17-BACKEND: DEBE existir runbook para rotación de emergencia.
```

---

## 6. Estrategia de Pruebas (Shift-Left)

### Stack de Testing
- **Unit:** `flutter_test`, `mocktail` para CertificatePinner
- **Integration:** `integration_test` con servidor mock
- **Security:** Pruebas manuales con mitmproxy, Charles Proxy, Burp Suite

### Escenarios Críticos Obligatorios

| # | Escenario | Qué Validar | Plataforma | Tipo |
|:-:|:----------|:------------|:-----------|:-----|
| 1 | **MITM con Charles Proxy** | Requests a endpoints pineados DEBEN fallar. | Android + iOS | Security (Manual) |
| 2 | **Rotación de certificado** | App DEBE seguir funcionando con backup pin. | Flutter | Integration |
| 3 | **Todos los pins expirados** | Error claro, alerta crítica, NO conectar sin pinning. | Flutter | Unit + Integration |
| 4 | **Network Security Config** | Verificar que pins de XML se aplican correctamente. | Android | Security (Manual) |
| 5 | **ATS Pinning** | Verificar que NSPinnedDomains funciona. | iOS | Security (Manual) |

---

## Referencias

- [OWASP Certificate Pinning Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Pinning_Cheat_Sheet.html)
- [Android Network Security Config](https://developer.android.com/training/articles/security-config)
- [iOS App Transport Security](https://developer.apple.com/documentation/bundleresources/information_property_list/nsapptransportsecurity)
- [PCI-DSS Requirement 4.1](https://www.pcisecuritystandards.org/)

---

*Anterior: [Biometría Falsificada](caso-02-biometria-falsificada.md) | Siguiente: [Root/Jailbreak Detection](caso-04-root-jailbreak-detection.md)*
