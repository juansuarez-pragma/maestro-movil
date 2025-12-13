# Caso 10: El Empleado Deshonesto
## Auditoría y Trazabilidad de Acciones en Apps Internas

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | auditoría, trazabilidad, log de acciones, compliance, insider threat, app interna, backoffice |
| **Patrón Técnico** | Audit Trail, Event Sourcing, Immutable Logs, Chain of Custody |
| **Stack Seleccionado** | Dio Interceptors + drift (SQLite) para log local + BLoC (AuditBloc) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Logs solo en backend o modificables por admins permiten tampering; sin hash encadenado/inmutabilidad, la evidencia no es confiable.
- Sin correlación (trace_id/user_id/device_id) y contexto completo, las investigaciones son incompletas.
- Falta de sync offline → gaps de auditoría; sin retención clara, se pierde evidencia o se expone privacidad.

### Escenario de Negocio

> *"Como oficial de cumplimiento, necesito saber qué empleado realizó qué acción, cuándo, desde qué dispositivo, y tener evidencia inmutable para investigaciones."*

Apps internas manejan operaciones sensibles. Sin audit trail robusto, es imposible investigar o defenderse en disputas/regulación.

### Incidentes reportados

- **Wells Fargo (2016):** 3.5M cuentas fraudulentas creadas; falta de auditoría granular permitió fraude prolongado.
- **Société Générale (2008):** Trader causó pérdidas de €4.9B; controles no detectaron anomalías a tiempo.
- **ACFE 2022:** 5% del revenue se pierde por fraude; 42% cometido por empleados; detección promedio: 12 meses.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| ACFE Report to the Nations 2022 | Global | 5% del revenue se pierde por fraude ocupacional; 42% perpetrado por empleados. |
| Verizon DBIR 2023 | Global | Brechas internas y manipulación de datos son foco creciente; necesidad de logs con integridad. |
| Auditorías SOX/GDPR | Global | Requieren registros completos y trazables; falta de audit trail es hallazgo recurrente. |

