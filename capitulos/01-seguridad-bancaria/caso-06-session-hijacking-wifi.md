# Caso 6: Session Hijacking en WiFi Público
## El Caso del Café que Vació Cuentas

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | session hijacking, WiFi público, red insegura, sidejacking, seguridad de red |
| **Patrón Técnico** | Network Security Detection, Session Binding, Additional Verification on Untrusted Networks |
| **Stack Seleccionado** | connectivity_plus + Platform Channels (Network Security) + BLoC (NetworkSecurityBloc) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero poder revisar mi cuenta bancaria desde el WiFi de la cafetería mientras espero mi café."*

Redes WiFi públicas son terreno fértil para ataques. ARP spoofing, rogue access points, SSL stripping, y sniffing de tráfico son amenazas reales.

### Evidencia de Industria

**DarkHotel APT (2014-presente):** Grupo APT que comprometía WiFi de hoteles de lujo para atacar ejecutivos de empresas Fortune 500. Instalaba malware via portales cautivos falsos.

**Estudio Kaspersky 2023:** 25% de puntos WiFi públicos no tienen cifrado, otro 25% usan cifrado débil (WEP). Solo 50% usa WPA2/WPA3.

**Caso Firesheep (2010):** Herramienta que demostró lo trivial que es capturar cookies de sesión en redes no cifradas. Afectó a millones de usuarios de Facebook, Twitter.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Session tokens robados permiten Account Takeover |
| **Reputacional** | "Usuario robado en Starbucks" genera crisis de PR |
| **Legal** | Banco demandado por no advertir sobre riesgos de WiFi público |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Confiar en HTTPS, sin detección de red | **INADECUADO:** SSL stripping, proxies maliciosos, certificados fraudulentos. Usuario inconsciente del riesgo. |
| **ACEPTABLE** | Detectar WiFi público + mostrar warning al usuario | **CUMPLE MÍNIMOS:** Usuario informado. PERO: muchos ignoran warnings, no hay protección activa. |
| **ENTERPRISE** | **Defensa en Profundidad:** Detectar tipo de red, re-verificación en redes no confiables, session binding a contexto de red, VPN sugerida, operaciones limitadas | **ÓPTIMO PARA BANCA:** Protección activa, no solo advertencia. Balance seguridad vs usabilidad. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Detectar tipo de red (WiFi/Celular/VPN). Identificar WiFi segura vs insegura por nivel de cifrado. Step-up authentication en redes no confiables. Vincular sesión a contexto de red. Limitar operaciones por nivel de confianza. Sugerir VPN. |
| **Restricciones Duras (NO permite)** | **Certeza absoluta:** No puede garantizar que una red "privada" no esté comprometida. **VPN detection:** Complejo y no siempre confiable. **iOS restricciones:** APIs más limitadas para inspección de red. **Battery drain:** Monitoreo constante consume batería. |
| **Criterio de Selección** | Se usa **BLoC** porque estado de red cambia dinámicamente y múltiples features deben reaccionar (transfers, auth, visualización de datos). `connectivity_plus` proporciona detección básica, Platform Channels para info detallada de red. |

---

## 4. Manos a la Obra: Estrategia de Implementación

### Justificación del Plan

La estrategia se deriva del análisis de DarkHotel APT y Firesheep:

1. **WiFi público es territorio hostil** → Detección activa de tipo de red
2. **Usuarios ignoran warnings** → Protección activa con limitación de operaciones
3. **Contexto de red cambia** → Session binding y re-evaluación continua
4. **No podemos bloquear completamente** → Balance: permitir operaciones de bajo riesgo

La arquitectura usa BLoC para gestionar estado de red reactivamente, con Platform Channels para obtener detalles de seguridad de red específicos por plataforma.

---

### Fase 1: Diseño — Modelo de Confianza de Red

**Niveles de Confianza:**
- `trusted`: Red celular, WiFi empresarial configurada, VPN activa
- `moderate`: WiFi doméstico con WPA2/WPA3
- `untrusted`: WiFi público, WiFi abierto, red con cifrado débil
- `hostile`: Red en blacklist conocida, comportamiento anómalo detectado

**Matriz de Acciones por Nivel:**

