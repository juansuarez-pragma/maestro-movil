# Caso 18: El Dashboard que Colapsó
## Orquestar 15 Streams de Datos Simultáneos

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | dashboard, streams múltiples, backpressure, prioridad de datos |
| **Patrón Técnico** | [Stream Multiplexing](#term-stream-multiplexing "Gestionar múltiples flujos y combinarlos con prioridad."), [Backpressure](#term-backpressure "Estrategia para manejar flujos más rápidos que el consumidor (drop, buffer, latest).") Handling, Priority Scheduling |
| **Stack Seleccionado** | Flutter + Riverpod Streams + Isolates para parsing + WebSockets/HTTP2 |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- 15 streams simultáneos sin prioridad/backpressure saturan UI/red → jank, lag y datos atrasados.
- Sin dedup/throttle, se consumen recursos y se muestran datos obsoletos (ticks perdidos).
- Parsing en UI thread bloquea rendering; sin isolates, el dashboard se congela.

### Escenario de Negocio

> *"Como usuario, quiero un dashboard con tasas, saldos y alertas en tiempo real sin que la app se congele."*

### Incidentes reportados
- **Trading apps:** Dashboards con múltiples feeds requieren throttling y prioridad; sin ello, se pierden ticks.
- **Flutter perf:** Recomienda aislar parsing pesado en Isolates para evitar jank.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Apps de trading/finanzas | Global | Lag y pérdida de ticks sin priorización/backpressure. |
| Estudios Flutter perf | Global | Parsing en UI genera jank; isolates recomendados. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; orquestación de streams y perf son áreas críticas. |

**Resumen global**
- Sin priorización/backpressure, dashboards pierden datos y congelan la UI.
- Isolates y selectors son necesarios para perf estable con múltiples feeds.

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
| **Capacidades (SÍ permite)** | Clasificar feeds (p. ej., alertas y saldos como alta prioridad; banners como baja). Aplicar `debounce/throttle` por stream. Usar `latest` para descartar datos viejos en secundarios. Parsing JSON pesado en [Isolate](#term-isolate "Hilo ligero de Dart para procesamiento paralelo sin bloquear UI."). Selectors para evitar rebuilds masivos. |
| **Restricciones Duras (NO permite)** | **Consistencia fuerte entre feeds:** Llegada asíncrona puede mostrar datos ligeramente desalineados. **Sin server-side throttling:** Cliente solo mitiga; el backend debe soportar [QoS](#term-qos "Calidad de servicio; priorización y límites de entrega."). **Battery/CPU:** Streams constantes consumen recursos; requiere políticas de pausa en background. |
| **Criterio de Selección** | Riverpod Streams para control fino y cancelación; Isolates para parsing; priorización inspirada en sistemas de trading para balancear frescura vs costo. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Prioridad/selector solo notifica vistas afectadas | Equipo móvil, CI |
| Integration (CI) | Backpressure/drop/latest evita colas infinitas; throttling reduce carga | Móvil/Backend, CI |
| Performance | FPS/jank bajo carga con 15 streams; parsing en isolates | QA/Perf, dispositivos reales |
| Observabilidad | Métricas `dashboard.*` (latencia feed, drop rate) | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Prioridades | Alta: saldos/alertas; Media: tasas; Baja: banners | UX enfoca datos críticos |
| Throttle/debounce | Ajustar por feed (p.ej., banners 2s, alertas inmediato) | Balance frescura/carga |
| Modo background | Pausar feeds no críticos; mantener alertas | Ahorra batería/red |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Dedup | Un solo stream compartido por feed; evitar suscripciones duplicadas | Ahorra red |
| QoS backend | Coordinar límites y prioridades con backend | Evita sobrecarga |
| Latencia/caídas | Alertar si drop rate supera umbral; fallback a polling si falla socket | Resiliencia |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | 15 streams sin orquestación causan jank y pérdida de datos. |
| Opciones evaluadas | Suscripción ciega; throttling simple; priorización + backpressure + isolates. |
| Decisión | Priorizar feeds + backpressure (drop/latest) + parsing en isolates + selectors. |
| Consecuencias | Mayor complejidad de orquestación; tuning fino por feed. |
| Riesgos aceptados | Consistencia eventual entre feeds; dependencia de QoS backend. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| FPS/jank en dashboard | Estable (sin dropped frames significativos) | Alerta si jank sube | UX fluida |
| Latencia feeds críticos | p95 < 500 ms | Warning si sube | Datos frescos |
| Drop rate feeds secundarios | Controlado (p.ej., < 5%) | Alerta si sube | Recursos optimizados |
| Suscripciones duplicadas | 0 | Crítico si > 0 | Ahorro de red |
| Tickets por datos atrasados | ↓ vs baseline | Alerta si no baja | Menos soporte |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-backpressure"></a>Backpressure | Estrategia para manejar flujos más rápidos que el consumidor (drop, buffer, latest). |
| <a id="term-throttling-debounce"></a>Throttling/Debounce | Limitar frecuencia de eventos para reducir carga y jank. |
| <a id="term-stream-multiplexing"></a>Stream Multiplexing | Gestionar múltiples flujos y combinarlos con prioridad. |
| <a id="term-selector"></a>Selector | Derivar porciones de estado/stream para minimizar rebuild. |
| <a id="term-isolate"></a>Isolate | Hilo ligero de Dart para procesamiento paralelo sin bloquear UI. |
| <a id="term-qos"></a>QoS | Calidad de servicio; priorización y límites de entrega. |

---

## Referencias

- [Flutter Performance - Streams](https://docs.flutter.dev/perf/best-practices#asynchronous-programming)
- [Reactive Streams Backpressure](https://www.reactive-streams.org/)
- [Isolates in Flutter](https://docs.flutter.dev/perf/isolates)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
