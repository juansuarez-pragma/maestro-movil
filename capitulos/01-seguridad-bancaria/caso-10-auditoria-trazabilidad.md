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

## 4. Manos a la Obra: Estrategia de Implementación

### Justificación del Plan

La estrategia se deriva del análisis de Wells Fargo y los requisitos de SOX/GDPR:

1. **Logs modificables no sirven como evidencia** → Hash encadenado inmutable
2. **Admins no deben poder alterar** → Verificación criptográfica
3. **Red puede fallar** → Log local primero, sync después
4. **Investigaciones necesitan contexto** → Metadata rica en cada evento

La arquitectura usa BLoC como punto central de auditoría, drift para almacenamiento local con integridad, y sync batch al backend.

---

### Fase 1: Diseño — Estructura del Audit Event

**Campos del Evento:**
- `event_id`: UUID v4 único
- `timestamp`: ISO 8601 con timezone (generado en cliente)
- `server_timestamp`: Timestamp del servidor (para correlación)
- `user_id`: ID del empleado autenticado
- `session_id`: Sesión activa (para correlacionar acciones)
- `device_id`: Fingerprint del dispositivo
- `action_type`: VIEW, CREATE, UPDATE, DELETE, APPROVE, REJECT, EXPORT, LOGIN, LOGOUT
- `resource_type`: ACCOUNT, TRANSACTION, USER, CONFIG, REPORT
- `resource_id`: ID específico del recurso afectado
- `action_details`: JSON con detalles específicos de la acción
- `ip_address`: IP del dispositivo
- `location`: Coordenadas GPS si disponibles
- `previous_hash`: Hash SHA-256 del evento anterior
- `event_hash`: SHA-256 de todos los campos anteriores concatenados

**Estructura de Carpetas:**
```
lib/core/audit/
├── data/
│   ├── datasources/
│   │   ├── audit_local_datasource.dart
│   │   └── audit_remote_datasource.dart
│   ├── models/
│   │   └── audit_event_model.dart
│   └── repositories/
│       └── audit_repository_impl.dart
├── domain/
│   ├── entities/
│   │   └── audit_event.dart
│   └── usecases/
│       ├── log_audit_event.dart
│       ├── sync_audit_events.dart
│       └── verify_audit_chain.dart
└── presentation/
    └── bloc/
        ├── audit_bloc.dart
        ├── audit_event.dart
        └── audit_state.dart
```

---

### Fase 2: Implementación por Plataforma

#### 2.1 Flutter (Cross-Platform) — Capa de Aplicación

**AuditBloc:**
- Punto central: cada feature emite `AuditEventRequested`
- Evento: `AuditEventRequested(actionType, resourceType, resourceId, details)`
- BLoC: enriquece con contexto (user, device, session), calcula hash, guarda local, encola sync
- Estados: `AuditReady`, `AuditLogging`, `AuditSyncing`, `AuditError`

**AuditEventEnricher:**
- Obtener user_id del AuthBloc
- Obtener device_id del DeviceInfoProvider
- Obtener session_id del SessionManager
- Obtener location si permisos disponibles
- Obtener IP (puede requerir service externo)

**HashChainCalculator:**
- Obtener último evento local
- Concatenar campos del nuevo evento + previous_hash
- Calcular SHA-256
- Verificar que no existe gap en sequence

**LocalAuditRepository (drift):**
- Tabla `audit_events` con todos los campos
- Insert-only: método `insertEvent`, NO métodos update/delete
- Índices en: timestamp, user_id, action_type, synced
- `getUnsyncedEvents()`: Eventos pendientes de sync
- `markAsSynced(eventIds)`: Marcar como sincronizados
- `verifyChain()`: Recalcular y verificar todos los hashes

---

#### 2.2 Android — Consideraciones Específicas

**SQLite Security Android:**
- SQLCipher opcional para cifrado de DB local
- Configurar `android:allowBackup="false"` para evitar extracción
- Room/drift almacena en `data/data/package/databases/`

**Device Fingerprint Android:**
- `Settings.Secure.ANDROID_ID`
- Combinar con `Build.MODEL`, `Build.MANUFACTURER`
- Hash SHA-256 del conjunto

**Location Android:**
- `ACCESS_FINE_LOCATION` si se requiere GPS
- Usar `FusedLocationProviderClient` para eficiencia
- Fallback a IP geolocation si GPS no disponible

**Consideraciones Android:**
- Verificar espacio disponible antes de logging intensivo
- WorkManager para sync en background
- Battery optimization: usar constraints de red

---

#### 2.3 iOS — Consideraciones Específicas

