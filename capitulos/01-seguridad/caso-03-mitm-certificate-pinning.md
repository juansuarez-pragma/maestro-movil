# Caso 3: El Man-in-the-Middle Silencioso
## Certificate Pinning en Transferencias Interbancarias

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | certificate pinning, SSL pinning, MITM attack, transferencia interbancaria, SPEI, ACH, seguridad en tránsito |
| **Patrón Técnico** | Certificate Pinning, Public Key Pinning, Trust on First Use ([TOFU](#term-tofu "Trust On First Use; confiar en el primer certificado visto y fijarlo para sesiones futuras.")) |
| **Stack Seleccionado** | Dio + dio_http2_adapter + certificados/pins en config firmada + BLoC (TransferBloc) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- TLS estándar confía en cualquier CA instalada en el dispositivo. Con un proxy + CA maliciosa, el atacante lee credenciales, tokens y payloads de transferencias.
- Implementaciones con **un solo certificado** fallan al rotar el cert: dejan la app inoperable o fuerzan a desactivar pinning para “arreglarlo”.
- Sin telemetría, los eventos de bypass de pinning no generan alertas: el SOC no detecta intentos de MITM ni debugging indebido.

### Escenario de negocio
> *"Como tesorero corporativo, quiero enviar una transferencia SPEI de $250,000 desde la app, sabiendo que nadie puede interceptar ni modificar los datos en tránsito."*

- Caso realista: empleado con dispositivo rooteado instala cert corporativo para navegar. Sin pinning, un malware en la misma red capta tokens y payloads de transferencia.
- Resultado: transferencia redirigida a cuenta del atacante; cliente reporta fraude y exige reverso → costo directo y pérdida de confianza.

### Incidentes reportados
- **DigiNotar (2011):** CA comprometida emitió >500 certificados falsos; tráfico bancario interceptado en Irán.
- **Lenovo Superfish (2015):** CA preinstalada permitía MITM en HTTPS para millones de equipos; evidencia del riesgo de confiar en “cualquier” CA.
- **Estudio académico (2019):** 73% de apps bancarias Android fallaban en pinning; proxies como mitmproxy/Charles leían credenciales y tokens.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Estudio académico (2019) | 100+ apps bancarias Android | 73% no implementaban pinning correctamente; expuestas a MITM con CA instalada. |
| NowSecure State of Mobile App Security (2024) | 1,000+ apps iOS/Android (US/EU) | Fallas de comunicación segura (pinning/ATS) en el top 3 de hallazgos MASVS; 85% fallan ≥1 control MASVS. |
| OWASP MASVS Community Testing (2023) | Apps móviles globales | Uso de un solo certificado sin pins de respaldo/rotación es un error recurrente. |

**Resumen global de hallazgos**
- Más del 70% de apps bancarias evaluadas en Android presentan pinning ausente o mal configurado (estudio 2019).
- 85% de apps móviles fallan en ≥1 control MASVS; pinning/ATS figura entre los fallos más frecuentes (NowSecure 2024).
- Uso de un solo cert sin pins de respaldo es un patrón común que causa caídas en rotación y expone a MITM (OWASP MASVS tests 2023).

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Transferencias redirigidas; pérdidas potenciales > $250K por incidente; robo de credenciales reutilizables |
| **Regulatorio** | Incumplimiento PCI-DSS 4.1/PSD2 (cifrado en tránsito robusto); exposición a sanciones |
| **Legal/Reputacional** | Responsabilidad civil por negligencia si no se adoptó pinning; pérdida de confianza y fuga de clientes |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | HTTPS estándar con Dio, confiar en CAs del sistema | **INADECUADO:** Vulnerable a cualquier CA comprometida. Proxy (Charles/mitmproxy) + certificado instalado lee todo en claro. |
| **ACEPTABLE** | [Certificate pinning](#term-cert-pinning) con `SecurityContext` y certificado en assets | **CUMPLE MÍNIMOS:** Bloquea CAs desconocidas. PERO: si cert expira, app muere; sin pins de respaldo ni rotación planificada. |
| **ENTERPRISE** | **Multi-layer pinning:** 1) [Public Key pinning](#term-spki) (2–3 SPKI en Base64), 2) Backup pins para rollover 90 días, 3) Fallback controlado (`fail-closed` salvo ventana de rotación), 4) Config remota firmada para actualizar pins, 5) Detección de proxy/root/jailbreak y alertas SOC | **ÓPTIMO PARA BANCA:** Sobrevive rotaciones, minimiza indisponibilidad, bloquea MITM y genera telemetría accionable. Cumple PCI-DSS 4.1 y OWASP MAS. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Bloquear certificados no autorizados incluso de CAs “confiables”. Sobrevivir rotación con [Public Key pinning](#term-spki) y 2–3 pins activos. Detección de proxies de debug/root/jailbreak. Actualización de pins sin release via config firmada. |
| **Restricciones Duras (NO permite)** | **Debugging:** Pinning complica tráfico vía proxy (habilitar solo en debug). **Proxies corporativos:** Usuarios con proxy SSL legítimo se bloquean por política. **Expiración:** Si vencen todos los pins sin rollover, la app queda inoperable. **ATS:** Configurar excepciones iOS si el endpoint usa TLS intermedio. |
| **Criterio de Selección** | Se usa **[Public Key pinning](#term-spki)** sobre certificate pinning porque la clave pública sobrevive renovaciones. Se elige **[BLoC](#term-bloc)** para `TransferBloc` por flujo con múltiples eventos y necesidad de auditoría de estados. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Interceptor de Dio aplica pinning y rechaza cert no autorizado | Equipo móvil, CI |
| Integration (CI) | Cert expirado → se usa backup pin; config remota firma válida antes de aplicar | Móvil/Backend, CI + staging |
| Seguridad manual | Proxy MITM (Charles/mitmproxy) con CA instalada debe ser bloqueado; alerta generada | Seguridad/QE en dispositivos reales |
| Observabilidad | Evento `pinning.blocked` con hash del cert presentado, motivo y endpoint | Equipo móvil/SRE, CI |

### 3.2 Arquitectura práctica (flujo resumido)
```
App móvil → Handshake TLS → Valida SPKI contra pins (2-3 activos)
    ↳ Pin match: continúa
    ↳ Pin mismatch: bloquea, emite evento SOC, registra fingerprint del cert
Config firmada (remota): rota pins cada 90 días; backup pin siempre presente
```

### 3.3 Operación y resiliencia
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Rollover | 2–3 SPKI activos; agregar nuevo pin 2 semanas antes, retirar viejo tras validar | Minimiza downtime por rotación |
| Fallback | `fail-closed` por defecto; `fail-open` solo en ventana aprobada y monitoreada | Controla indisponibilidad sin abrir MITM |
| Debug | Pinning desactivado solo en `debug`; `release` siempre activo | Facilita pruebas sin comprometer producción |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | MITM posible con CA instalada; caída de app al rotar cert único. |
| Opciones evaluadas | HTTPS estándar; pinning por certificado único; pinning por SPKI con backup y config remota. |
| Decisión | Pinning por SPKI con 2–3 pins, backup, config remota firmada, detección de proxy/root/JB. |
| Consecuencias | Necesita gestión de pins en backend/config; posible bloqueo en proxies corporativos (política fail-closed). |
| Riesgos aceptados | Debug restringido; usuarios en proxies SSL legítimos pueden verse bloqueados (política de seguridad). |

### 3.5 Observabilidad y seguridad
| Elemento | Detalle | Umbral/Alerta |
|:---------|:--------|:--------------|
| Eventos | `pinning.blocked`, `pinning.matched`, `pinning.failopen_enabled` | Alertar al SOC en bloqueos |
| Métricas | p95 handshake TLS < 500 ms; pin-match > 99.9% | Warning si se acerca a límites |
| Alertas SOC | `pinning.blocked` > 3 eventos/hora/dispositivo | Indica MITM/debug indebido |

### 3.6 Política de pins (rotación y custodia)
| Aspecto | Política |
|:--------|:---------|
| Cantidad de pins activos | 2–3 SPKI simultáneos |
| Formato | SPKI en Base64 (RSA/ECDSA) |
| Rotación | Cada 90 días o al rotar key pair; agregar pin nuevo antes de retirar el viejo |
| Fallback | `fail-closed` salvo ventana de rotación aprobada; revertir si `pinning.blocked` se dispara |
| Publicación | Config remota firmada (JWKS/firmware) con `kid`, algoritmo y expiración |

---

## 4. Impacto esperado

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Bloqueo de MITM (pin match) | > 99.9% de sesiones | Alertar mismatches > 0.1% | Tráfico protegido contra CA/propios maliciosos |
| Disponibilidad en rotación | 0 caídas; ventana < 24 h con ≥2 SPKI | Alerta si pin faltante o caída por rotación | Continuidad del servicio al rotar cert/keys |
| Rendimiento handshake TLS | p95 < 500 ms; overhead pinning < 30 ms | Warning si se acerca al límite | UX sin latencia extra notable |
| Eventos `pinning.blocked` | ≤ 3 por dispositivo/hora | Alerta si > 3 | Detección de MITM/debug indebido |
| Cobertura de pinning | 100% de endpoints críticos | Alerta si endpoint sin pinning | Cierra superficie de ataque en APIs sensibles |
| Detección proxy/root/JB | Eventos con huella de cert y motivo | Alerta inmediata a SOC | Respuesta rápida ante ataques/interceptores |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-cert-pinning"></a>Certificate Pinning | Validar que el certificado presentado coincide con pins definidos en la app. |
| <a id="term-spki"></a>Public Key Pinning (SPKI) | Pin sobre la clave pública del certificado, sobrevive renovaciones si la key pair se mantiene. |
| <a id="term-backup"></a>Backup Pins | Pins adicionales para rollover seguro cuando cambia el certificado o la clave. |
| <a id="term-tofu"></a>TOFU | Trust On First Use; confiar en el primer certificado visto y fijarlo para sesiones futuras. |
| <a id="term-https-intercept"></a>HTTPS Interception | Uso de proxies/CA adicionales para inspeccionar tráfico; pinning lo bloquea. |
| <a id="term-ats"></a>ATS (iOS) | App Transport Security; configuración de seguridad de red en iOS que interactúa con pinning. |
| <a id="term-bloc"></a>BLoC | Patrón de estado basado en eventos y transición de estados con auditoría. |

---

## Referencias

- [OWASP Certificate Pinning Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Pinning_Cheat_Sheet.html)
- [Android Network Security Config](https://developer.android.com/training/articles/security-config)
- [iOS App Transport Security](https://developer.apple.com/documentation/bundleresources/information_property_list/nsapptransportsecurity)
- [PCI-DSS Requirement 4.1](https://www.pcisecuritystandards.org/)
- [OWASP MASVS/MASTG - Network Communication](https://mas.owasp.org/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
- [Why TLS/Certificate Pinning Fails (2019 study)](https://www.usenix.org/conference/usenixsecurity19/presentation/de-pereira)

---

*Anterior: [Biometría Falsificada](caso-02-biometria-falsificada.md) | Siguiente: [Root/Jailbreak Detection](caso-04-root-jailbreak-detection.md)*
