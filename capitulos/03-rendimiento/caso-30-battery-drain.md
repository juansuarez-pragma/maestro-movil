# Caso 30: Battery Drain Silencioso
## Optimizar Background Tasks sin Matar la Batería

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | battery drain, background tasks, eficiencia, consumo de energía |
| **Patrón Técnico** | Work Scheduling, Doze/Aggressive App Standby, Rate Limiting |
| **Stack Seleccionado** | Flutter + platform channels ([WorkManager](#term-workmanager "Scheduler Android que respeta políticas de energía y condiciones.")/BackgroundFetch) + Riverpod para políticas + batching |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Tareas en background sin batching/constraints drenan batería; el SO restringe/kills procesos → se pierden alertas/sync.
- Polling constante sin push-to-sync genera wakeups innecesarios.
- Falta de monitoreo de consumo oculta regresiones energéticas.

### Escenario de Negocio

> *"Como usuario, quiero notificaciones y sync sin que la batería se agote."*

### Incidentes reportados
- **Estudios móviles:** Apps con consumo elevado son desinstaladas rápidamente.
- **Android/iOS power policies:** [Doze/App Standby](#term-doze-app-standby "Modos de ahorro de energía que limitan tareas en segundo plano.") y Background App Refresh limitan tareas.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Reportes de consumo | Global | Consumo elevado correlaciona con desinstalaciones. |
| Android/iOS power policies | Global | Restricciones de Doze/Background App Refresh afectan tareas mal programadas. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; consumo/battery es hallazgo frecuente. |

**Resumen global**
- Sin scheduling eficiente, el battery drain aumenta y el SO limita la app.
- [Push-to-sync](#term-push-to-sync "Usar notificación push como trigger en lugar de polling.") y batching son claves para minimizar wakeups y consumo.

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
| **Capacidades (SÍ permite)** | Configurar WorkManager con constraints (WiFi, charging). [Batching](#term-batching "Agrupar múltiples trabajos en una sola ejecución para reducir wakeups.") de syncs en una sola tarea. Usar push para disparar sync en lugar de polling. Cancelar trabajos al logout. Monitorear consumo con herramientas nativas. |
| **Restricciones Duras (NO permite)** | **Garantía de ejecución exacta:** SO puede diferir tareas. **iOS background limitado:** Ventanas cortas; requiere BGAppRefresh/Tasks y justificación. **Alarmas exactas:** Restringidas en Android 12+. |
| **Criterio de Selección** | Preferir push + batch; intervalos mínimos compatibles con políticas; constraints razonables; observar métricas de consumo real. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit/Integration (CI) | Tareas se encolan con constraints correctos y se cancelan al logout | Móvil/Backend, CI |
| Performance/Consumo | Medir consumo con herramientas nativas; validar impacto de intervalos | QA/Perf |
| Observabilidad | Métricas de wakeups, duración de tareas, tasa de éxito | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Push-to-sync | Usar push para disparar sync; evitar polling | Menos consumo |
| Batching | Agrupar trabajos para reducir wakeups | Eficiencia |
| Constraints | WiFi/carga cuando sea posible; fallback documentado | Respeto de políticas |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Intervalos | Ajustar según criticidad; evitar intervalos agresivos | Balance consumo/entrega |
| Cancelación | Cancelar trabajos al cerrar sesión | Seguridad y consumo |
| Monitoreo | Alertar si consumo/wakeups suben | Prevención de regresiones |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Battery drain por tareas en background sin control. |
| Opciones evaluadas | Polling constante; intervalos fijos; push-to-sync + batching + constraints. |
| Decisión | Scheduling eficiente: push-to-sync, batching, constraints, cancel en logout, monitoreo. |
| Consecuencias | Configuración específica por plataforma; depende de calidad de backend/push. |
| Riesgos aceptados | Tareas pueden ser diferidas por políticas; variabilidad por OEM. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Consumo de batería | Dentro de presupuesto definido | Alerta si sube | Menor desinstalación |
| Wakeups | Reducidos vía batching/push | Alerta si aumentan | Eficiencia |
| Entrega de tareas | Éxito alto con constraints | Alerta si falla | Confiabilidad |
| Tickets por batería | ↓ vs baseline | Alerta si no baja | Reputación |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-workmanager"></a>WorkManager | Scheduler Android que respeta políticas de energía y condiciones. |
| <a id="term-background-fetch"></a>Background Fetch | Mecanismo iOS para tareas periódicas limitadas. |
| <a id="term-batching"></a>Batching | Agrupar múltiples trabajos en una sola ejecución para reducir wakeups. |
| <a id="term-push-to-sync"></a>Push-to-sync | Usar notificación push como trigger en lugar de polling. |
| <a id="term-doze-app-standby"></a>Doze/App Standby | Modos de ahorro de energía que limitan tareas en segundo plano. |
| <a id="term-wakeup"></a>Wakeup | Evento que despierta la CPU; costoso energéticamente. |

---

## Referencias

- [Android WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager)
- [iOS Background Tasks](https://developer.apple.com/documentation/backgroundtasks)
- [Android Power Management](https://developer.android.com/training/monitoring-device-state/doze-standby)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