**SQLite Security iOS:**
- Core Data o drift almacena en sandbox de app
- Configurar `NSFileProtectionComplete` para cifrado
- Excluir de backup de iCloud

**Device Fingerprint iOS:**
- `identifierForVendor` (persiste mientras app instalada)
- Combinar con modelo de dispositivo
- Hash SHA-256 del conjunto

**Location iOS:**
- `NSLocationWhenInUseUsageDescription` requerido
- `CLLocationManager` para coordenadas
- `CLGeocoder` para reverse geocoding si necesario

**Consideraciones iOS:**
- Background App Refresh para sync
- `BGTaskScheduler` para sync diferido
- Límites de tiempo en background

---

### Fase 3: Observability — Métricas y Alertas

**Métricas:**
- `audit.events.logged` (por action_type)
- `audit.events.synced` / `audit.events.sync_failed`
- `audit.chain.verified` / `audit.chain.broken`
- `audit.storage.used_mb`

**Alertas:**

| Evento | Severidad | Acción |
|:-------|:----------|:-------|
| `audit.chain.broken` | P1 Critical | Incidente de seguridad, investigar manipulación |
| `audit.sync_failed` > 1 hora | P2 High | Verificar conectividad, investigar |
| Acción anómala (ej: DELETE masivo) | P2 High | Alertar a seguridad |

---

## 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

### TACs Flutter (Cross-Platform)

```
[ ] TAC-10.1-FLUTTER: TODA acción de negocio DEBE generar audit event.

[ ] TAC-10.2-FLUTTER: Evento DEBE incluir: timestamp, user_id, device_id,
    session_id, action_type, resource_type, resource_id, details.

[ ] TAC-10.3-FLUTTER: Eventos DEBEN encadenarse con hash SHA-256 criptográfico.

[ ] TAC-10.4-FLUTTER: Log local DEBE ser insert-only, SIN métodos UPDATE/DELETE.

[ ] TAC-10.5-FLUTTER: DEBE sincronizar con backend con reintentos exponenciales.

[ ] TAC-10.6-FLUTTER: Si cadena de hashes rota, ALERTAR y potencialmente BLOQUEAR.

[ ] TAC-10.7-FLUTTER: Búsqueda local por user_id, fecha, action_type.
```

### TACs Android

```
[ ] TAC-10.8-ANDROID: DB local DEBE excluirse de Android backup.

[ ] TAC-10.9-ANDROID: Sync DEBE usar WorkManager con constraints de red.

[ ] TAC-10.10-ANDROID: Device fingerprint DEBE incluir ANDROID_ID + Build info.

[ ] TAC-10.11-ANDROID: DEBE manejar límites de storage local.
```

### TACs iOS

```
[ ] TAC-10.12-IOS: DB local DEBE tener NSFileProtectionComplete.

[ ] TAC-10.13-IOS: DB DEBE excluirse de iCloud backup.

[ ] TAC-10.14-IOS: Sync DEBE usar BGTaskScheduler para background.

[ ] TAC-10.15-IOS: Device fingerprint DEBE usar identifierForVendor.
```

### TACs Backend (Referencia)

```
[ ] TAC-10.16-BACKEND: DEBE verificar integridad de hash chain al recibir eventos.

[ ] TAC-10.17-BACKEND: DEBE usar almacenamiento WORM (Write Once Read Many).

[ ] TAC-10.18-BACKEND: Logs exportables en formato estándar para auditorías.

[ ] TAC-10.19-BACKEND: Retención mínima 7 años para cumplir regulaciones.

[ ] TAC-10.20-BACKEND: DEBE existir endpoint de búsqueda para investigaciones.
```

---

## 6. Estrategia de Pruebas (Shift-Left)

### Stack de Testing
- **Unit:** `flutter_test` para hash calculation, chain verification
- **Integration:** `integration_test` con drift in-memory DB
- **Security:** Intentar modificar DB local, verificar detección

### Escenarios Críticos Obligatorios

| # | Escenario | Qué Validar | Plataforma | Tipo |
|:-:|:----------|:------------|:-----------|:-----|
| 1 | **Modificar DB local** | App detecta chain rota, alerta, potencialmente bloquea | Flutter | Security |
| 2 | **Operación offline → sync** | Eventos sincronizados con hashes correctos, orden preservado | Flutter | Integration |
| 3 | **Query de investigación** | Dado user_id y rango de fechas, obtener todas las acciones | Flutter | Integration |
| 4 | **Gap en sequence** | Detectar si faltan eventos en la cadena | Flutter | Unit |
| 5 | **Storage lleno** | Manejar gracefully, alertar, no perder eventos | Android + iOS | Integration |

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
