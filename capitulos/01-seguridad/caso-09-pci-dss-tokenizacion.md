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

### Problema detectado (técnico)
- Si la app procesa PAN/CVV directamente, todo el stack entra en scope PCI (infra, app, logs), con costos altos y riesgo legal.
- Filtrados en logs/crashes o analytics pueden exponer PAN/CVV aunque uses SDK si no se sanitiza.
- Sin 3DS2/SCA y tokenización correcta, se incrementa el riesgo de fraude y el rechazo por cumplimiento.

### Escenario de Negocio

> *"Como usuario, quiero guardar mi tarjeta de crédito en la app para compras futuras sin preocuparme por la seguridad de mis datos."*

Problema regulatorio: PCI-DSS tiene 12 requisitos y >300 controles; manejar PAN/CVV en código propio amplía el alcance y costo de auditoría.

### Incidentes reportados

- **Target (2013):** Brecha de 40M tarjetas por almacenamiento interno; costo estimado $292M.
- **British Airways (2018):** Multa GDPR £183M por robo de datos de pago via JS malicioso.
- **Verizon PCI Report 2023:** Solo 43.4% mantiene cumplimiento PCI todo el año.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Verizon PCI Report (2023) | Global | 43.4% mantienen cumplimiento PCI-DSS todo el año. |
| IBM Cost of Data Breach (2023) | Global | Promedio de costo por brecha con datos de pago: ~$4.45M. |
| Stripe/Adyen adoption (2024) | Integraciones móviles | Tokenización reduce scope a SAQ A-EP; menor coste de auditoría vs manejo directo de PAN. |

**Resumen global**
- Menos de la mitad de las organizaciones mantienen cumplimiento PCI constante.
- Brechas con datos de pago son costosas (>$4M promedio) y conllevan multas (PCI/GDPR).
- Tokenizar vía proveedor reduce alcance y costo, cumpliendo PSD2/SCA con 3DS2.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Provider maneja estados (loading/success/error) sin exponer PAN/CVV | Equipo móvil, CI |
| Integration (CI) | SDK retorna token; app nunca ve PAN/CVV; logs sanitizados | Móvil/Backend, CI + staging |
| Seguridad manual | Revisar logs/crashes/analytics para asegurar que no hay PAN/CVV; inspección de tráfico asegura solo token | Seguridad/QE |
| Observabilidad | Evento `payment.tokenized` con `card_brand`, `last4`, sin datos sensibles | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Captura | Usar hosted fields/SDK nativo; no TextField propio | Reduce scope y fugas |
| Reautenticación | 3DS2/SCA en operaciones sensibles; step-up si riesgo | Cumple PSD2/SCA |
| Display | Mostrar solo marca y last4; nunca CVV/exp completa | Minimiza exposición |
| Errores | Mensajes claros sin detallar datos de tarjeta | Evita filtraciones en UX |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Logs/analytics | Sanitizar; bloquear eventos con PAN/CVV | Previene fugas |
| Scope PCI | Mantener SAQ A-EP; no almacenar ni transmitir PAN | Reduce costo de auditoría |
| Tokens | Solo backend maneja tokens para cargo; app almacena metadata no sensible | Seguridad y compliance |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Manejar PAN/CVV en app amplía scope PCI y riesgo de fuga. |
| Opciones evaluadas | TextFields propios; SDK tokenización; hosted fields; 3DS2 opcional vs obligatorio. |
| Decisión | SDK nativo via Platform Channels + tokenización + 3DS2 obligatorio en flujos sensibles. |
| Consecuencias | Dependencia de proveedor; requerir config nativa; más pasos de integración. |
| Riesgos aceptados | Latencia de 3DS2; dependencia de servicios del PSP. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Scope PCI | SAQ A-EP (sin PAN/CVV en app) | Alerta si se detecta PAN/CVV en logs/analytics | Costos y riesgo regulatorio reducidos |
| Brechas con datos de tarjeta | 0 incidentes | Cualquier > 0 es crítico | Evitar multas y pérdida de reputación |
| Tiempo tokenización p95 | < 800 ms | Warning si se acerca | UX aceptable |
| Éxito 3DS2/SCA | > 98% en flujos sensibles | Alerta si baja | Cumplimiento y conversión |
| Logs/errores con datos | 0 eventos | Crítico si > 0 | Previene fugas |

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
| 3DS2 | Strong Customer Authentication basado en EMV 3-D Secure 2. |
| SAQ A-EP | Cuestionario de autoevaluación para merchants con tokenización/hosted fields. |

---

## Referencias

- [PCI-DSS Requirements](https://www.pcisecuritystandards.org/document_library)
- [Stripe Mobile SDKs](https://stripe.com/docs/mobile)
- [Adyen Mobile SDKs](https://docs.adyen.com/online-payments/build-your-integration/mobile)
- [PSD2 Strong Customer Authentication](https://ec.europa.eu/info/law/payment-services-psd-2-directive-eu-2015-2366_en)
- [Verizon PCI Report 2023](https://www.verizon.com/business/resources/TB/2023-payment-security-report.pdf)
- [IBM Cost of a Data Breach 2023](https://www.ibm.com/reports/data-breach)

---

*Anterior: [MFA SIM Swapping](caso-08-mfa-sim-swapping.md) | Siguiente: [Auditoría y Trazabilidad](caso-10-auditoria-trazabilidad.md)*
