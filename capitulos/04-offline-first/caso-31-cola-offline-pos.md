# Caso 31: Vender en el Metro
## Cola de Transacciones Offline para POS Móvil

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | POS offline, cola de transacciones, pagos sin red, sincronización diferida |
| **Patrón Técnico** | Offline Queue, Store-and-Forward, Idempotent Operations |
| **Stack Seleccionado** | Flutter + SQLite/Isar para cola + Riverpod para orquestar + retry/backoff |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como vendedor, necesito cobrar en el metro sin conexión y sincronizar cuando vuelva la red."*

Sin una cola robusta, los cobros offline se pierden o se duplican al reconectarse.

### Evidencia de Industria

- **POS móviles:** Store-and-forward es estándar para zonas sin cobertura.
- **Incidentes retail:** Pérdida de ventas por fallos de sincronización offline→online.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Ventas perdidas o doble cobro al reintentar |
| **Reputacional** | Clientes sin recibo o con cargos duplicados |
| **Técnico** | Divergencia de inventario y conciliación compleja |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Guardar en memoria y reintentar manual | **INADECUADO:** Se pierden datos al cerrar app. |
| **ACEPTABLE** | Cola local simple con retries | **MEJORA:** Persiste, pero sin idempotencia ni control de orden. |
| **ENTERPRISE** | **Queue persistente + idempotencia:** almacenar payloads con Idempotency-Key, retries con backoff, confirmación del backend, reconciliación post-sync | **ÓPTIMO:** Minimiza pérdidas y duplicados, trazable. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Persistir transacciones en SQLite/Isar con estado (pending/sent/confirmed/failed). Reintentar con backoff y límite. Adjuntar Idempotency-Key para evitar dobles cargos. Reconstruir recibos tras sync. |
| **Restricciones Duras (NO permite)** | **Autorización real offline:** Solo pagos diferidos; requiere aceptación de riesgo. **Conflictos de stock:** Resolver en backend. **TTL de cola:** Transacciones expiran según políticas de adquirente. |
| **Criterio de Selección** | Queue persistente para confiabilidad; idempotencia para seguridad; Riverpod para exponer estado de cola y progreso. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Store-and-Forward | Guardar transacción local y enviarla cuando haya red. |
| Idempotency-Key | Identificador único para evitar doble procesamiento. |
| Backoff | Aumentar tiempo entre reintentos para reducir carga. |
| Reconciliación | Ajustar registros tras sincronizar con backend. |
| TTL | Tiempo de vida de una transacción antes de caducar. |

---

## Referencias

- [Stripe Idempotency](https://stripe.com/docs/idempotency)
- [NIST Offline Transactions Guidance](https://csrc.nist.gov/)
- [SQLite Best Practices](https://www.sqlite.org/cli.html)
