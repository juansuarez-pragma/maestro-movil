# Caso 81: BLE Seguro para Wearables
## Sincronizar Dispositivos sin Exponer Datos

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | BLE, wearables, seguridad, pairing, cifrado |
| **Patrón Técnico** | Secure BLE Pairing, GATT Permissions, Data Encryption |
| **Stack Seleccionado** | Flutter + plugins BLE nativos + pairing seguro + cifrado app-layer |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero sincronizar mi wearable sin riesgo de interceptar mis datos de salud/finanzas."*

BLE sin seguridad permite sniffing, spoofing y modificación de datos.

### Evidencia de Industria

- **OWASP MASVS:** Requiere cifrado y autenticación en BLE.
- **Incidentes IoT:** Datos de salud filtrados por pairing débil.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Seguridad** | Interceptación/modificación de datos |
| **Regulatorio** | GDPR/HIPAA en datos de salud/finanzas |
| **Reputacional** | Pérdida de confianza en el dispositivo/app |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | BLE sin pairing/cifrado | **INADECUADO:** Riesgo extremo. |
| **ACEPTABLE** | Pairing estándar con bonding, sin cifrado app-layer | **MEJORA:** Mejor que nada, pero vulnera datos si pairing es roto. |
| **ENTERPRISE** | **Secure BLE + cifrado app-layer:** pairing con autenticación, whitelisting de device IDs, cifrado de payloads, rotación de keys | **ÓPTIMO:** Minimiza riesgo de interceptación y spoofing. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Pairing seguro (Just Works no recomendado; usar Passkey/Numeric Comparison). Whitelist de dispositivos pareados. Cifrado a nivel app de datos sensibles. Firmas/HMAC en payload. Validar servicios/características esperadas. |
| **Restricciones Duras (NO permite)** | **Hardware limitado:** Algunos wearables no soportan pairing fuerte. **Canales inseguros:** BLE puede ser jammeado. **UX:** Pairing demasiado estricto puede frustrar usuarios. |
| **Criterio de Selección** | Pairing autenticado, cifrado app-layer, validación de servicios GATT, rotación de claves; Riverpod para estado de conexión. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| BLE | Bluetooth Low Energy. |
| GATT | Perfil de atributos genéricos en BLE. |
| Pairing | Proceso de establecimiento de clave compartida. |
| Whitelist | Lista de dispositivos autorizados. |
| HMAC | Código de autenticación de mensajes basado en hash. |

---

## Referencias

- [OWASP MASVS - Device Communication](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Bluetooth SIG Security](https://www.bluetooth.com/learn-about-bluetooth/bluetooth-technology/bluetooth-security/)