**Resumen global**
- Fraude interno causa pérdidas significativas y tarda en detectarse (~12 meses).
- Reguladores exigen evidencia trazable e inmutable; falta de integridad en logs es hallazgo común.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Regulatorio** | Reguladores (SOX, GDPR Art. 30) exigen audit trails completos |
| **Legal** | Sin evidencia, imposible probar o refutar alegaciones en disputas |
| **Operacional** | Fraude interno no detectado, pérdidas continuadas |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Logs de servidor con timestamp y user_id | **INADECUADO:** Modificables por admins, sin contexto de dispositivo, sin verificación de integridad, gaps si hay errores de red. |
| **ACEPTABLE** | Logs estructurados + ELK/Splunk centralizados | **CUMPLE MÍNIMOS:** Centralización dificulta tampering. PERO: admin de infra puede modificar, sin log local como backup, sin hash de integridad. |
| **ENTERPRISE** | **Audit Trail Inmutable:** Log local firmado con hash encadenado, sync a servidor con verificación, almacenamiento WORM en backend | **ÓPTIMO:** Tamper-evident (cualquier modificación rompe cadena). Evidencia válida legalmente. Cumple SOX, GDPR. Resistente a manipulación. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Registrar cada acción con contexto completo (quién, qué, cuándo, dónde, cómo). Hash encadenado detecta cualquier manipulación. Sync eventual con reintentos. Consulta local para investigaciones inmediatas. Exportación para auditorías externas. |
| **Restricciones Duras (NO permite)** | **Borrado:** Eventos solo se pueden archivar con política de retención, nunca borrar. **Modificación:** Cualquier cambio rompe cadena de hashes. **Offline extendido:** Forzar sync periódico o bloquear operaciones sensibles. **Storage local:** Límite de espacio en dispositivo. |
| **Criterio de Selección** | Se usa **drift** (SQLite) para log local porque soporta queries complejos, integridad transaccional, y es más robusto que archivos. Se usa **BLoC** porque cada evento auditable debe pasar por punto central y el flujo tiene estados (logging→syncing→synced). |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | BLoC registra evento con hash encadenado y estado de sync | Equipo móvil, CI |
| Integration (CI) | Sync con backend valida hash chain y rechaza tampering; reintentos ante fallos de red | Móvil/Backend, CI + staging |
| Seguridad manual | Intento de modificar registros locales rompe cadena; UI alerta inconsistencia | Seguridad/QE, dispositivos reales |
| Observabilidad | Evento `audit.sync` y `audit.tamper_detected` con trace_id | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Visibilidad | Mostrar estado de sync para auditores internos; permitir exportar lote cifrado | Transparencia en auditoría |
| Privacidad | Pseudoanonimizar PII; registrar solo lo necesario | Cumplimiento GDPR/privacidad |
| Offline | Permitir captura offline pero forzar sync periódico; bloquear operaciones sensibles si no sync | Evita gaps prolongados |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Retención | Definir TTL y archivado (WORM en backend) | Cumple requerimientos regulatorios |
| Integridad | Hash chain + firma/clave app para cada evento | Evidencia tamper-evident |
| Correlación | Incluir trace_id, device_id, user_id pseudoanon, timestamp preciso | Facilita investigación |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Logs modificables/no correlacionados no sirven como evidencia. |
| Opciones evaluadas | Logs backend simples; ELK sin hash; hash chain local + sync + WORM backend. |
| Decisión | Log local con hash chain + sync validado + almacenamiento WORM. |
| Consecuencias | Mayor complejidad de sync y almacenamiento; manejo de espacio local. |
| Riesgos aceptados | Dispositivos con poco espacio; dependencia de conectividad para sync. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Integridad de logs | 0 eventos de tampering aceptados | Crítico si hash chain falla | Evidencia confiable |
| Cobertura de eventos auditables | 100% de acciones sensibles logueadas | Alerta si faltan eventos | Cumplimiento y trazabilidad |
| Sync a backend | p95 de sync < 5 min; reintentos automáticos | Alerta si backlog crece | Minimiza gaps |
| Retención | Cumplir política (ej. 1-5 años según regulación) | Alerta si expira sin archivar | Evita pérdida de evidencia |
| Incidentes internos detectados | Tiempo de detección ↓ vs baseline | Alertar si no mejora | Reduce pérdidas por fraude interno |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Auditoría | Registro fiable y ordenado de eventos para investigación forense y cumplimiento. |
| Trazabilidad | Capacidad de seguir el ciclo completo de una operación (quién, qué, cuándo, desde dónde). |
| Correlación | Enlazar eventos con ids comunes (trace_id, session_id, user_id pseudoanon) para reconstruir flujos. |
| Retención | Política de cuánto tiempo se almacenan logs/eventos según regulación y costo. |
| Inmutabilidad | Garantía de que los registros no han sido alterados (WORM, firmas, hash chain). |
| PII Minimization | Registrar solo lo necesario, pseudoanonimizar datos para cumplir privacidad. |

---

## Referencias

- [NIST SP 800-92 - Guide to Computer Security Log Management](https://csrc.nist.gov/publications/detail/sp/800-92/final)
- [SOX Compliance Requirements](https://www.sec.gov/spotlight/sarbanes-oxley.htm)
- [GDPR Article 30 - Records of Processing Activities](https://gdpr-info.eu/art-30-gdpr/)
- [ACFE Report to the Nations on Occupational Fraud](https://www.acfe.com/report-to-the-nations/2022/)
- [Verizon DBIR 2023](https://www.verizon.com/business/resources/reports/dbir/)
- [ACFE Report 2022](https://acfepublic.s3-us-west-2.amazonaws.com/2022-Report-to-the-Nations.pdf)

---

*Anterior: [PCI-DSS Tokenización](caso-09-pci-dss-tokenizacion.md)*

---

# FIN DEL CAPÍTULO 1

**Siguiente Capítulo:** [Gestión de Estado Compleja](../02-gestion-estado/README.md)
