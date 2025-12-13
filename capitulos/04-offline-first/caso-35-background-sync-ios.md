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

### Problema detectado (técnico)
- iOS limita fuertemente el trabajo en background; polling constante es bloqueado y consume batería.
- Background App Refresh no es garantizado; tareas pueden ser diferidas o canceladas si se excede la ventana.
- Sin push-to-sync y batching, los syncs críticos fallan o llegan tarde.

### Escenario de Negocio

> *"Como usuario, quiero que mis datos se sincronicen aunque iOS cierre la app, sin agotar batería."*

### Incidentes reportados
- **Apps de productividad/banca:** Usan BGTaskScheduler + push para sync crítico.
- **Apple docs:** Background App Refresh no es garantizado; tareas pueden diferirse.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Apple docs/prácticas | Global | BGTask/Refresh no garantizan ejecución exacta; ventanas cortas. |
| Apps productivas/banca | iOS | Push-to-sync + BGTask para mejorar fiabilidad. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; background/sync es punto crítico. |

**Resumen global**
- Sin push/batching y tareas cortas, los syncs en iOS son poco confiables y costosos en batería.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Integration (CI) | Registro de BGTask y ejecución dentro de ventana | Móvil/QA, staging iOS |
| Consumo | Medir impacto energético en dispositivos reales | QA/Perf |
| Observabilidad | Eventos `bg.sync` con duración/resultado; rate de ejecución | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Push-to-sync | Usar notificación silenciosa para sync crítico | Minimiza polling |
| Batching | Agrupar operaciones en la ventana BG | Eficiencia |
| Fallback | Reintentar en foreground si BG falló | Confiabilidad percibida |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Ventana BG | Mantener trabajo < 30s y cancelable | Cumple políticas |
| Entitlements | Declarar y justificar capacidades BG | Cumplimiento |
| Restricciones SO | Manejar diferimientos/cancelaciones | Resiliencia |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | iOS limita BG; polling bloqueado, sync falla o consume batería. |
| Opciones evaluadas | Polling; BGAppRefresh solo; push-to-sync + BGTask + batching. |
| Decisión | Push silencioso + BGTaskScheduler con trabajo breve y batch; fallback en foreground. |
| Consecuencias | Configuración de entitlements y monitoreo energético. |
| Riesgos aceptados | Ejecución no garantizada; variabilidad por políticas iOS. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Éxito de sync BG | Alto, dentro de ventana | Alerta si baja | Datos frescos |
| Duración de BGTask | < 30s | Alerta si supera | Cumple políticas |
| Consumo batería | Controlado | Alerta si sube | Mejora NPS |
| Tickets “no sincroniza” | ↓ vs baseline | Alerta si no baja | Confianza |

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
