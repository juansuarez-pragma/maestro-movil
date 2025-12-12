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

### Escenario de Negocio

> *"Como usuario frecuente, la app debe correr horas sin crecer indefinidamente en memoria."*

Sesiones largas con streams, controllers y caches mal gestionados causan OOM y cierres inesperados.

### Evidencia de Industria

- **Apps financieras:** Sesiones de trading largas expusieron leaks en streams no cancelados.
- **Flutter perf:** Recomienda `autoDispose` y cierre explícito de controllers/listeners.

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

---

## Referencias

- [Flutter DevTools Memory](https://docs.flutter.dev/tools/devtools/memory)
- [Riverpod autoDispose](https://riverpod.dev/docs/concepts/providers/#auto-dispose)
- [Android/iOS Memory Profiling](https://developer.android.com/studio/profile/memory)