| Operación | trusted | moderate | untrusted | hostile |
|:----------|:--------|:---------|:----------|:--------|
| Ver saldo | ✓ | ✓ | ✓ | ⚠️ Warning |
| Transferir < $100 | ✓ | ✓ | ⚠️ Re-auth | ✗ Bloquear |
| Transferir > $1000 | ✓ | ⚠️ OTP | ✗ Bloquear | ✗ Bloquear |
| Agregar beneficiario | ✓ | ⚠️ OTP | ✗ Bloquear | ✗ Bloquear |

**Estructura de Carpetas:**
```
lib/core/network_security/
├── data/
│   ├── datasources/
│   │   ├── network_info_datasource.dart
│   │   └── vpn_detection_datasource.dart
│   └── repositories/
│       └── network_security_repository_impl.dart
├── domain/
│   ├── entities/
│   │   └── network_trust_level.dart
│   └── usecases/
│       ├── evaluate_network_trust.dart
│       └── check_operation_allowed.dart
└── presentation/
    └── bloc/
        ├── network_security_bloc.dart
        ├── network_security_event.dart
        └── network_security_state.dart
```

---

### Fase 2: Implementación por Plataforma

#### 2.1 Flutter (Cross-Platform) — Capa de Aplicación

**NetworkSecurityBloc:**
- Escuchar cambios de conectividad con `connectivity_plus`
- Evaluar nivel de confianza en cada cambio de red
- Estados: `NetworkTrusted`, `NetworkModerate`, `NetworkUntrusted`, `NetworkHostile`
- Eventos: `NetworkChanged`, `VPNStatusChanged`, `NetworkEvaluationRequested`

**NetworkTrustEvaluator:**
- Obtener tipo de conexión (WiFi, Cellular, Ethernet)
- Si WiFi: obtener SSID, BSSID, tipo de seguridad
- Verificar contra whitelist de redes confiables
- Verificar si VPN está activa
- Calcular nivel de confianza compuesto

**SessionNetworkBinding:**
- Guardar fingerprint de red al crear/refrescar sesión
- Verificar consistencia en cada request crítica
- Cambio drástico (ej: país diferente en segundos) → invalidar sesión o requerir re-auth

**OperationGatekeeper:**
- Interceptor que verifica nivel de red antes de operaciones
- Determina si permitir, requerir step-up, o bloquear
- UI components para warnings y re-auth

---

#### 2.2 Android — Integración Nativa

**NetworkInfoDatasource Android (Platform Channel):**
- Usar `ConnectivityManager` para tipo de conexión
- Usar `WifiManager` para detalles de WiFi (SSID, BSSID)
- `WifiInfo.getSecurityType()` para tipo de cifrado (WPA2, WPA3, Open)
- Verificar `NetworkCapabilities` para VPN

**VPNDetection Android:**
- `ConnectivityManager.getActiveNetwork()`
- Verificar `NET_CAPABILITY_NOT_VPN` en capabilities
- Listar `NetworkInterface` para interfaces de VPN

**Consideraciones Android:**
- `ACCESS_FINE_LOCATION` requerido para SSID en API 28+
- Background location para monitoreo continuo
- Battery optimization puede afectar detección

---

#### 2.3 iOS — Integración Nativa

**NetworkInfoDatasource iOS (Platform Channel):**
- Usar `NEHotspotHelper` para info de WiFi (requiere entitlement especial)
- `CNCopyCurrentNetworkInfo()` deprecated en iOS 13+
- Alternativa: `NEHotspotConfiguration` para WiFi conocidos

**VPNDetection iOS:**
- `NEVPNManager.shared().connection.status`
- `CFNetworkCopySystemProxySettings()` para detectar configuración de proxy

**Consideraciones iOS:**
- Obtener SSID requiere Location permission y entitlement
- Apple ha limitado acceso a info de red por privacidad
- NEHotspotHelper requiere aprobación especial de Apple
- Alternativa: usar heurísticas (IP local, gateway) sin SSID

---

### Fase 3: Observability — Métricas y Alertas

**Métricas:**
- `network.trust_level` (distribución por nivel)
- `network.operation.blocked` (por nivel de red y tipo de operación)
- `network.operation.stepup_required`
- `network.session.invalidated` (por cambio de contexto)
- `network.vpn.detected`

**Alertas:**

