# Caso 17: [Wizard](#term-wizard "Flujo multi-paso guiado para completar un proceso largo.") de 12 Pasos
## Persistir Progreso de Solicitud de Crédito Hipotecario

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | wizard, progreso persistente, onboarding largo, formularios multi-paso |
| **Patrón Técnico** | Stepper State Machine, [Draft](#term-draft "Borrador persistente del progreso parcial del usuario.") Persistence, Checkpointing |
| **Stack Seleccionado** | Flutter + Riverpod/StateNotifier + Hive/SQLite para drafts + Finite State Machine ([FSM](#term-fsm "Máquina de estados finitos que define transiciones válidas entre pasos.")) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Estado solo en memoria o guardado manual provoca pérdida de progreso al cerrar la app o cambiar de dispositivo.
- Sin FSM ni checkpoints, el usuario puede saltar pasos ilegales, generando datos incompletos o inválidos para KYC/hipoteca.
- Sin autosave/background sync, se reprocesan pasos completos y sube el abandono.

### Escenario de Negocio

> *"Como solicitante, quiero completar mi hipoteca en varios intentos sin perder mi progreso."*

### Incidentes reportados
- **Estudios hipotecarios:** Formularios >10 pasos tienen abandono > 40% sin guardado automático.
- **Banca 2021:** 18% de solicitudes incompletas por pérdida de progreso al cambiar de dispositivo.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Estudios hipotecarios | Apps de crédito | Abandono > 40% sin autosave/checkpoints en flujos largos. |
| Banca 2021 | Solicitudes multi-dispositivo | 18% incompletas por pérdida de progreso. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; persistencia/estado es fuente de bugs UX. |

**Resumen global**
- Wizards largos sin autosave/checkpoints elevan abandono y soporte.
- FSM con checkpoints mitiga saltos ilegales y pérdida de datos.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Abandono de solicitudes valiosas, pérdida de comisiones |
| **UX** | Frustración por reingresar datos, caída de NPS |
| **Técnico** | Datos parcialmente válidos sin contexto de estado |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Estado solo en memoria, sin guardado | **INADECUADO:** Cerrar app = perder todo. |
| **ACEPTABLE** | Guardado manual al final de cada paso | **MEJORA:** Reduce pérdida, pero depende del usuario; no cubre cierres inesperados. |
| **ENTERPRISE** | **FSM + autosave + checkpoints:** guardar draft tras cada paso, sincronizar en background, reanudar con validación parcial | **ÓPTIMO:** Cero pérdida de datos, reanudación en cualquier dispositivo, validación incremental. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Guardar draft local y sincronizar en segundo plano. Validar por paso y cross-step en checkpoints. Reanudar en paso exacto. FSM evita saltos ilegales de estado. Manejar expiración de sesión y solicitar reautenticación sin perder datos. |
| **Restricciones Duras (NO permite)** | **Conflictos multi-dispositivo:** Requiere merge/ultima edición o control de lock en backend. **Consistencia de reglas:** Cambios en reglas KYC pueden invalidar drafts antiguos. **Privacidad:** Debe cifrar drafts sensibles en reposo. |
| **Criterio de Selección** | FSM para restricciones de flujo; Riverpod/StateNotifier por claridad y testeo; Hive/SQLite para almacenamiento ligero y cifrado opcional. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | FSM impide saltos ilegales; autosave persiste borrador | Equipo móvil, CI |
| Integration (CI) | Reanudar en paso exacto tras cerrar/app kill; sync en background funciona | Móvil/Backend, CI + staging |
| Seguridad/consistencia | Draft cifrado; no se filtran datos en logs | Seguridad/QE |
| Observabilidad | Eventos `wizard.*` con paso, checkpoint, estado de sync | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| [Autosave](#term-autosave "Guardado automático tras cambios sin acción explícita del usuario.") | Guardar al salir de cada paso y en background | Minimiza pérdida |
| Reanudación | Mostrar “continuar donde quedaste” con paso actual | Reduce abandono |
| Validación | Validar por paso y cross-step en checkpoints | Calidad de datos |
| Multi-dispositivo | Merge último cambio o bloquear edición simultánea | Consistencia |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| TTL de drafts | Expirar drafts tras X días; notificar al usuario | Control regulatorio |
| Conflictos | Resolver por timestamp o solicitar elección al usuario | Minimiza pérdida |
| Cambios de reglas | Invalidar/revalidar drafts al cambiar reglas KYC | Cumplimiento |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Pérdida de progreso y saltos ilegales en wizards largos. |
| Opciones evaluadas | Estado en memoria; guardado manual; FSM + autosave + checkpoints. |
| Decisión | FSM con checkpoints, autosave y sync en background. |
| Consecuencias | Mayor complejidad de estado y almacenamiento; manejo de expiraciones. |
| Riesgos aceptados | Conflictos multi-dispositivo; drafts expiran si no se reanudan. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Abandono de wizard | ↓ vs baseline | Alerta si no mejora | Conversión mejorada |
| Pérdida de progreso | 0 incidentes por cierre/app kill | Crítico si > 0 | Menos soporte |
| Reanudación exitosa | > 99% reanuda en paso correcto | Alerta si baja | Confianza del usuario |
| Draft cifrado | 0 filtraciones en logs/storage | Crítico si > 0 | Cumplimiento y privacidad |
| Tickets por reingreso de datos | ↓ ≥ 30% | Alerta si no baja | Soporte controlado |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-wizard"></a>Wizard | Flujo multi-paso guiado para completar un proceso largo. |
| <a id="term-fsm"></a>FSM | Máquina de estados finitos que define transiciones válidas entre pasos. |
| <a id="term-draft"></a>Draft | Borrador persistente del progreso parcial del usuario. |
| <a id="term-checkpoint"></a>Checkpoint | Punto de guardado con validaciones parciales antes de avanzar. |
| <a id="term-autosave"></a>Autosave | Guardado automático tras cambios sin acción explícita del usuario. |
| <a id="term-ttl-de-draft"></a>TTL de draft | Tiempo máximo que se conserva un borrador antes de expirar. |

---

## Referencias

- [State Machines and Wizards (Fowler)](https://martinfowler.com/articles/uiState.html)
- [NNG - Long Forms UX](https://www.nngroup.com/articles/long-forms/)
- [Flutter State Restoration](https://docs.flutter.dev/development/ui/interactive#state-restoration)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
