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

> *"Como oficial de cumplimiento, necesito saber qué empleado realizó qué acción, cuándo, y desde qué dispositivo."*

Apps internas (backoffice, call center) manejan operaciones sensibles. Sin audit trail, imposible investigar incidentes.

### Evidencia de Industria

**Caso Wells Fargo (2016):** 3.5 millones de cuentas fraudulentas creadas por empleados. Falta de auditoría granular permitió que continuara años.

**Caso Société Générale (2008):** Trader causó pérdidas de €4.9B. Controles no detectaron anomalías.

**Estadística ACFE:** 5% del revenue perdido por fraude, 42% por empleados.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Regulatorio** | Reguladores exigen audit trails |
| **Legal** | Sin evidencia, imposible probar/refutar |
| **Operacional** | Fraude interno no detectado |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Rol | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:----|:-----------------------|:----------------------------------|
| **Junior** | Logs servidor con timestamp y user_id | **FALLA:** Modificables, sin contexto dispositivo, sin hash integridad. |
| **Senior** | Logs estructurados + ELK | **MEJORA:** Centralización dificulta tampering. PERO: admin puede modificar, sin log local. |
| **Architect** | **Audit Trail Inmutable:** Log local firmado con hash encadenado, sync server con verificación, WORM storage | **ENTERPRISE:** Tamper-evident. Evidencia legal. Cumple SOX, GDPR. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Registrar cada acción con contexto completo. Hash encadenado detecta manipulación. Sync eventual. Consulta local. Exportación para auditorías. |
| **Restricciones Duras (NO permite)** | **Borrado:** Solo archivar con retención. **Modificación:** Rompe cadena de hashes. **Offline extendido:** Forzar sync o bloquear. |
| **Criterio de Selección** | Se usa **drift** para log local por queries, integridad transaccional. Se usa **BLoC** porque cada evento auditable debe pasar por punto central. |

---

## 4. Manos a la Obra: Estrategia de Implementación

### Fase 1: Estructura del Audit Event

**Campos:**
- `event_id`: UUID
- `timestamp`: ISO 8601
- `user_id`: ID empleado
- `session_id`: Sesión activa
- `device_id`: Fingerprint
- `action_type`: VIEW, CREATE, UPDATE, DELETE, APPROVE, REJECT
- `resource_type`: ACCOUNT, TRANSACTION, USER, CONFIG
- `resource_id`: ID específico
- `action_details`: JSON detalles
- `ip_address`: IP
- `location`: Coordenadas
- `previous_hash`: Hash evento anterior
- `event_hash`: SHA-256 de todo

### Fase 2: Implementación

**AuditBloc:**
- Cada feature emite eventos via AuditBloc
- Evento: `AuditEventRequested(actionType, resourceType, resourceId, details)`
- BLoC: enriquecer con contexto, calcular hash, guardar, encolar sync

**LocalAuditRepository (drift):**
- Tabla `audit_events`
- Insert-only: no UPDATE, no DELETE
- Índices en timestamp, user_id, action_type
- `verifyChain()`: recalcular y verificar hashes

**AuditSyncService:**
- Background job cada X minutos
- Batch de eventos a backend
- Backend verifica hashes
- Mark as synced si confirma

**TamperDetection:**
- Al iniciar: `verifyChain()`
- Si inconsistencia: alertar, bloquear operaciones

---

## 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

```
[ ] TAC-10.1: TODA acción de negocio DEBE generar audit event.
[ ] TAC-10.2: Evento DEBE incluir: timestamp, user_id, device_id, action_type,
    resource_type, resource_id, details.
[ ] TAC-10.3: Eventos DEBEN encadenarse con hash criptográfico.
[ ] TAC-10.4: Log local DEBE ser insert-only, SIN UPDATE/DELETE.
[ ] TAC-10.5: DEBE sincronizar con backend que verifica integridad.
[ ] TAC-10.6: Si manipulación detectada, ALERTAR y BLOQUEAR.
[ ] TAC-10.7: Logs exportables para auditorías externas.
[ ] TAC-10.8: Retención mínima 7 años.
[ ] TAC-10.9: Backend DEBE usar almacenamiento WORM.
[ ] TAC-10.10: Búsqueda por user_id, fecha, action_type.
```

---

## 6. Estrategia de Pruebas (Shift-Left)

| # | Escenario | Qué Validar | Tipo |
|:-:|:----------|:------------|:-----|
| 1 | **Modificar DB local** | App detecta chain rota, alerta, bloquea | Security |
| 2 | **Operación offline → sync** | Eventos sincronizados con hashes correctos | Integration |
| 3 | **Query de investigación** | Dado user_id y fechas, obtener acciones completas | Integration |

---

## Referencias

- [NIST SP 800-92 - Guide to Computer Security Log Management](https://csrc.nist.gov/publications/detail/sp/800-92/final)
- [SOX Compliance Requirements](https://www.sec.gov/spotlight/sarbanes-oxley.htm)
- [GDPR Article 30 - Records of Processing Activities](https://gdpr-info.eu/art-30-gdpr/)

---

*Anterior: [PCI-DSS Tokenización](caso-09-pci-dss-tokenizacion.md)*

---

# FIN DEL CAPÍTULO 1

**Siguiente Capítulo:** [Gestión de Estado Compleja](../02-gestion-estado/README.md)
