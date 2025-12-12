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

### Escenario de Negocio

> *"Como usuario, quiero navegar 100,000 transacciones sin que la app se congele ni se coma la memoria."*

Listas grandes con celdas ricas (avatars, montos formateados) generan jank y OOM si no se virtualizan y cachean correctamente.

### Evidencia de Industria

- **Apps bancarias:** Quejas de jank al cargar historial largo; mitigado con paginación y placeholders.
- **Flutter perf guides:** Recomiendan builders perezosos y evitar `shrinkWrap`/layouts costosos.

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
| **ENTERPRISE** | **Paginación estructurada:** `infinite_scroll_pagination` o controller propio con estados (loading/error/empty), cache local, placeholders, parsing en Isolate | **ÓPTIMO:** Smooth scroll, uso de memoria controlado, UX consistente ante errores. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Paginación con clave (cursor/offset) y cache parcial en SQLite. Placeholders skeleton mientras carga. Parsing de JSON pesado en Isolate. Reintentos con backoff por página. Prefetch de la siguiente página al 80% del scroll. |
| **Restricciones Duras (NO permite)** | **Sin API paginada:** Si backend no expone cursor/offset, se degrada a fetch completo. **Layouts complejos con shrinkWrap:** Aumentan costo de layout. **Orden garantizado:** Cambios en backend pueden invalidar posición; requiere stable IDs. |
| **Criterio de Selección** | Virtualización de celdas con builders; paginador estructurado para estados claros; cache local para offline breve; Isolate para parsing de lotes grandes. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Virtualización | Renderizar solo los ítems visibles + buffer cercano. |
| Cursor/Offset | Mecanismo de paginación; cursor es más estable ante inserciones. |
| Placeholder/Skeleton | Celda de carga que mantiene estabilidad del layout. |
| Prefetch | Solicitar anticipadamente la siguiente página cerca del final del scroll. |
| Isolate | Hilo ligero de Dart para trabajo pesado sin bloquear UI. |

---

## Referencias

- [Flutter Performance Best Practices](https://docs.flutter.dev/perf/best-practices)
- [infinite_scroll_pagination](https://pub.dev/packages/infinite_scroll_pagination)
- [Google Perf - Large Lists](https://developer.android.com/topic/performance/rendering/optimize-view-hierarchies)
