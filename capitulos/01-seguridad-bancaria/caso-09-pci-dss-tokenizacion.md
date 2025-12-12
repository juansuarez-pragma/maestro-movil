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

## 4. Manos a la Obra: Estrategia de Implementación

### Justificación del Plan

La estrategia se deriva del caso Target y los requisitos de PCI-DSS:

1. **Tocar datos = scope completo** → SDK del processor maneja captura
2. **Logs son vectores de fuga** → Sanitización agresiva
3. **Backend no necesita PAN** → Solo tokens opacos
4. **PSD2 requiere SCA** → Implementar 3DS2

La arquitectura usa Platform Channels para invocar SDKs nativos certificados, con Provider para estado simple de métodos de pago guardados.

---

### Fase 1: Diseño — Arquitectura de Tokenización

**Flujo Agregar Tarjeta:**
1. Usuario toca "Agregar tarjeta"
2. Flutter invoca Platform Channel
3. Platform Channel lanza SDK nativo (Stripe/Adyen)
4. SDK presenta su propia UI segura
5. Usuario ingresa datos en UI del SDK (nunca pasa por nuestra app)
6. SDK envía datos directamente a Stripe/Adyen (nunca pasa por nuestro backend)
7. SDK retorna token opaco
8. App envía token a nuestro backend
9. Backend asocia user_id ↔ payment_token

**Flujo Cobro:**
1. App envía: product_id, quantity, payment_token_id
2. Backend obtiene token real de DB
3. Backend llama a Stripe/Adyen con token
4. Processor procesa, retorna resultado
5. App muestra resultado

**Estructura de Carpetas:**
```
lib/features/payments/
├── data/
│   ├── datasources/
│   │   └── payment_native_datasource.dart
│   └── repositories/
│       └── payment_repository_impl.dart
├── domain/
│   ├── entities/
│   │   ├── payment_method.dart
│   │   └── payment_result.dart
│   └── usecases/
│       ├── add_payment_method.dart
│       └── process_payment.dart
└── presentation/
    ├── providers/
    │   └── payment_method_provider.dart
    └── widgets/
        └── payment_method_card.dart
```

---

### Fase 2: Implementación por Plataforma

#### 2.1 Flutter (Cross-Platform) — Capa de Aplicación

**PaymentNativeChannel:**
- MethodChannel: `com.app/payment_native`
- `createPaymentMethod()`: Lanza SDK, retorna token ID
- `confirmPayment(clientSecret)`: Para 3DS2 challenge
- Manejo de errores específicos del SDK

**PaymentMethodProvider:**
- Lista de payment methods del usuario (solo metadata: últimos 4, marca, alias)
- NUNCA almacenar datos sensibles localmente
- Sincronizar con backend al iniciar

**LogSanitizer:**
- Interceptor de logs que detecta y redacta patrones de tarjeta
- Regex: `\b(?:\d{4}[-\s]?){3}\d{4}\b` para PANs
- Configurar scrubbing en Sentry/Crashlytics
- Bloquear campos: card, pan, cvv, cvc, number, expiry

---

#### 2.2 Android — Integración SDK Nativo

**Stripe Android SDK:**
- Agregar dependencia en `build.gradle`
- Configurar `publishableKey` en Application
- Usar `PaymentSheet` o `CardInputWidget`

**Platform Channel Handler (Kotlin):**
```kotlin
class PaymentMethodChannel(private val activity: Activity) : MethodChannel.MethodCallHandler {
    override fun onMethodCall(call: MethodCall, result: MethodChannel.Result) {
        when (call.method) {
            "createPaymentMethod" -> launchStripeCardInput(result)
            "confirmPayment" -> confirmWith3DS(call.argument("clientSecret"), result)
        }
    }
}
```

**ProGuard Rules:**
- No ofuscar clases del SDK de Stripe/Adyen
- Mantener reflection para JSON parsing

**Consideraciones Android:**
- SDK requiere Activity context para UI
- Manejar lifecycle durante captura de tarjeta
- Network Security Config: permitir dominios del processor

---

#### 2.3 iOS — Integración SDK Nativo

**Stripe iOS SDK:**
- Agregar pod en Podfile
- Configurar `STPAPIClient.shared.publishableKey`
- Usar `STPPaymentCardTextField` o `PaymentSheet`

**Platform Channel Handler (Swift):**
```swift
class PaymentMethodChannel: NSObject, FlutterPlugin {
    func handle(_ call: FlutterMethodCall, result: @escaping FlutterResult) {
        switch call.method {
        case "createPaymentMethod":
            presentStripeCardInput(result: result)
        case "confirmPayment":
            confirmWith3DS(clientSecret: clientSecret, result: result)
        }
    }
}
```

