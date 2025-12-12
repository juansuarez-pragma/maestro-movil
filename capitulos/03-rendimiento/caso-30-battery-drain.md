# Caso 30: Battery Drain Silencioso
## Optimizar Background Tasks sin Matar la Batería

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | battery drain, background tasks, eficiencia, consumo de energía |
| **Patrón Técnico** | Work Scheduling, Doze/Aggressive App Standby, Rate Limiting |
| **Stack Seleccionado** | Flutter + platform channels (WorkManager/BackgroundFetch) + Riverpod para políticas + batching |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero notificaciones y sync sin que la batería se agote."*

Sin control de tareas en background, la app consume batería excesiva; el SO puede restringir o matar procesos, afectando entregas de eventos críticos (alertas, sync).

### Evidencia de Industria

- **Estudios móviles:** Apps con consumo elevado son desinstaladas rápidamente.
- **Android/iOS power policies:** Doze/App Standby y Background App Refresh limitan tareas.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Batería drenada, desinstalaciones |
| **Técnico** | Tareas críticas no se ejecutan si el SO bloquea la app |
| **Reputacional** | Malas reseñas por consumo |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Tareas en loops o timers constantes | **INADECUADO:** Despiertan CPU innecesariamente, consumo alto. |
| **ACEPTABLE** | WorkManager/BackgroundFetch con intervalos fijos | **MEJORA:** Respeta políticas, pero puede seguir siendo costoso sin batching. |
| **ENTERPRISE** | **Scheduling eficiente:** agrupar tareas (batching), limitar frecuencia por contexto (WiFi/carga), usar push-to-sync, cancelar trabajos al cerrar sesión | **ÓPTIMO:** Minimiza wakeups, respeta políticas de energía, entrega confiable. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Configurar WorkManager con constraints (WiFi, charging). Batching de syncs en una sola tarea. Usar push para disparar sync en lugar de polling. Cancelar trabajos al logout. Monitorear consumo con herramientas nativas. |
| **Restricciones Duras (NO permite)** | **Garantía de ejecución exacta:** SO puede diferir tareas. **iOS background limitado:** Ventanas cortas; requiere BGAppRefresh/Tasks y justificación. **Alarmas exactas:** Restringidas en Android 12+. |
| **Criterio de Selección** | Preferir push + batch; intervalos mínimos compatibles con políticas; constraints razonables; observar métricas de consumo real. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| WorkManager | Scheduler Android que respeta políticas de energía y condiciones. |
| Background Fetch | Mecanismo iOS para tareas periódicas limitadas. |
| Batching | Agrupar múltiples trabajos en una sola ejecución para reducir wakeups. |
| Push-to-sync | Usar notificación push como trigger en lugar de polling. |
| Doze/App Standby | Modos de ahorro de energía que limitan tareas en segundo plano. |

---

## Referencias

- [Android WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager)
- [iOS Background Tasks](https://developer.apple.com/documentation/backgroundtasks)
- [Android Power Management](https://developer.android.com/training/monitoring-device-state/doze-standby)
