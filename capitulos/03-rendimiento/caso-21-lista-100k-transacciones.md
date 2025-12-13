# Caso 21: La Lista de 100,000 Transacciones
## Scroll Infinito sin Memory Leaks

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | lista infinita, paginación, rendimiento, transacciones, memory leak |
| **Patrón Técnico** | Infinite Scrolling, Pagination with Cache, List Virtualization |
| **Stack Seleccionado** | Flutter + `ScrollablePositionedList`/`ListView.builder` + `infinite_scroll_pagination` + Isolates para parsing |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Listas grandes con celdas ricas sin virtualización ni paginación → jank, OOM, cierres.
- Parsing en el hilo UI y requests duplicadas al hacer scroll rápido saturan CPU/red.
- Sin placeholders y cache, la UX se ve interrumpida (pop-in, flashes de layout).

### Escenario de Negocio

> *"Como usuario, quiero navegar 100,000 transacciones sin que la app se congele ni se coma la memoria."*

### Incidentes reportados
- **Apps bancarias:** Quejas de jank al cargar historial largo; mitigado con paginación y placeholders.
- **Flutter perf guides:** Recomiendan builders perezosos y evitar `shrinkWrap`/layouts costosos.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Reportes de soporte bancario (2023) | Historiales largos | Jank/ANR al listar +50K items sin paginación. |
| Flutter perf (guías) | Global | Builders perezosos y virtualización reducen uso de memoria y jank. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; renderizado/listas son foco común de bugs de perf. |

**Resumen global**
- Listas sin virtualización/paginación causan jank y cierres en historiales grandes.
- Parsing en UI y requests duplicadas aumentan consumo y soporte.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Scroll entrecortado, congelamientos, cierres de la app |
| **Técnico** | Fugas de memoria por celdas retenidas, parsing en el hilo UI |
| **Reputacional/Económico** | Usuarios abandonan o no encuentran transacciones críticas |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Cargar toda la lista en memoria, `ListView` sin builder | **INADECUADO:** OOM, jank severo. |
| **ACEPTABLE** | `ListView.builder` + paginación manual | **MEJORA:** Menor memoria, pero requiere manejo manual de loading/error y puede duplicar requests. |
| **ENTERPRISE** | **Paginación estructurada:** `infinite_scroll_pagination` o controller propio con estados (loading/error/empty), cache local, placeholders, parsing en [Isolate](#term-isolate "Hilo ligero de Dart para trabajo pesado sin bloquear UI.") | **ÓPTIMO:** Smooth scroll, uso de memoria controlado, UX consistente ante errores. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Paginación con clave (cursor/offset) y cache parcial en SQLite. Placeholders skeleton mientras carga. Parsing de JSON pesado en Isolate. Reintentos con backoff por página. [Prefetch](#term-prefetch "Solicitar anticipadamente la siguiente página cerca del final del scroll.") de la siguiente página al 80% del scroll. |
| **Restricciones Duras (NO permite)** | **Sin API paginada:** Si backend no expone cursor/offset, se degrada a fetch completo. **Layouts complejos con shrinkWrap:** Aumentan costo de layout. **Orden garantizado:** Cambios en backend pueden invalidar posición; requiere stable IDs. |
| **Criterio de Selección** | [Virtualización](#term-virtualizacion "Renderizar solo los ítems visibles + buffer cercano.") de celdas con builders; paginador estructurado para estados claros; cache local para offline breve; Isolate para parsing de lotes grandes. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Paginador maneja estados loading/error/empty y no duplica requests | Equipo móvil, CI |
| Integration (CI) | Prefetch al 80% y backoff en fallos; cache local reusa páginas | Móvil/Backend, CI |
| Performance | FPS/jank aceptable con 100K items; memoria estable | QA/Perf, dispositivos reales |
| Observabilidad | Eventos `list.page` con cursor, latencia y errores | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Placeholders | Skeleton/blur mientras carga página | Evita pop-in |
| Prefetch | Siguiente página al 70-80% del scroll | Scroll suave |
| Reintentos | Backoff y botón “reintentar” por página | Control de fallos |
| Offline breve | Mostrar cache y estado “posible desactualizado” | Transparencia |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Stable IDs | Requeridos para mantener posición y evitar parpadeos | Consistencia visual |
| Cache TTL | Definir TTL de páginas y limpieza | Control de memoria |
| Isolates | Parsing de lotes pesados fuera del UI thread | Previene jank |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Jank/OOM al listar 100K items sin virtualización. |
| Opciones evaluadas | Lista completa en memoria; builder con paginación manual; paginador estructurado + cache + isolates. |
| Decisión | Paginación estructurada + cache local + parsing en Isolate + placeholders/prefetch. |
| Consecuencias | Más complejidad en estados de lista y cache; requiere API paginada. |
| Riesgos aceptados | Sin API paginada el beneficio es menor; depende de IDs estables. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Jank/FPS en lista | Scroll estable; sin dropped frames visibles | Alerta si jank sube | UX fluida |
| Memoria usada | Estable; sin OOM al llegar a 100K | Crítico si crece sin límite | Estabilidad |
| Latencia de página | p95 < 700 ms | Warning si se acerca | Percepción de rapidez |
| Reintentos fallidos | < 1% de páginas | Alerta si supera | Resiliencia |
| Tickets por lista lenta | ↓ vs baseline | Alerta si no baja | Menos soporte |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-virtualizacion"></a>Virtualización | Renderizar solo los ítems visibles + buffer cercano. |
| <a id="term-cursor-offset"></a>Cursor/Offset | Mecanismo de paginación; cursor es más estable ante inserciones. |
| <a id="term-placeholder-skeleton"></a>Placeholder/Skeleton | Celda de carga que mantiene estabilidad del layout. |
| <a id="term-prefetch"></a>Prefetch | Solicitar anticipadamente la siguiente página cerca del final del scroll. |
| <a id="term-isolate"></a>Isolate | Hilo ligero de Dart para trabajo pesado sin bloquear UI. |
| <a id="term-stable-id"></a>Stable ID | Identificador consistente para cada ítem, usado para claves de lista. |

---

## Referencias

- [Flutter Performance Best Practices](https://docs.flutter.dev/perf/best-practices)
- [infinite_scroll_pagination](https://pub.dev/packages/infinite_scroll_pagination)
- [Google Perf - Large Lists](https://developer.android.com/topic/performance/rendering/optimize-view-hierarchies)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
