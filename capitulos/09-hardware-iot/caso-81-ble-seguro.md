# Caso 81: [BLE](#term-ble "Bluetooth Low Energy.") Seguro para Wearables
## Sincronizar Dispositivos sin Exponer Datos

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | BLE, wearables, seguridad, pairing, cifrado |
| **Patrón Técnico** | Secure BLE [Pairing](#term-pairing "Proceso de establecimiento de clave compartida."), [GATT](#term-gatt "Perfil de atributos genéricos en BLE.") Permissions, Data Encryption |
| **Stack Seleccionado** | Flutter + plugins BLE nativos + pairing seguro + cifrado app-layer |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Pairing débil o sin cifrado permite sniffing/spoofing de datos sensibles.
- Sin whitelist ni validación de servicios, se aceptan dispositivos falsos.
- Falta de cifrado app-layer expone datos aunque el canal BLE se rompa.

### Escenario de Negocio

> *"Como usuario, quiero sincronizar mi wearable sin riesgo de interceptar mis datos de salud/finanzas."*

### Incidentes reportados
- **IoT/health:** Datos de salud filtrados por pairing débil.
- **OWASP MASVS:** Exige cifrado y autenticación en BLE; fallas comunes en apps móviles.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Incidentes BLE/IoT | Global | Sniffing/spoofing comunes con pairing débil. |
| MASVS testing | Global | Cifrado y autenticación en BLE fallan con frecuencia. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; comunicaciones inseguras son recurrentes. |

**Resumen global**
- BLE sin pairing fuerte ni cifrado app-layer expone datos; se requieren pairing autenticado, whitelist y cifrado adicional.

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
| **Capacidades (SÍ permite)** | Pairing seguro (Just Works no recomendado; usar Passkey/Numeric Comparison). [Whitelist](#term-whitelist "Lista de dispositivos autorizados.") de dispositivos pareados. Cifrado a nivel app de datos sensibles. Firmas/[HMAC](#term-hmac "Código de autenticación de mensajes basado en hash.") en payload. Validar servicios/características esperadas. |
| **Restricciones Duras (NO permite)** | **Hardware limitado:** Algunos wearables no soportan pairing fuerte. **Canales inseguros:** BLE puede ser jammeado. **UX:** Pairing demasiado estricto puede frustrar usuarios. |
| **Criterio de Selección** | Pairing autenticado, cifrado app-layer, validación de servicios GATT, rotación de claves; Riverpod para estado de conexión. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Seguridad | Pairing fuerte y cifrado app-layer | Seguridad/QA |
| Integration (CI) | Whitelist y validación de servicios GATT | Móvil/QA |
| Observabilidad | Eventos `ble.*` (conexión, pairing, errores) | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Pairing | UI clara para Passkey/Numeric Comparison | Menos fricción |
| Reconexión | Manejo de reintentos con backoff | Estabilidad |
| Mensajes | Alertas ante dispositivos no autorizados | Seguridad percibida |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Rotación de claves | Periódica y atada a pairing | Defensa |
| Compatibilidad | Matriz de wearables y soporte de pairing fuerte | Planeación |
| Privacidad | No enviar PII en claro ni en logs | Cumplimiento |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Sincronizar BLE sin exponer datos sensibles. |
| Opciones evaluadas | BLE sin pairing; pairing estándar; pairing autenticado + cifrado app-layer + whitelist. |
| Decisión | Pairing autenticado, cifrado app-layer, whitelist/validación de servicios y rotación de claves. |
| Consecuencias | Requiere UX de pairing y compatibilidad de hardware; mayor complejidad. |
| Riesgos aceptados | Jam y limitaciones de dispositivos; posibles fricciones de UX. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Incidentes de sniffing/spoofing | 0 | Crítico si >0 | Seguridad |
| Tasa de pairing exitoso | > 95% | Warning si baja | UX |
| Errores de conexión | Tendencia a la baja | Alerta si sube | Estabilidad |
| Alertas de dispositivos no autorizados | Registradas y bloqueadas | Crítico si pasan | Protección |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-ble"></a>BLE | Bluetooth Low Energy. |
| <a id="term-gatt"></a>GATT | Perfil de atributos genéricos en BLE. |
| <a id="term-pairing"></a>Pairing | Proceso de establecimiento de clave compartida. |
| <a id="term-whitelist"></a>Whitelist | Lista de dispositivos autorizados. |
| <a id="term-hmac"></a>HMAC | Código de autenticación de mensajes basado en hash. |

---

## Referencias

- [OWASP MASVS - Device Communication](https://owasp.org/www-project-mobile-security-testing-guide/)
- [Bluetooth SIG Security](https://www.bluetooth.com/learn-about-bluetooth/bluetooth-technology/bluetooth-security/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
