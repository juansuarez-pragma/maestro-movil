# Caso 45: Request Deduplication
## Evitar Doble Cobro por Doble Tap

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | deduplicación, doble tap, requests duplicados, idempotencia |
| **Patrón Técnico** | In-flight Request Deduplication, Idempotent Client, Mutex per Endpoint |
| **Stack Seleccionado** | Flutter + Dio interceptors + in-flight map + Idempotency-Key |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, un doble tap no debe generar dos cobros."*

Sin deduplicación, taps repetidos o reintentos simultáneos causan cobros duplicados y errores de UX.

### Evidencia de Industria

- **Casos de pagos móviles:** Doble tap = doble cobro sin protección de idempotencia.
- **Guidelines de PSPs:** Recomiendan Idempotency-Key y dedup cliente/servidor.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Cobros duplicados, reembolsos costosos |
| **UX** | Mensajes de error inconsistentes; confusión del usuario |
| **Técnico** | Requests redundantes saturan el backend |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Deshabilitar botón en UI solamente | **INADECUADO:** No cubre multi-requests programáticos ni reintentos. |
| **ACEPTABLE** | Throttle/Debounce de acciones | **MEJORA:** Reduce taps, pero no elimina duplicados in-flight. |
| **ENTERPRISE** | **Dedup + idempotencia:** mantener mapa de requests en vuelo, rechazar duplicados, usar Idempotency-Key en pagos | **ÓPTIMO:** Previene cobros duplicados y reduce carga. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Identificar requests equivalentes por endpoint/payload. Reusar la misma future/respuesta para duplicados. Adjuntar Idempotency-Key para pagos. Cancelar duplicados opcionales. |
| **Restricciones Duras (NO permite)** | **Mutaciones no idempotentes:** Backend debe soportar. **Bindeo excesivo:** Dedup agresivo puede bloquear solicitudes legítimas si el payload difiere. **Estado compartido:** Dedup es local al cliente; múltiples dispositivos aún requieren idempotencia server-side. |
| **Criterio de Selección** | Interceptor central para dedup; claves basadas en método+path+payload hash; Idempotency-Key en pagos; UI también deshabilita acciones críticas. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| In-flight dedup | Reutilizar la misma respuesta para requests concurrentes iguales. |
| Idempotency-Key | Token único que evita doble procesamiento en backend. |
| Throttle/Debounce | Limitar frecuencia de acciones/requests. |
| Payload hash | Hash del cuerpo/payload para identificar duplicados. |
| Mutex | Exclusión mutua para serializar requests iguales. |

---

## Referencias

- [Stripe Idempotency](https://stripe.com/docs/idempotency)
- [Dio Interceptors](https://pub.dev/packages/dio)
