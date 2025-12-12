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

> *"Como usuario, quiero guardar mi tarjeta de crédito en la app para compras futuras."*

Problema regulatorio: PCI-DSS tiene 12 requisitos con 300+ controles. Si la app toca datos de tarjeta, toda la infraestructura entra en scope.

### Evidencia de Industria

**Caso Target (2013):** 40 millones de tarjetas por almacenar datos propios. Costo: $292M.

**Estadística Verizon PCI:** Solo 43% mantiene cumplimiento PCI-DSS todo el año.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Regulatorio** | Multas $5K-100K/mes de incumplimiento |
| **Económico** | Auditoría PCI Level 1: $50K-500K anuales |
| **Legal** | Responsabilidad directa si datos comprometidos |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Rol | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:----|:-----------------------|:----------------------------------|
| **Junior** | TextFields para PAN/CVV → backend → DB | **FALLA CRÍTICA:** Full PCI scope. Auditoría costosa. Multas seguras. |
| **Senior** | SDK Stripe/Adyen que retorna token | **MEJORA:** SDK maneja sensible. PERO: implementación puede filtrar en logs. |
| **Architect** | **PCI Scope Reduction:** SDK nativo via Platform Channels, nunca loggear, token en backend, 3DS2 | **ENTERPRISE:** PCI scope mínimo (SAQ A-EP). Sin datos sensibles propios. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Capturar datos sin pasar por código propio. Token reutilizable. Mostrar últimos 4 dígitos y marca. Soportar 3DS2. |
| **Restricciones Duras (NO permite)** | **Almacenar local:** PAN, CVV, expiry NUNCA. **Loggear:** Ningún dato de tarjeta. **Mostrar:** CVV nunca de vuelta. **Transmitir:** Solo token, nunca datos crudos. |
| **Criterio de Selección** | Se usa **Platform Channels** con SDKs nativos porque tienen certificación PCI y son auditados. Se usa **Provider** porque estado de métodos de pago es básico. |

---

## 4. Manos a la Obra: Estrategia de Implementación

### Fase 1: Arquitectura

**Flujo Agregar Tarjeta:**
1. Usuario toca "Agregar"
2. Platform Channel → SDK nativo
3. SDK presenta UI propia
4. SDK envía a Stripe/Adyen (nunca pasa por app)
5. Retorna token
6. App envía token a backend
7. Backend guarda user_id ↔ token

**Flujo Cobro:**
1. App envía: product_id, quantity, payment_token_id
2. Backend llama a Stripe con token
3. Stripe procesa, retorna resultado

### Fase 2: Implementación

**StripeNativeChannel:**
- MethodChannel `com.app/stripe_native`
- `createPaymentMethod`: presenta CardField, retorna ID
- `confirmPayment`: para 3DS2

**PaymentMethodProvider:**
- Lista de tokens (solo metadata: últimos 4, marca, alias)
- NUNCA almacenar datos sensibles localmente

**Logging Sanitization:**
- NUNCA capturar campos: card, pan, cvv, cvc, number, expiry
- Regex para detectar y redactar patrones de tarjeta
- Configurar scrubbing en Sentry/Crashlytics

---

## 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

```
[ ] TAC-9.1: Datos de tarjeta NUNCA pasan por código de la app.
[ ] TAC-9.2: DEBE usarse SDK certificado PCI-DSS del provider.
[ ] TAC-9.3: SDK DEBE invocarse via Platform Channels (nativo).
[ ] TAC-9.4: Solo token opaco se recibe y transmite.
[ ] TAC-9.5: NINGÚN dato en logs, analytics, crash reports.
[ ] TAC-9.6: UI solo muestra últimos 4 dígitos y marca.
[ ] TAC-9.7: Tokens en backend, NUNCA localmente.
[ ] TAC-9.8: DEBE implementarse 3DS2 para SCA.
[ ] TAC-9.9: Eliminación de método DEBE invalidar token.
[ ] TAC-9.10: Auditoría debe confirmar SAQ A-EP.
```

---

## 6. Estrategia de Pruebas (Shift-Left)

| # | Escenario | Qué Validar | Tipo |
|:-:|:----------|:------------|:-----|
| 1 | **PAN en logs** | Debug mode, agregar tarjeta, buscar en logs. NO debe aparecer. | Security |
| 2 | **3DS2 completo** | Tarjeta test que requiere 3DS. Completar challenge. | E2E |
| 3 | **Eliminar método** | Token debe fallar en backend | Integration |

---

*Anterior: [MFA SIM Swapping](caso-08-mfa-sim-swapping.md) | Siguiente: [Auditoría y Trazabilidad](caso-10-auditoria-trazabilidad.md)*
