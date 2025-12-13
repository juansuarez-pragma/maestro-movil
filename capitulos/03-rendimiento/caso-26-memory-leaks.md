# Caso 26: Memoria que No Regresa
## Hunting Memory Leaks en Sesiones Largas

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | memory leak, sesiones largas, perfilado, gc |
| **Patrón Técnico** | Leak Detection, Resource Disposal, Profiling |
| **Stack Seleccionado** | Flutter DevTools (Memory/CPU), Riverpod autoDispose, WeakReference patterns |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Streams, controllers y caches sin liberar en sesiones largas provocan crecimiento de memoria y OOM.
- Singletons/listeners estáticos retienen contexto; sin autoDispose ni límites, los leaks son silenciosos.
- Falta de profiling/snapshots hace difícil detectarlos hasta que la app ya degrada.

### Escenario de Negocio

> *"Como usuario frecuente, la app debe correr horas sin crecer indefinidamente en memoria."*

### Incidentes reportados
- **Apps financieras:** Sesiones de trading largas expusieron leaks en streams no cancelados.
- **Flutter perf:** Recomienda `autoDispose` y cierre explícito de controllers/listeners.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Apps de trading/banca | Sesiones largas | Leaks por streams/listeners sin cerrar. |
| Flutter perf guías | Global | `autoDispose` y límites de cache como práctica recomendada. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; manejo de recursos es hallazgo común. |

**Resumen global**
- Leaks silenciosos en sesiones largas causan OOM y cierres.
- autoDispose, límites de cache y profiling periódico son claves para estabilidad.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Cierres inesperados, degradación progresiva |
| **Técnico** | OOM, watchdogs del SO terminan la app |
| **Reputacional** | Percepción de app inestable/insegura |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | No cerrar controllers/streams | **INADECUADO:** Acumulación de listeners y buffers. |
| **ACEPTABLE** | Cerrar recursos manualmente en dispose | **MEJORA:** Mitiga leaks, depende de disciplina; falla en rutas condicionales. |
| **ENTERPRISE** | **Gestión explícita + autoDispose + monitoreo:** scopes claros, autoDispose en providers, profiling regular, límites de cache y limpieza | **ÓPTIMO:** Reduce leaks sistemáticamente y los detecta temprano. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Usar `autoDispose` para providers/streams de vida corta. Cerrar controllers/listeners en `dispose`. Limitar tamaño y TTL de caches. Monitorear heap en DevTools y tomar snapshots comparativos. |
| **Restricciones Duras (NO permite)** | **Leaks en nativo:** Dependen del lado platform. **Fugas por referencias estáticas/singletons:** Requieren revisión de diseño. **Caches agresivas:** Pueden ocultar leaks; necesidad de métricas. |
| **Criterio de Selección** | autoDispose para lifecycle predecible; DevTools para evidencia; patrones sin singletons globales salvo necesidad; límites en caches. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Providers autoDispose liberan recursos sin listeners activos | Equipo móvil, CI |
| Integration (CI) | Flujo largo no crece memoria; caches respetan TTL/tamaño | Móvil/QA, CI |
| Seguridad/consistencia | No hay referencias estáticas a contexto/Build | QA/Seguridad |
| Observabilidad | Snapshots de heap y métricas de uso; alertas si heap crece | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Cierre de recursos | Cerrar/autoDispose streams/controllers al salir de pantalla | Evita leaks |
| Caches | Definir límites/TTL; limpiar en background | Control de memoria |
| Monitoreo | Ejecutar sesiones largas en QA perf y capturar heap | Detecta temprano |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Singletons | Evitar retener contexto; usar DI correcta | Reduce fugas |
| Herramientas | DevTools, memory profiler en Android/iOS | Evidencia |
| Alarmas | Alertar si heap crece N MB/h sin liberar | Prevención |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Leaks de memoria en sesiones largas por recursos no liberados. |
| Opciones evaluadas | Cierre manual ad-hoc; autoDispose + límites de cache + monitoreo. |
| Decisión | autoDispose + cierre explícito + límites de cache + profiling periódico. |
| Consecuencias | Disciplina de lifecycle y monitoreo; mayor esfuerzo en QA perf. |
| Riesgos aceptados | Leaks en nativo requieren trabajo extra; caches agresivas pueden ocultar problemas. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Crecimiento de heap en sesión larga | Estable; sin tendencia ascendente | Alerta si crece sostenido | Estabilidad |
| OOM/crashes | 0 | Crítico si > 0 | Confiabilidad |
| Recursos liberados | 100% streams/controllers cerrados | Alerta si hay filtraciones | Previene leaks |
| Uso de cache | Dentro de límite configurado | Alerta si supera | Memoria controlada |
| Tickets por cierres inesperados | ↓ vs baseline | Alerta si no baja | Menos soporte |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| OOM | Out Of Memory; cierre por falta de memoria. |
| autoDispose | Opción de Riverpod para liberar providers al no tener oyentes. |
| Heap snapshot | Captura de memoria para comparar leaks. |
| Listener | Suscriptor a stream/cambio que debe liberarse. |
| Cache TTL | Tiempo máximo de retención de elementos en cache. |
| Leak | Retención de memoria no liberada tras uso. |

---

## Referencias

- [Flutter DevTools Memory](https://docs.flutter.dev/tools/devtools/memory)
- [Riverpod autoDispose](https://riverpod.dev/docs/concepts/providers/#auto-dispose)
- [Android/iOS Memory Profiling](https://developer.android.com/studio/profile/memory)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
