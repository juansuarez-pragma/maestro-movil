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

### Escenario de Negocio

> *"Como oficial de cumplimiento, necesito saber qué empleado realizó qué acción, cuándo, desde qué dispositivo, y tener evidencia inmutable para investigaciones."*

Apps internas (backoffice, call center, operaciones) manejan operaciones sensibles en nombre de clientes. Sin audit trail robusto, es imposible investigar incidentes o probar/refutar alegaciones.

### Evidencia de Industria

**Caso Wells Fargo (2016):** 3.5 millones de cuentas fraudulentas creadas por empleados presionados por metas. La falta de auditoría granular permitió que el fraude continuara por años antes de ser detectado.

**Caso Société Générale (2008):** Un trader causó pérdidas de €4.9 mil millones. Los controles de auditoría no detectaron las anomalías en tiempo real.

**Estadística ACFE 2022:** 5% del revenue anual se pierde por fraude. 42% del fraude ocupacional es cometido por empleados. Tiempo promedio de detección: 12 meses.

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

---

*Anterior: [PCI-DSS Tokenización](caso-09-pci-dss-tokenizacion.md)*

---

# FIN DEL CAPÍTULO 1

**Siguiente Capítulo:** [Gestión de Estado Compleja](../02-gestion-estado/README.md)
