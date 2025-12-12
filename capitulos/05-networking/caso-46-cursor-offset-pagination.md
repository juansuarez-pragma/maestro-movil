# Caso 46: Cursor vs Offset Pagination
## Navegar Millones de Productos Eficientemente

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | paginación, cursor, offset, rendimiento, listas grandes |
| **Patrón Técnico** | Cursor-based Pagination, Offset Pagination, Stable Sorting |
| **Stack Seleccionado** | Flutter + Dio + `infinite_scroll_pagination` + cache local |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero explorar millones de productos sin saltos ni repetidos."*

La paginación por offset se rompe con inserciones/borrados; cursor es más estable, pero requiere backend compatible.

### Evidencia de Industria

- **APIs modernas:** Prefieren cursor para estabilidad y performance.
- **Retail:** Listas grandes con offset generan duplicados/perdidos con datos cambiantes.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Ítems duplicados o faltantes, scroll errático |
| **Técnico** | Más carga si se re-fetchan páginas enteras; inconsistencias con offset |
| **Económico** | Pérdida de ventas por UX pobre en catálogos grandes |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Offset simple sin orden estable | **INADECUADO:** Saltos y duplicados. |
| **ACEPTABLE** | Offset con orden fijo y locks | **MEJORA:** Menos saltos, pero vulnerable a cambios de datos. |
| **ENTERPRISE** | **Cursor con orden estable:** claves por item, siguiente/previo, cache local, retries idempotentes | **ÓPTIMO:** Scroll estable, menos carga, UX consistente. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Scroll estable usando cursor (next/prev). Reducir carga al pedir solo siguientes elementos. Combinar con cache local para offline breve. Reintentos seguros con misma clave de página. |
| **Restricciones Duras (NO permite)** | **Backend sin cursor:** Solo offset disponible. **Orden no estable:** Cambios en datos aún pueden reordenar; requiere campo de orden fijo. **Saltos de pagina:** Cursor no permite salto arbitrario fácilmente. |
| **Criterio de Selección** | Preferir cursor para listas dinámicas; offset solo si el backend no soporta cursor; ordenar por campo estable (fecha/id). |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Cursor | Puntero/clave a la posición en una lista para la siguiente página. |
| Offset | Desplazamiento numérico de filas a saltar. |
| Orden estable | Campo que mantiene el orden aunque haya inserciones/borrados. |
| Paginación | Dividir resultados en bloques solicitables. |
| Cache local | Almacenar páginas para reutilizarlas offline o al volver. |

---

## Referencias

- [Pagination Best Practices](https://developer.twitter.com/en/docs/twitter-api/pagination)
- [infinite_scroll_pagination](https://pub.dev/packages/infinite_scroll_pagination)
