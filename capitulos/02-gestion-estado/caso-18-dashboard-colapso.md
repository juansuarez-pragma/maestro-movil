# Caso 18: El Dashboard que Colapsó
## Orquestar 15 Streams de Datos Simultáneos

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | dashboard, streams múltiples, backpressure, prioridad de datos |
| **Patrón Técnico** | Stream Multiplexing, Backpressure Handling, Priority Scheduling |
| **Stack Seleccionado** | Flutter + Riverpod Streams + Isolates para parsing + WebSockets/HTTP2 |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero un dashboard con tasas, saldos y alertas en tiempo real sin que la app se congele."*

15 streams simultáneos saturan la UI y la red; sin priorización y backpressure, se producen janks y datos atrasados.

### Evidencia de Industria

- **Trading apps:** Dashboards con múltiples feeds requieren throttling y priorización; sin ello, se pierden ticks.
- **Flutter perf:** Recomienda aislar parsing pesado en Isolates para evitar jank.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Datos atrasados llevan a decisiones erróneas (trades, alertas). |
| **UX** | Freezes y lag al renderizar múltiples streams. |
| **Técnico** | Memory leaks y colas sin consumir si no hay backpressure. |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Suscribirse a todos los streams y rebuild global | **INADECUADO:** Rebuild masivo, jank, desperdicio de red. |
| **ACEPTABLE** | Agrupar streams y throttling básico | **MEJORA:** Menos ruido, pero sin prioridad ni backpressure formal. |
| **ENTERPRISE** | **Orquestación con prioridad:** clasificar streams (críticos vs secundarios), throttling selectivo, backpressure (drop/latest), parsing en Isolates, selectors para UI | **ÓPTIMO:** Rendimiento estable, datos críticos frescos, menor jank. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Clasificar feeds (p. ej., alertas y saldos como alta prioridad; banners como baja). Aplicar `debounce/throttle` por stream. Usar `latest` para descartar datos viejos en secundarios. Parsing JSON pesado en Isolate. Selectors para evitar rebuilds masivos. |
| **Restricciones Duras (NO permite)** | **Consistencia fuerte entre feeds:** Llegada asíncrona puede mostrar datos ligeramente desalineados. **Sin server-side throttling:** Cliente solo mitiga; el backend debe soportar QoS. **Battery/CPU:** Streams constantes consumen recursos; requiere políticas de pausa en background. |
| **Criterio de Selección** | Riverpod Streams para control fino y cancelación; Isolates para parsing; priorización inspirada en sistemas de trading para balancear frescura vs costo. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Backpressure | Estrategia para manejar flujos más rápidos que el consumidor (drop, buffer, latest). |
| Throttling/Debounce | Limitar frecuencia de eventos para reducir carga y jank. |
| Stream Multiplexing | Gestionar múltiples flujos y combinarlos con prioridad. |
| Selector | Derivar porciones de estado/stream para minimizar rebuild. |
| Isolate | Hilo ligero de Dart para procesamiento paralelo sin bloquear UI. |

---

## Referencias

- [Flutter Performance - Streams](https://docs.flutter.dev/perf/best-practices#asynchronous-programming)
- [Reactive Streams Backpressure](https://www.reactive-streams.org/)
- [Isolates in Flutter](https://docs.flutter.dev/perf/isolates)
