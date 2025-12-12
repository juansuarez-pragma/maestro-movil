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

### Escenario de Negocio

> *"Como cliente, quiero saber exactamente en qué estado está mi transferencia y no ver estados imposibles."*

Sin un modelo de estados bien definido, la UI muestra combinaciones inválidas (ej. "Completada" y luego "En proceso") y la trazabilidad se pierde.

### Evidencia de Industria

- **Bancos LATAM:** Reclamos por estados inconsistentes de SPEI/ACH; se resolvieron con FSM estrictas y eventos firmados.
- **PCI/SOC2:** Exigen trazabilidad clara de transacciones financieras.

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

---

## Referencias

- [FSM Patterns](https://martinfowler.com/articles/uiState.html)
- [ACH Operating Rules](https://www.nacha.org/rules)
- [SPEI - Banxico Specs](https://www.banxico.org.mx/sistemas-de-pago/que-es-el-spei.html)
