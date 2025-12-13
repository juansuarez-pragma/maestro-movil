# Caso 37: El Pedido Duplicado
## Idempotencia en Retry de Operaciones Críticas

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | idempotencia, retries, pedidos duplicados, operaciones críticas |
| **Patrón Técnico** | Idempotent Operations, Retry with Backoff, Idempotency-Key |
| **Stack Seleccionado** | Flutter + Dio interceptors + Idempotency-Key + Riverpod estado de pedidos |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Retries por fallas de red duplican pedidos/cobros si no hay idempotencia end-to-end.
- Sin `Idempotency-Key` persistida y reuse, reintentos simultáneos generan doble operación.
- Falta de trazabilidad/telemetría oculta duplicados y complica soporte.

### Escenario de Negocio

> *"Como usuario, si reintento una compra, no quiero que se genere doble pedido o cobro."*

### Incidentes reportados
- **PSP/fintech:** Idempotency-Key es estándar para pagos/órdenes.
- **Incidentes e-commerce:** Doble pedido por reintentos simultáneos o timeouts.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| PSPs (Stripe/Adyen) | Global | Idempotencia requerida para operaciones financieras. |
| Casos e-commerce | Retail | Duplicados por reintentos simultáneos/timeouts. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; idempotencia/errores es hallazgo común. |

**Resumen global**
- Idempotencia es práctica estándar; sin ella, duplicados impactan fraude/soporte.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Doble cobro/pedido, reembolsos costosos |
| **Reputacional** | Pérdida de confianza del cliente |
| **Técnico** | Inconsistencia entre cliente y backend |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Retries sin control ni idempotencia | **INADECUADO:** Riesgo alto de duplicados. |
| **ACEPTABLE** | Limitar retries y deshabilitar UI | **MEJORA:** Menos duplicados, pero no garantiza idempotencia. |
| **ENTERPRISE** | **Idempotency end-to-end:** clave única por operación, retries seguros, validación en backend, UI coherente | **ÓPTIMO:** Evita duplicados y mantiene consistencia. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Generar Idempotency-Key por operación y persistirla en cola de acciones. Reusar la clave en reintentos. UI refleja estado pendiente/confirmado. Limpiar clave al concluir. Telemetría de intentos. |
| **Restricciones Duras (NO permite)** | **Sin soporte backend:** Idempotencia real requiere servidor. **Claves reusadas:** Deben ser únicas; evitar colisiones. **Expiración:** Backend puede expirar claves; manejar errores claros. |
| **Criterio de Selección** | Interceptor para agregar clave; Riverpod para estado de pedidos; backoff con límites; observabilidad de duplicados. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Se genera/persiste clave única por pedido; reintentos reusan clave | Equipo móvil, CI |
| Integration (CI) | Reintentos con misma key → un solo pedido/cargo | Móvil/Backend, CI |
| Seguridad/consistencia | Manejo de expiración/colisión de claves; UI refleja estados | QA/Seguridad |
| Observabilidad | Eventos `order.idempotent` con `idempotency_key` y resultado | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Estado de operación | Mostrar pendiente/confirmado; evitar múltiples taps | Reduce duplicados |
| Reintentos | Backoff con misma key; CTA “reintentar” seguro | UX clara |
| Fallback | Si backend expira key, solicitar nueva operación claramente | Transparencia |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Persistencia | Guardar clave en cola para reintentos/crash recovery | Confiabilidad |
| Colisiones | Generar claves únicas (UUID + contexto); validar en backend | Evita falsos duplicados |
| Límite | Respetar TTL/limitaciones del PSP | Cumplimiento |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Reintentos duplican pedidos/cobros sin idempotencia. |
| Opciones evaluadas | Deshabilitar UI; throttling; idempotencia end-to-end con keys persistentes. |
| Decisión | Idempotency-Key única por operación, persistida y reutilizada en reintentos + validación backend. |
| Consecuencias | Requiere soporte backend; manejo de expiración/colisiones; más telemetría. |
| Riesgos aceptados | Si backend no soporta, riesgo residual; keys caducan según PSP. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Pedidos/cobros duplicados | 0 | Crítico si > 0 | Confianza y menos reembolsos |
| Reintentos seguros | 100% reusan la misma key | Alerta si falla | Consistencia |
| Tickets por doble cargo | ↓ vs baseline | Alerta si no baja | Soporte controlado |
| Latencia de confirmación | En línea con SLA | Warning si sube | UX aceptable |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Idempotencia | Repetir una operación produce el mismo resultado sin duplicarla. |
| Idempotency-Key | Identificador único por operación para deduplicar en backend. |
| Backoff | Espera creciente entre reintentos. |
| Pedido crítico | Operación que no debe duplicarse (pagos, órdenes). |
| Retry seguro | Reintento que preserva idempotencia y consistencia. |

---

## Referencias

- [Stripe Idempotency](https://stripe.com/docs/idempotency)
- [AWS Architecture - Backoff](https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/)
