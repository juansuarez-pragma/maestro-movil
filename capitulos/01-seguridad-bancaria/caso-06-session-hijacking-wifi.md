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

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Kaspersky WiFi Report (2023) | Global | 25% de puntos públicos sin cifrado; 25% con WEP. |
| NowSecure State of Mobile App Security (2024) | 1,000+ apps (US/EU) | 85% fallan ≥1 control MASVS; fallas de comunicación segura entre los más comunes. |
| Cisco DNS/Network Security (2022) | Tráfico empresarial | ~20% de accesos corporativos pasan por redes no confiables al menos 1 vez/mes. |

**Resumen global**
- Redes abiertas o WEP siguen siendo ~50% del inventario público, facilitando sidejacking/SSL stripping.
- Fallas en controles de comunicación segura son comunes en apps móviles (MASVS).
- Uso frecuente de redes no confiables en escenarios corporativos/retail incrementa la exposición de tokens.

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
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Mensajes | En WiFi abierto: “Red no confiable, solo consulta; usa datos móviles/VPN para operar” | Alinea expectativas y reduce soporte |
| CTA | Botón para habilitar VPN o cambiar de red | Guía acción mitigadora |
| Cambio de red | Revalidar riesgo y limpiar tokens si cambia fingerprint de red | Evita reuse fuera del contexto original |

### 3.3 Política de riesgo y límites
| Clasificación de red | Comportamiento | Nota |
|:---------------------|:--------------|:-----|
| Untrusted (abierta/WEP) | Solo consulta; bloquear operaciones sensibles; step-up para login | Minimiza exposición en redes inseguras |
| Trusted (WPA2/WPA3) | Operaciones permitidas; monitorear cambios rápidos de BSSID | Detecta rogos/AP spoof |
| VPN | Riesgo medio-bajo; revalidar al desconectarse | Mantiene sesión al mover red segura → insegura |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Tokens robados en WiFi público permiten ATO; warnings solos son ignorados. |
| Opciones evaluadas | Solo HTTPS/pinning; warning sin acción; defensa en profundidad con detección de red, binding y step-up. |
| Decisión | Defensa en profundidad: detección de red + session binding + step-up/bloqueo en untrusted + sugerencia de VPN. |
| Consecuencias | Más lógica en cliente; potencial fricción en redes públicas; depende de exactitud de clasificación. |
| Riesgos aceptados | Falsos positivos en algunas redes corporativas; limitaciones de APIs iOS para inspección. |

---

## 4. Impacto esperado

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Secuestro de sesión en redes públicas | Reducción ≥ 70% | Alerta si no mejora tras rollout | Fraude contenido |
| Tiempo de alerta por red no confiable | < 5 s | Alerta si > 5 s | Respuesta rápida |
| Operaciones bloqueadas en untrusted | 100% de operaciones sensibles | Alerta si se ejecutan en untrusted | Minimiza exposición |
| Tickets “wifi bloqueada” | < 10/semana | Alerta si > 10 | Balance UX/seguridad controlado |
| Revalidaciones por cambio de red | 100% al detectar cambio de fingerprint | Alerta si se omite | Evita reuse fuera de contexto |

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
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
- [Kaspersky WiFi Security Report 2023](https://securelist.com/over-a-quarter-of-wi-fi-hotspots-are-insecure/105195/)
- [Cisco DNS/Network Security Report 2022](https://www.cisco.com/c/en/us/solutions/executive-perspectives/annual-cybersecurity-report.html)

---

*Anterior: [Secure PIN Pad](caso-05-secure-pin-pad.md) | Siguiente: [Remember Me](caso-07-remember-me-seguro.md)*
