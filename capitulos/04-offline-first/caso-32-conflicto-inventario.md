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

### Problema detectado (técnico)
- Deltas offline sin reservas ni merge determinista generan sobreventa y datos inconsistentes.
- LWW basado solo en timestamp descarta ediciones válidas y depende de reloj (skew).
- Sin log de conflictos/resolución, soporte y auditoría son difíciles.

### Escenario de Negocio

> *"Como vendedor offline, quiero actualizar stock sin vender por encima de lo disponible."*

### Incidentes reportados
- **Retail omnicanal:** Sobreventa común sin reservas temporales.
- **Casos POS:** Ajustes conflictivos generan pérdidas/devoluciones.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Retail omnicanal | Global | Sobreventa cuando no hay reservas/merge sólido. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; sync/merge es fuente de bugs. |
| Casos POS 2022 | Retail | Inventario desalineado por LWW sin contexto. |

**Resumen global**
- Sin reservas y merges con contexto, la sobreventa y desalineación son frecuentes.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Merges por delta y detección de conflictos | Equipo móvil, CI |
| Integration (CI) | Reservas temporales aplican; reconciliación backend-cliente consistente | Móvil/Backend, CI |
| Seguridad/consistencia | Log de conflictos/resolución sin PII innecesaria | QA/Seguridad |
| Observabilidad | Eventos `inventory.merge` con política aplicada | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Conflictos | Mostrar prompt/resumen cuando la regla de dominio no resuelve | Transparencia |
| Reservas | Indicar reservas temporales y expiración | Claridad operativa |
| Offline | Avisar expiración de deltas; permitir re-sync controlado | Controla expectativas |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Expiración | TTL para deltas/reservas; limpiar y notificar | Evita datos obsoletos |
| Política de merge | CRDT/OT o reglas por campo; LWW solo como fallback | Consistencia mejorada |
| Auditoría | Registrar merges y conflictos | Forense y soporte |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Ediciones concurrentes de stock offline causan sobreventa y datos inconsistentes. |
| Opciones evaluadas | Server-wins/client-wins; LWW; deltas + reservas + merge por dominio. |
| Decisión | Deltas + reservas temporales en backend + políticas de merge con contexto + prompts si aplica. |
| Consecuencias | Mayor complejidad de dominio y backend; requiere UX para conflictos. |
| Riesgos aceptados | Expiración de deltas; casos semánticos aún requieren intervención. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Sobreventa por conflicto | 0 incidentes | Crítico si > 0 | Evita pérdidas/devoluciones |
| Conflictos no resueltos | < 0.5% | Alerta si supera | Soporte manejable |
| Latencia de reconciliación | p95 < SLA | Warning si sube | Operación estable |
| Tickets por stock inconsistente | ↓ vs baseline | Alerta si no baja | Menos soporte |

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
