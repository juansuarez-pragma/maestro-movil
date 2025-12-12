# Caso 17: Wizard de 12 Pasos
## Persistir Progreso de Solicitud de Crédito Hipotecario

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | wizard, progreso persistente, onboarding largo, formularios multi-paso |
| **Patrón Técnico** | Stepper State Machine, Draft Persistence, Checkpointing |
| **Stack Seleccionado** | Flutter + Riverpod/StateNotifier + Hive/SQLite para drafts + Finite State Machine (FSM) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como solicitante, quiero completar mi hipoteca en varios intentos sin perder mi progreso."*

Wizards largos fallan si el usuario pierde conexión, cierra la app o recibe un error; reingresar datos causa abandono y llamadas a soporte.

### Evidencia de Industria

- **Estudios hipotecarios:** Formularios de más de 10 pasos tienen abandono > 40% sin guardado automático.
- **Caso Banca 2021:** 18% de solicitudes incompletas por pérdida de progreso al cambiar de dispositivo.

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

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Wizard | Flujo multi-paso guiado para completar un proceso largo. |
| FSM | Máquina de estados finitos que define transiciones válidas entre pasos. |
| Draft | Borrador persistente del progreso parcial del usuario. |
| Checkpoint | Punto de guardado con validaciones parciales antes de avanzar. |
| Autosave | Guardado automático tras cambios sin acción explícita del usuario. |

---

## Referencias

- [State Machines and Wizards (Fowler)](https://martinfowler.com/articles/uiState.html)
- [NNG - Long Forms UX](https://www.nngroup.com/articles/long-forms/)
- [Flutter State Restoration](https://docs.flutter.dev/development/ui/interactive#state-restoration)