| Evento | Severidad | Acción |
|:-------|:----------|:-------|
| Operación bloqueada en `hostile` | P3 Medium | Log para análisis |
| Cambio drástico de ubicación | P2 High | Posible session hijacking |
| Múltiples usuarios misma red `hostile` | P2 High | Investigar red |

---

## 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

### TACs Flutter (Cross-Platform)

```
[ ] TAC-6.1-FLUTTER: DEBE detectar y clasificar nivel de confianza de red
    (trusted, moderate, untrusted, hostile).

[ ] TAC-6.2-FLUTTER: En redes untrusted, operaciones > $100 DEBEN requerir
    autenticación adicional.

[ ] TAC-6.3-FLUTTER: En redes hostile, DEBE bloquearse operaciones financieras.

[ ] TAC-6.4-FLUTTER: Usuario DEBE ser informado cuando está en red no confiable.

[ ] TAC-6.5-FLUTTER: DEBE existir session binding a contexto de red.

[ ] TAC-6.6-FLUTTER: Cambio drástico de contexto (país) DEBE invalidar sesión
    o requerir re-auth.

[ ] TAC-6.7-FLUTTER: DEBE sugerir uso de VPN en redes no confiables.
```

### TACs Android

```
[ ] TAC-6.8-ANDROID: DEBE obtener tipo de seguridad de WiFi
    (WPA2, WPA3, Open, WEP).

[ ] TAC-6.9-ANDROID: DEBE detectar si VPN está activa via NetworkCapabilities.

[ ] TAC-6.10-ANDROID: DEBE solicitar permisos de ubicación para obtener SSID
    (API 28+).

[ ] TAC-6.11-ANDROID: DEBE manejar battery optimization para monitoreo de red.
```

### TACs iOS

```
[ ] TAC-6.12-IOS: DEBE detectar VPN activa via NEVPNManager.

[ ] TAC-6.13-IOS: DEBE usar heurísticas alternativas si SSID no disponible.

[ ] TAC-6.14-IOS: DEBE detectar configuración de proxy para redes sospechosas.

[ ] TAC-6.15-IOS: DEBE solicitar Location permission si se requiere info de WiFi.
```

### TACs Backend (Referencia)

```
[ ] TAC-6.16-BACKEND: DEBE recibir contexto de red con cada request crítica.

[ ] TAC-6.17-BACKEND: DEBE mantener whitelist de redes confiables por usuario.

[ ] TAC-6.18-BACKEND: Certificate pinning DEBE estar activo independiente de red.

[ ] TAC-6.19-BACKEND: Telemetría de operaciones por nivel de red para análisis.
```

---

## 6. Estrategia de Pruebas (Shift-Left)

### Stack de Testing
- **Unit:** `flutter_test`, `bloc_test` para NetworkSecurityBloc
- **Integration:** `integration_test` con diferentes tipos de red
- **Security:** Pruebas en redes controladas con herramientas de pentesting

### Escenarios Críticos Obligatorios

| # | Escenario | Qué Validar | Plataforma | Tipo |
|:-:|:----------|:------------|:-----------|:-----|
| 1 | **WiFi abierto (sin cifrado)** | Detecta untrusted, muestra warning, requiere step-up para operaciones | Flutter | Integration |
| 2 | **Cambio de red durante sesión** | Session re-evalúa confianza, notifica si baja nivel | Flutter | Integration |
| 3 | **Cambio de país abrupto** | Session invalidada o re-auth requerida | Flutter | Integration |
| 4 | **VPN activa** | Detecta VPN, eleva nivel de confianza | Android + iOS | Integration |
| 5 | **Red en blacklist** | Operaciones financieras bloqueadas | Flutter | Unit |

---

## Referencias

- [OWASP Mobile Security - Network Communication](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Android ConnectivityManager](https://developer.android.com/reference/android/net/ConnectivityManager)
- [iOS NEHotspotHelper](https://developer.apple.com/documentation/networkextension/nehotspothelper)
- [NIST Guidelines on Network Security](https://csrc.nist.gov/publications/detail/sp/800-153/final)

---

*Anterior: [Secure PIN Pad](caso-05-secure-pin-pad.md) | Siguiente: [Remember Me](caso-07-remember-me-seguro.md)*