**Info.plist:**
- `NSCameraUsageDescription` si se usa scan de tarjeta
- App Transport Security: permitir dominios del processor

**Consideraciones iOS:**
- Usar `UIViewController` para presentar UI del SDK
- Manejar `SceneDelegate` si app usa scenes
- Soporte para Apple Pay via SDK

---

### Fase 3: Observability — Métricas y Alertas

**Métricas:**
- `payment.method.added.success` / `payment.method.added.failure`
- `payment.processed.success` / `payment.processed.failure`
- `payment.3ds.triggered` / `payment.3ds.completed`
- `payment.method.deleted`

**Alertas:**

| Evento | Severidad | Acción |
|:-------|:----------|:-------|
| `payment.processed.failure` rate > 5% | P2 High | Investigar con processor |
| `payment.3ds.failure` rate > 10% | P3 Medium | Revisar flujo 3DS |
| PAN detectado en logs | P1 Critical | Incidente de seguridad |

---

## 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

### TACs Flutter (Cross-Platform)

```
[ ] TAC-9.1-FLUTTER: Datos de tarjeta NUNCA pasan por código de la app.

[ ] TAC-9.2-FLUTTER: DEBE usarse SDK certificado PCI-DSS del payment processor.

[ ] TAC-9.3-FLUTTER: Solo token opaco se recibe de SDK y transmite a backend.

[ ] TAC-9.4-FLUTTER: NINGÚN dato de tarjeta en logs, analytics, crash reports.

[ ] TAC-9.5-FLUTTER: UI solo muestra últimos 4 dígitos y marca de tarjeta.

[ ] TAC-9.6-FLUTTER: Tokens SOLO en backend, NUNCA almacenados localmente.
```

### TACs Android

```
[ ] TAC-9.7-ANDROID: SDK DEBE invocarse via Platform Channels para UI nativa.

[ ] TAC-9.8-ANDROID: ProGuard rules DEBEN preservar clases del SDK de pagos.

[ ] TAC-9.9-ANDROID: Network Security Config DEBE permitir dominios del processor.

[ ] TAC-9.10-ANDROID: DEBE manejarse lifecycle de Activity durante captura.
```

### TACs iOS

```
[ ] TAC-9.11-IOS: SDK DEBE presentarse en UIViewController apropiado.

[ ] TAC-9.12-IOS: App Transport Security DEBE permitir dominios del processor.

[ ] TAC-9.13-IOS: DEBE soportarse Apple Pay si el SDK lo ofrece.

[ ] TAC-9.14-IOS: Info.plist DEBE incluir NSCameraUsageDescription si hay scan.
```

### TACs Backend (Referencia)

```
[ ] TAC-9.15-BACKEND: DEBE implementarse 3DS2 para cumplir PSD2/SCA.

[ ] TAC-9.16-BACKEND: Eliminación de método DEBE invalidar token en processor.

[ ] TAC-9.17-BACKEND: Auditoría DEBE confirmar SAQ A-EP (no full PCI-DSS).

[ ] TAC-9.18-BACKEND: Webhooks del processor DEBEN estar firmados y verificados.
```

---

## 6. Estrategia de Pruebas (Shift-Left)

### Stack de Testing
- **Unit:** `flutter_test` para lógica de sanitización
- **Integration:** `integration_test` con tarjetas de prueba del processor
- **Security:** Verificar que PAN no aparece en ningún log

### Escenarios Críticos Obligatorios

| # | Escenario | Qué Validar | Plataforma | Tipo |
|:-:|:----------|:------------|:-----------|:-----|
| 1 | **PAN en logs** | Debug mode, agregar tarjeta, buscar en todos los logs. NO debe aparecer. | Flutter | Security |
| 2 | **3DS2 completo** | Tarjeta de prueba que requiere 3DS. Completar challenge exitosamente. | Android + iOS | E2E |
| 3 | **Eliminar método** | Token debe quedar invalidado en processor, no poder usarse | Flutter | Integration |
| 4 | **Red error durante tokenización** | Error graceful, no exponer datos parciales | Flutter | Integration |
| 5 | **Tarjeta rechazada** | Mensaje claro sin revelar detalles de seguridad | Flutter | Integration |

---

## Referencias

- [PCI-DSS Requirements](https://www.pcisecuritystandards.org/document_library)
- [Stripe Mobile SDKs](https://stripe.com/docs/mobile)
- [Adyen Mobile SDKs](https://docs.adyen.com/online-payments/build-your-integration/mobile)
- [PSD2 Strong Customer Authentication](https://ec.europa.eu/info/law/payment-services-psd-2-directive-eu-2015-2366_en)

---

*Anterior: [MFA SIM Swapping](caso-08-mfa-sim-swapping.md) | Siguiente: [Auditoría y Trazabilidad](caso-10-auditoria-trazabilidad.md)*
