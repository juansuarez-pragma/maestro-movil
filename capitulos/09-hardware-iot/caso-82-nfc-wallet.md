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

### Escenario de Negocio

> *"Como usuario, quiero usar NFC para pagos/tokens sin exponer mis datos."*

NFC sin controles permite sniffing/clonado de tags; pagos requieren tokenización y seguridad fuerte.

### Evidencia de Industria

- **Wallets móviles:** Usan HCE/tokenización para evitar exponer PAN.
- **Incidentes:** Clonado de tags NFC sin autenticación.

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
