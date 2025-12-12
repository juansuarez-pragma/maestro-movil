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

### Problema detectado (técnico)
- Redes no confiables permiten robo de tokens/sesiones (sidejacking) vía ARP spoofing, rogue AP o SSL stripping si no hay [pinning](caso-03-mitm-certificate-pinning.md#glosario-de-terminos-clave).
- Sin binding de sesión a contexto de red, el token robado se reutiliza desde otra red sin fricción.
- Solo mostrar un warning es insuficiente: los usuarios lo ignoran; se necesita protección activa (step-up, restricción de operaciones, VPN sugerida).

### Escenario de Negocio

> *"Como usuario, quiero poder revisar mi cuenta bancaria desde el WiFi de la cafetería mientras espero mi café."*

- Caso: cliente usa WiFi abierto, atacante con rogue AP captura token y realiza transferencias → fraude y pérdida de confianza.
- Caso: la app no alerta ni limita operaciones en WiFi abierto; el banco enfrenta reclamaciones legales.

### Incidentes reportados

- **DarkHotel APT (2014-presente):** Comprometía WiFi de hoteles de lujo, instalando malware vía portales falsos a ejecutivos.
- **Firesheep (2010):** Demostró lo trivial de capturar cookies de sesión en redes no cifradas; millones afectados.
- **Kaspersky 2023:** 25% de WiFi públicos sin cifrado, 25% con cifrado débil (WEP); superficie ideal para sidejacking.

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
| **Capacidades (SÍ permite)** | Detectar tipo de red (WiFi/Celular/VPN). Clasificar WiFi segura vs insegura por cifrado. Step-up authentication en redes no confiables. Vincular sesión a fingerprint de red ([Session Binding](#term-session-binding)). Limitar operaciones por nivel de confianza. Sugerir VPN. |
| **Restricciones Duras (NO permite)** | **Certeza absoluta:** No garantiza que una red “privada” no esté comprometida. **VPN detection:** No siempre confiable. **iOS:** APIs limitadas para inspección detallada. **Consumo:** Monitoreo continuo impacta batería si no se optimiza. |
| **Criterio de Selección** | Se usa **[BLoC](#term-bloc)** porque el estado de red cambia dinámicamente y múltiples features deben reaccionar (transferencias, auth). `connectivity_plus` cubre lo básico; Platform Channels aportan detalles de seguridad de red. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Bloc cambia estados de red (trusted/untrusted/VPN) y notifica features dependientes | Equipo móvil, CI |
| Integration (CI) | En WiFi abierto → step-up requerido/operaciones sensibles bloqueadas; en celular/VPN → flujo normal | Móvil/QA, CI + staging |
| Seguridad manual | Sidejacking con rogue AP: token robado no reutilizable por session binding; warning visible y operaciones bloqueadas | Seguridad/QE en redes de prueba |
| Observabilidad | Eventos `network.context` con tipo de red, cifrado y acción tomada; alertas si untrusted > umbral | Móvil/SRE, CI |

### 3.2 UX y operación
- Mensaje claro en WiFi abierto: “Red no confiable, solo consulta habilitada; usa datos móviles o VPN para operar”.
- CTA para habilitar VPN de la empresa o cambiar a red segura.
- Revalidar riesgo al cambiar de red; limpiar tokens si cambia fingerprint de red.

### 3.3 Política de riesgo y límites
- **Untrusted (abierta/WEP):** Solo consulta; operaciones sensibles bloqueadas; step-up para login.
- **Trusted (WPA2/WPA3):** Operaciones permitidas; monitor de cambios rápidos de BSSID.
- **VPN:** Considerar riesgo medio-bajo, pero revalidar al desconectarse.

---

## 4. Impacto esperado
- Reducción de sesiones secuestradas en redes públicas ≥ 70% por binding y step-up.
- Alertas de redes no confiables en < 5 s; operaciones sensibles bloqueadas automáticamente.
- Soporte: tickets por “wifi bloqueada” controlados (<10 por semana) con mensajes claros.

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-session-hijacking"></a>Session Hijacking | Toma de sesión mediante robo de tokens/cookies en tránsito. |
| <a id="term-rogue-ap"></a>Rogue Access Point | Punto de acceso que suplanta una red legítima para interceptar tráfico. |
| <a id="term-arp-spoof"></a>ARP Spoofing | Ataque de red local que redirige tráfico a un atacante en la misma LAN. |
| <a id="term-ssl-stripping"></a>SSL Stripping | Downgrade de HTTPS a HTTP para robar credenciales/sesiones. |
| <a id="term-session-binding"></a>Session Binding a red | Asociar sesión a fingerprint de red (SSID/BSSID) y requerir revalidación al cambiar. |
| <a id="term-untrusted"></a>Red no confiable | Clasificación de WiFi abierto/cautivo que dispara controles adicionales (OTP, bloqueo de operaciones). |
| <a id="term-bloc"></a>BLoC | Patrón basado en eventos/estados, adecuado para cambios dinámicos de red. |

---

## Referencias

- [OWASP Mobile Security - Network Communication](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Android ConnectivityManager](https://developer.android.com/reference/android/net/ConnectivityManager)
- [iOS NEHotspotHelper](https://developer.apple.com/documentation/networkextension/nehotspothelper)
- [NIST Guidelines on Network Security](https://csrc.nist.gov/publications/detail/sp/800-153/final)

---

*Anterior: [Secure PIN Pad](caso-05-secure-pin-pad.md) | Siguiente: [Remember Me](caso-07-remember-me-seguro.md)*
