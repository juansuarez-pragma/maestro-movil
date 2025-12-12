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

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Session Hijacking | Toma de sesión mediante robo de tokens/cookies en tránsito. |
| Rogue Access Point | Punto de acceso que suplanta una red legítima para interceptar tráfico. |
| ARP Spoofing | Ataque de red local que redirige tráfico a un atacante en la misma LAN. |
| SSL Stripping | Downgrade de HTTPS a HTTP para robar credenciales/sesiones. |
| Session Binding a red | Asociar sesión a fingerprint de red (SSID/BSSID) y requerir revalidación al cambiar. |
| Red no confiable | Clasificación de WiFi abierto/cautivo que dispara controles adicionales (OTP, bloqueo de operaciones). |

---

## Referencias

- [OWASP Mobile Security - Network Communication](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Android ConnectivityManager](https://developer.android.com/reference/android/net/ConnectivityManager)
- [iOS NEHotspotHelper](https://developer.apple.com/documentation/networkextension/nehotspothelper)
- [NIST Guidelines on Network Security](https://csrc.nist.gov/publications/detail/sp/800-153/final)

---

*Anterior: [Secure PIN Pad](caso-05-secure-pin-pad.md) | Siguiente: [Remember Me](caso-07-remember-me-seguro.md)*
