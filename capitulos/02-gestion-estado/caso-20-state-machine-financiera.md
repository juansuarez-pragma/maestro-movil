# Caso 20: State Machine Financiera
## Modelar el Ciclo de Vida de una Transferencia SPEI/ACH

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | state machine, transferencia SPEI, ACH, ciclo de vida, estados transaccionales |
| **Patrón Técnico** | Finite State Machine (FSM), Event-Driven Architecture, Retry/Dead-letter |
| **Stack Seleccionado** | Flutter + Riverpod StateNotifier + FSM declarativa + WebSockets/Push para eventos |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Flags sueltos/enum simple permiten combinaciones inválidas (“Completada” luego “En proceso”).
- Sin FSM + eventos firmados, no hay trazabilidad ni integridad; UI puede inventar estados.
- Sin retries/DLQ, fallos en estados transitorios quedan en limbo.

### Escenario de Negocio

> *"Como cliente, quiero saber exactamente en qué estado está mi transferencia y no ver estados imposibles."*

### Incidentes reportados
- **Bancos LATAM:** Reclamos por estados inconsistentes SPEI/ACH; se resolvieron con FSM estrictas y eventos firmados.
- **PCI/SOC2:** Exigen trazabilidad clara de transacciones financieras.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Reclamos en banca (LATAM) | Apps de pagos | Estados contradictorios generan tickets y desconfianza. |
| PCI/SOC2 auditorías | Global | Requieren historial claro de estados/eventos. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; modelado de estado es foco crítico en finanzas. |

**Resumen global**
- Estados inválidos erosionan confianza y disparan soporte.
- FSM declarativa con eventos firmados es práctica clave para trazabilidad en pagos.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Doble envíos o cancelaciones erróneas; soporte saturado |
| **Reputacional** | Usuarios pierden confianza si ven estados contradictorios |
| **Regulatorio** | Auditorías requieren historial claro de estados y eventos |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Flags sueltos en el modelo (isPending, isDone) | **INADECUADO:** Estados inválidos combinados, difícil de auditar. |
| **ACEPTABLE** | Enum simple con estados básicos | **MEJORA:** Reduce combinaciones inválidas, pero sin modelar eventos y transiciones condicionales. |
| **ENTERPRISE** | **FSM explícita:** estados + eventos + guardas; transiciones auditadas; retries con DLQ; actualización por eventos push | **ÓPTIMO:** Sin estados inválidos, trazabilidad completa, recuperable tras fallos. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Definir estados válidos (p. ej., Created → Validating → InProgress → Settled/Failed/Cancelled). Aplicar guardas (filtros AML, saldo) antes de transicionar. Registrar transiciones con timestamp y actor. Reintentar estados transitorios con backoff y mover a DLQ si fallan. Actualizar UI via push/WebSocket con eventos firmados. |
| **Restricciones Duras (NO permite)** | **Estados fuera de especificación:** Cualquier estado nuevo requiere desplegar FSM actualizada. **Dependencia backend:** La fuente de verdad es el backend; el cliente no puede inventar estados. **Reloj:** Timestamps deben venir del backend para trazabilidad. |
| **Criterio de Selección** | FSM declarativa en cliente para validar y mostrar estados coherentes; Riverpod StateNotifier para exponer estado derivado; eventos firmados para integridad; DLQ para manejo de fallos en reintentos. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | FSM solo permite transiciones válidas; guardas funcionan | Equipo móvil, CI |
| Integration (CI) | Eventos push actualizan FSM y auditan transiciones | Móvil/Backend, CI |
| Resiliencia | Retries con backoff y DLQ para estados transitorios | QA/Seguridad |
| Observabilidad | Evento `transfer.state_change` con firma, trace_id | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Visibilidad | Mostrar estado actual y último evento; evitar saltos atrás | Confianza del usuario |
| Errores | Mensajes claros en fallos/cancelaciones con causa | Transparencia |
| Sincronización | Usar timestamps backend; resolver conflictos a favor de backend | Trazabilidad coherente |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| DLQ y retries | Backoff y DLQ para eventos fallidos | Evita limbo |
| Integridad | Firmas/verificación de eventos | Asegura origen/alteración |
| Auditoría | Log de transiciones con actor/timestamp/trace_id | Cumplimiento |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Estados invalidos/contradictorios por flags/enum simple. |
| Opciones evaluadas | Flags sueltos; enum plano; FSM con eventos firmados y DLQ. |
| Decisión | FSM explícita + guardas + eventos firmados + DLQ/retry. |
| Consecuencias | Más modelado y coordinación con backend; requiere manejo de eventos perdidos. |
| Riesgos aceptados | Consistencia eventual por asincronía; dependencia de backend para verdad. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Estados inválidos/contradictorios | 0 incidentes | Crítico si > 0 | Confianza y cumplimiento |
| Latencia de actualización de estado | p95 < 1 s con push | Warning si sube | Transparencia al usuario |
| Retries/DLQ exitosos | > 99% de recuperación de transitorios | Alerta si baja | Resiliencia |
| Tickets por estado erróneo | ↓ vs baseline | Alerta si no baja | Soporte controlado |
| Auditoría de transiciones | 100% con firma/trace_id | Crítico si falta | Cumplimiento PCI/SOC2 |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| FSM | Máquina de estados finitos con transiciones y eventos definidos. |
| Guard | Condición que debe cumplirse para permitir una transición. |
| DLQ (Dead-letter Queue) | Cola de mensajes que no pudieron procesarse tras reintentos. |
| SPEI/ACH | Sistemas de pagos interbancarios (MX/US). |
| Evento firmado | Mensaje con firma/verificación para asegurar integridad y origen. |
| Traceability | Capacidad de reconstruir el historial de estados con evidencia verificable. |

---

## Referencias

- [FSM Patterns](https://martinfowler.com/articles/uiState.html)
- [ACH Operating Rules](https://www.nacha.org/rules)
- [SPEI - Banxico Specs](https://www.banxico.org.mx/sistemas-de-pago/que-es-el-spei.html)
