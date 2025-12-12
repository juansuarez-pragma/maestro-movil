# Caso 35: Background Sync Prohibido
## Estrategias cuando iOS Mata tu Proceso

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | background sync, iOS restrictions, tasks, offline |
| **Patrón Técnico** | Opportunistic Sync, Background Tasks, Push-to-Sync |
| **Stack Seleccionado** | Flutter + BGTaskScheduler (iOS) via Platform Channels + Riverpod políticas + push triggers |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero que mis datos se sincronicen aunque iOS cierre la app, sin agotar batería."*

iOS limita severamente el trabajo en background; los syncs confiables requieren triggers oportunistas y push.

### Evidencia de Industria

- **Apps de productividad/banca:** Usan BGTaskScheduler + push para sync crítico.
- **Apple docs:** Background App Refresh no es garantizado; tareas pueden ser diferidas.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Datos desactualizados al abrir la app |
| **Técnico** | Tareas canceladas por el sistema; consumo de batería si mal configurado |
| **Reputacional** | Usuarios perciben app "que no sincroniza" |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Polling en background | **INADECUADO:** iOS lo bloquea, consume batería. |
| **ACEPTABLE** | BGAppRefresh con intervalo amplio | **MEJORA:** Mejor que polling, pero no garantizado. |
| **ENTERPRISE** | **Push-to-sync + BGTaskScheduler:** notificación silenciosa dispara sync corto, tareas registradas con límites, batching y cancelación | **ÓPTIMO:** Respeta políticas, más confiable y eficiente. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Registrar tareas BG con identificadores claros. Usar push silencioso para disparar sync breve. Batching de operaciones. Cancelar/posponer si batería baja o sin conectividad. |
| **Restricciones Duras (NO permite)** | **Garantía absoluta:** iOS puede omitir tareas. **Ventanas cortas:** Trabajo debe ser rápido (<30s). **Permisos:** Necesita entitlement y justificación. |
| **Criterio de Selección** | Push-to-sync para eventos críticos; BGTaskScheduler para sync periódico ligero; minimizar trabajo y medir consumo. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| BGTaskScheduler | API iOS para tareas en background programadas. |
| Push silencioso | Notificación sin UI para disparar lógica. |
| Opportunistic Sync | Sincronizar solo cuando hay condiciones favorables. |
| Batching | Agrupar operaciones para reducir wakeups. |
| Entitlement | Permiso declarado para usar capacidades especiales. |

---

## Referencias

- [Apple BGTaskScheduler](https://developer.apple.com/documentation/backgroundtasks)
- [Apple Push Notifications](https://developer.apple.com/documentation/usernotifications)
