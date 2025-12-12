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

### Escenario de Negocio

> *"Como usuario, la app no debe congelarse cuando se procesan lotes grandes de datos."*

Parsing JSON grande, compresión o cifrado en el hilo UI causan congelamientos perceptibles y ANR si superan el presupuesto de frame.

### Evidencia de Industria

- **Fintech con feeds grandes:** Parsing en UI produjo jank; se mitigó moviendo a Isolates.
- **Flutter perf:** Recomienda Isolates para CPU-bound que duren > 4–8ms.

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

---

## Referencias

- [Isolates in Flutter](https://docs.flutter.dev/perf/isolates)
- [Performance Best Practices](https://docs.flutter.dev/perf/best-practices)
- [Dart Isolates Overview](https://dart.dev/guides/libraries/concurrency)
