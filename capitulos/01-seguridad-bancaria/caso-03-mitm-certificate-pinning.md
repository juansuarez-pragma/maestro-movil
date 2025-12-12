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

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Certificate Pinning | Validar que el certificado presentado coincide con uno o varios pins definidos en la app. |
| Public Key Pinning (SPKI) | Pin sobre la clave pública del certificado, sobrevive renovaciones si la key pair se mantiene. |
| Backup Pins | Pins adicionales para rollover seguro cuando cambia el certificado o la clave. |
| TOFU | Trust On First Use; confiar en el primer certificado visto y fijarlo para sesiones futuras. |
| HTTPS Interception | Uso de proxies/CA adicionales para inspeccionar tráfico; pinning lo bloquea. |
| ATS (iOS) | App Transport Security; configuración de seguridad de red en iOS que interactúa con pinning. |

---

## Referencias

- [OWASP Certificate Pinning Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Pinning_Cheat_Sheet.html)
- [Android Network Security Config](https://developer.android.com/training/articles/security-config)
- [iOS App Transport Security](https://developer.apple.com/documentation/bundleresources/information_property_list/nsapptransportsecurity)
- [PCI-DSS Requirement 4.1](https://www.pcisecuritystandards.org/)

---

*Anterior: [Biometría Falsificada](caso-02-biometria-falsificada.md) | Siguiente: [Root/Jailbreak Detection](caso-04-root-jailbreak-detection.md)*
