# Caso 9: PCI-DSS en el Bolsillo
## Tokenización de Tarjetas sin Tocar Datos Sensibles

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | PCI-DSS, tokenización, datos de tarjeta, PAN, CVV, procesamiento de pagos, pasarela de pagos |
| **Patrón Técnico** | Tokenization, Hosted Fields, PCI Scope Reduction |
| **Stack Seleccionado** | Platform Channels (SDKs de Stripe/Adyen) + Provider (PaymentMethodProvider) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero guardar mi tarjeta de crédito en la app para compras futuras sin preocuparme por la seguridad de mis datos."*

Problema regulatorio: PCI-DSS tiene 12 requisitos principales con más de 300 controles. Si la app maneja datos de tarjeta (PAN, CVV), toda la infraestructura entra en scope de auditoría.

### Evidencia de Industria

**Caso Target (2013):** Brecha que expuso 40 millones de tarjetas por almacenar datos de tarjeta en sistemas propios. Costo total: $292 millones entre multas, compensaciones y remediación.

**Caso British Airways (2018):** Multa GDPR de £183 millones por robo de datos de pago via JavaScript malicioso.

**Estadística Verizon PCI Report 2023:** Solo 43.4% de las organizaciones mantienen cumplimiento PCI-DSS todo el año. El resto falla en algún punto.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Regulatorio** | Multas de $5,000-100,000/mes de incumplimiento PCI |
| **Económico** | Auditoría PCI Level 1 (si almacenas datos): $50K-500K anuales |
| **Legal** | Responsabilidad directa si datos comprometidos, pérdida de capacidad de procesar pagos |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | TextFields para PAN/CVV → enviar a backend → guardar en DB | **FALLA CRÍTICA:** Full PCI scope. Auditoría anual costosa. Multas seguras. Responsabilidad legal. |
| **ACEPTABLE** | SDK de Stripe/Adyen en Flutter que retorna token | **CUMPLE MÍNIMOS:** SDK maneja datos sensibles. PERO: implementación puede filtrar datos en logs, analytics, o crashes. |
| **ENTERPRISE** | **PCI Scope Reduction:** SDK nativo via Platform Channels, sanitización de logs, token solo en backend, 3DS2 para SCA | **ÓPTIMO:** PCI scope mínimo (SAQ A-EP). Sin datos sensibles en código propio. Cumple PSD2/SCA. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Capturar datos de tarjeta sin que pasen por código propio. Token reutilizable para cobros recurrentes. Mostrar últimos 4 dígitos y marca de tarjeta. Soportar 3DS2 para Strong Customer Authentication. Múltiples métodos de pago. |
| **Restricciones Duras (NO permite)** | **Almacenar local:** PAN, CVV, fecha de expiración NUNCA en dispositivo. **Loggear:** Ningún dato de tarjeta en logs, analytics, crash reports. **Mostrar:** CVV nunca de vuelta al usuario. **Transmitir:** Solo token opaco, nunca datos crudos a backend propio. |
| **Criterio de Selección** | Se usa **Platform Channels** con SDKs nativos porque tienen certificación PCI-DSS y son auditados por los providers. Se usa **Provider simple** porque estado de métodos de pago es básico (lista, seleccionado, loading). |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Tokenización | Reemplazar datos sensibles (PAN) por tokens no sensibles. |
| PAN | Primary Account Number de la tarjeta; dato regulado por PCI-DSS. |
| Vault | Servicio seguro que almacena PAN y realiza detokenización bajo políticas estrictas. |
| Detokenización | Proceso de recuperar el PAN original desde un token bajo controles de autorización. |
| Scope PCI | Alcance de cumplimiento PCI; reducirlo disminuye superficie de auditoría. |
| PCI-DSS SAQ | Self-Assessment Questionnaire para evaluar cumplimiento según tipo de integración. |

---

## Referencias

- [PCI-DSS Requirements](https://www.pcisecuritystandards.org/document_library)
- [Stripe Mobile SDKs](https://stripe.com/docs/mobile)
- [Adyen Mobile SDKs](https://docs.adyen.com/online-payments/build-your-integration/mobile)
- [PSD2 Strong Customer Authentication](https://ec.europa.eu/info/law/payment-services-psd-2-directive-eu-2015-2366_en)

---

*Anterior: [MFA SIM Swapping](caso-08-mfa-sim-swapping.md) | Siguiente: [Auditoría y Trazabilidad](caso-10-auditoria-trazabilidad.md)*
