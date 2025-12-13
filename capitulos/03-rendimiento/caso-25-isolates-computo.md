# Caso 25: El Widget que Congeló el Hilo Principal
## Cómputo Pesado con Isolates

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | computo pesado, hilo UI, isolates, parsing, jank |
| **Patrón Técnico** | Background Processing, Isolate Pattern, Work Scheduling |
| **Stack Seleccionado** | Flutter + Isolates/`compute` + Stream/IsolateChannel + Riverpod para orquestar |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Parsing/CPU-bound en el hilo UI (> 4–8ms) causa jank y ANR; freezes perceptibles.
- Copiar grandes datasets sin chunking ni canales incrementa tiempo y memoria.
- Sin cancelación/prioridad, tareas largas bloquean UX o consumen batería.

### Escenario de Negocio

> *"Como usuario, la app no debe congelarse cuando se procesan lotes grandes de datos."*

### Incidentes reportados
- **Fintech con feeds grandes:** Parsing en UI produjo jank; se mitigó moviendo a Isolates.
- **Flutter perf:** Recomienda Isolates para CPU-bound > 4–8ms.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Apps con feeds/ETL | Global | Congelamientos por parsing en UI; ANR en sesiones largas. |
| Flutter perf guías | Global | Isolates para CPU-bound; evitar bloquear main thread. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; perf/concurrency son hallazgos comunes. |

**Resumen global**
- CPU-bound en UI = jank/ANR; aislar en Isolates con chunking/cancelación es clave.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | UI congelada, toques sin respuesta |
| **Técnico** | ANRs, watchdogs del SO matan la app |
| **Reputacional** | Percepción de app inestable |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Ejecutar tareas pesadas en UI thread | **INADECUADO:** Congela UI, riesgo ANR. |
| **ACEPTABLE** | `compute` para tareas puntuales pequeñas | **MEJORA:** Aísla, pero limitado a funciones top-level/estáticas. |
| **ENTERPRISE** | **Isolates dedicados + scheduling:** canal de mensajes, chunking de trabajo, cancelación, prioridad | **ÓPTIMO:** Evita jank, permite procesar lotes grandes y cancelables. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Mover CPU-bound (parsing, cifrado, compresión) a Isolate. Procesar en chunks para progresos parciales. Cancelar trabajos largos. Priorizar tareas críticas. |
| **Restricciones Duras (NO permite)** | **Acceso UI:** Isolates no manipulan widgets/BuildContext. **Memoria:** Copia de datos grandes entre Isolates tiene costo. **Complejidad:** Más difícil depurar y sincronizar resultados. |
| **Criterio de Selección** | Isolate dedicado para flujos recurrentes; `compute` para tareas simples; chunking para datasets grandes; canales con estados para reportar progreso. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Performance | Frame time/jank no se degrada al procesar lotes | QA/Perf |
| Unit (CI) | Tareas en Isolate retornan/cancelan según lo esperado | Equipo móvil, CI |
| Integration (CI) | Chunking/canales devuelven progreso y respetan prioridad | Móvil/Backend, CI |
| Observabilidad | Métricas `isolate.task_time`, cancelaciones, tamaño de payload | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Progreso | Mostrar avance si tarea > X ms; opción de cancelar | Transparencia |
| Prioridad | Tareas críticas primero; secundarias en background | UX estable |
| Tamaño de payload | Evitar pasar blobs enormes; usar streaming/chunking | Menos copias |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Pooling | Reusar Isolates para trabajos recurrentes | Menos overhead |
| Cancelación | Soportar cancel en tareas largas | UX responsiva |
| Batería | Evitar CPU sostenida sin necesidad; pausar en background | Ahorro energético |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Cómputo pesado en main thread provoca jank/ANR. |
| Opciones evaluadas | Todo en UI; `compute` puntual; Isolates dedicados + chunking + cancelación. |
| Decisión | Isolates dedicados + chunking/cancel + prioridad; `compute` para tareas simples. |
| Consecuencias | Mayor complejidad de orquestación; manejo de mensajes/errores. |
| Riesgos aceptados | Overhead de copias; depuración más compleja. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Jank/frame time | Sin degradación perceptible durante tareas | Alerta si sube | UX fluida |
| ANR/crashes por CPU | 0 | Crítico si > 0 | Estabilidad |
| Tiempo de tarea en Isolate | Dentro de SLA definido | Warning si se acerca | Rendimiento |
| Cancelaciones | Soportadas y efectivas | Alerta si fallan | UX responsable |
| Consumo de batería | Controlado (sin CPU sostenida innecesaria) | Alerta si sube | Energía |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Isolate | Hilo ligero de Dart con memoria propia; se comunica por mensajes. |
| `compute` | Helper para ejecutar una función en Isolate temporal. |
| Chunking | Dividir trabajo grande en porciones pequeñas procesables. |
| ANR | App Not Responding; el SO mata apps que bloquean el hilo principal. |
| Canal de mensajes | Mecanismo para enviar/recibir datos entre Isolates. |
| Prioridad | Orden de ejecución de tareas según criticidad. |

---

## Referencias

- [Isolates in Flutter](https://docs.flutter.dev/perf/isolates)
- [Performance Best Practices](https://docs.flutter.dev/perf/best-practices)
- [Dart Isolates Overview](https://dart.dev/guides/libraries/concurrency)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
