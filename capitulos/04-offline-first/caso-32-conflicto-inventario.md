# Caso 32: Conflicto de Inventario
## Dos Vendedores, Un Producto, Cero Stock

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | inventario, conflicto, offline, consistencia, stock |
| **Patrón Técnico** | Optimistic Concurrency, Reservation Pattern, Conflict Resolution |
| **Stack Seleccionado** | Flutter + SQLite cola de cambios + Riverpod + políticas de merge |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como vendedor offline, quiero actualizar stock sin vender por encima de lo disponible."*

Ediciones concurrentes en stock offline pueden causar sobreventa si no hay reservas ni reconciliación adecuada.

### Evidencia de Industria

- **Retail omnicanal:** Sobreventa común sin reservas temporales.
- **Casos POS:** Ajustes de stock conflictivos generan pérdidas y devoluciones.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Sobreventa, devoluciones, pérdida de margen |
| **Reputacional** | Clientes sin producto, mala experiencia |
| **Técnico** | Datos inconsistentes difíciles de conciliar |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Actualizar stock local y sobrescribir | **INADECUADO:** Pierde cambios y permite sobreventa. |
| **ACEPTABLE** | LWW con timestamp, luego reconciliar manual | **MEJORA:** Menos conflicto, pero aún puede sobre-vender. |
| **ENTERPRISE** | **Reservas + políticas de merge:** reservas temporales, merge por delta, validación en backend, prompts si conflicto semántico | **ÓPTIMO:** Minimiza sobreventa, trazable y auditable. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Guardar deltas de stock con actor y timestamp. Aplicar reservas temporales en backend. Detectar conflictos y solicitar resolución (usuario o regla de dominio). Registrar merges para auditoría. |
| **Restricciones Duras (NO permite)** | **Sin backend de reservas:** El cliente solo sugiere, backend decide. **Offline largo:** Deltas pueden expirar o invalidarse. **Conflictos semánticos:** Algunos requieren revisión manual. |
| **Criterio de Selección** | Deltas en vez de valores absolutos para merges; políticas de reserva en backend; cola local para offline. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Reserva temporal | Bloqueo de stock por tiempo limitado hasta confirmar venta. |
| Delta de stock | Cambio incremental (+/-) aplicado al inventario. |
| LWW | Last Writer Wins; estrategia basada en timestamp. |
| Reconciliación | Ajustar diferencias entre cliente y backend. |
| Conflicto semántico | Regla de negocio que no se resuelve solo con timestamp. |

---

## Referencias

- [Event Sourcing y Deltas](https://martinfowler.com/eaaDev/EventSourcing.html)
- [Retail Inventory Conflicts](https://www.nngroup.com/articles/)
