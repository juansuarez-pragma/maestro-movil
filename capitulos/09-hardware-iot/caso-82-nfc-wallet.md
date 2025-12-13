# Caso 82: NFC Wallet Seguro
## Leer/Escribir Tags y Tokens sin Riesgo

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | nfc, wallet, pagos, tokenización, seguridad |
| **Patrón Técnico** | Secure NFC, Tokenization, HCE (Host Card Emulation) |
| **Stack Seleccionado** | Flutter + Platform Channels NFC + tokenización + cifrado en app |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Leer/escribir NFC sin autenticación/cifrado permite clonado y replay.
- Sin tokenización ni HCE/SE, el PAN o datos sensibles quedan expuestos.
- Variabilidad de hardware/OS complica la compatibilidad si no se prueba.

### Escenario de Negocio

> *"Como usuario, quiero usar NFC para pagos/tokens sin exponer mis datos."*

### Incidentes reportados
- **Clonado NFC:** Tags sin autenticación fueron copiados para fraude.
- **Wallets móviles:** Exponen PAN cuando no usan tokenización/HCE.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| PCI-DSS casos | Global | Tokenización recomendada para minimizar exposición de PAN. |
| Incident reports | Global | Clonado/replay de tags sin autenticación. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; comunicaciones/almacenamiento inseguro frecuentes. |

**Resumen global**
- Tokenizar y usar HCE/SE cuando sea posible reduce fraude; autenticación/cifrado en NFC es clave para integridad.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Seguridad** | Clonado, replay, intercepción de tokens |
| **Regulatorio** | PCI-DSS aplica a datos de pago |
| **Reputacional** | Fraude en pagos NFC afecta confianza |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Leer/escribir NFC sin autenticación ni cifrado | **INADECUADO:** Exposición total. |
| **ACEPTABLE** | Autenticación básica, sin tokenización | **MEJORA:** Menos riesgo, pero PAN/datos aún expuestos. |
| **ENTERPRISE** | **Tokenización + HCE/SE:** tokens en lugar de PAN, autenticación/tag, cifrado, verificación de integridad, storage seguro | **ÓPTIMO:** Minimiza riesgo y cumple PCI. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Usar tokens opacos en vez de PAN. Autenticación mutua para escritura/lectura. Cifrar payloads NFC. Validar integridad (MAC). HCE para emulación segura. |
| **Restricciones Duras (NO permite)** | **Hardware/SE:** No todos los dispositivos soportan Secure Element; HCE depende del SO. **Latencia:** Operaciones seguras pueden ser más lentas. **Compatibilidad:** Varias APIs de NFC varían por fabricante. |
| **Criterio de Selección** | Tokenización y HCE donde posible; cifrado y MAC; policies de lectura/escritura; almacenamiento seguro de claves. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Seguridad | Tokenización y cifrado/MAC de payloads | Seguridad |
| Integration (devices) | Compatibilidad HCE/SE y lectura/escritura autenticada | Móvil/QA |
| Observabilidad | Eventos `nfc.*` con errores y métricas de intento | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Mensajes | Guiar acercamiento NFC y errores claros | Menos fricción |
| Fallback | Si no hay NFC/HCE, opciones alternativas | Cobertura |
| Latencia | Optimizar timeouts y feedback háptico | UX ágil |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Claves | En SE o secure storage; rotación | Seguridad |
| Regulación | Cumplir PCI-DSS (sin PAN en claro) | Compliance |
| Compatibilidad | Matriz de dispositivos/fabricantes soportados | Planeación |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Riesgo de clonado/replay y exposición de PAN en NFC. |
| Opciones evaluadas | NFC sin autenticación; autenticación básica; tokenización + HCE/SE con cifrado/MAC. |
| Decisión | Tokenización con HCE/SE, autenticación mutua, cifrado/MAC y storage seguro de claves. |
| Consecuencias | Necesita pruebas de compatibilidad y manejo de claves; mayor complejidad. |
| Riesgos aceptados | Variabilidad de hardware; posibles latencias. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Fraude/clonado NFC | 0 | Crítico si >0 | Seguridad |
| Éxito de transacción NFC | > 98% en dispositivos soportados | Warning si baja | UX |
| Tiempos de lectura/escritura | Estables | Alerta si suben | Performance |
| Incidentes de compatibilidad | Tendencia a la baja | Crítico si sube | Cobertura |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| HCE | Host Card Emulation; emula tarjeta en software. |
| Tokenización | Sustituir PAN/dato sensible por token opaco. |
| Secure Element | Hardware seguro para claves/operaciones. |
| MAC | Message Authentication Code para integridad. |
| PAN | Primary Account Number; dato sensible de tarjeta. |

---

## Referencias

- [PCI-DSS Tokenization](https://www.pcisecuritystandards.org/)
- [Android NFC/HCE](https://developer.android.com/guide/topics/connectivity/nfc/hce)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
