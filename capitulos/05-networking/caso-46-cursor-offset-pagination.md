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

### Problema detectado (técnico)
- Offset con datos mutables causa duplicados/perdidos; depende de reloj/orden inestable.
- Sin cursor/orden estable, el scroll salta y la UX se degrada.
- Reintentos sin idempotencia de página pueden mezclar datos o repetir requests.

### Escenario de Negocio

> *"Como usuario, quiero explorar millones de productos sin saltos ni repetidos."*

### Incidentes reportados
- **APIs modernas:** Prefieren cursor para estabilidad/performance.
- **Retail:** Offset genera duplicados/perdidos con datos cambiantes.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| APIs modernas (Twitter/FB) | Global | Cursor reduce inconsistencias y carga. |
| Retail | Catálogos grandes | Offset produce saltos cuando hay inserciones/borrados. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; paginación/cache es área recurrente. |

**Resumen global**
- Cursor con orden estable mejora UX y reduce carga; offset es frágil con datos dinámicos.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Integration (CI) | Cursor mantiene orden y evita duplicados/perdidos | Móvil/Backend, CI |
| Performance | Latencia y peso de página vs offset | QA/Perf |
| Observabilidad | Eventos `pagination.*` con cursor/offset, errores | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Orden estable | Usar campo fijo (fecha/id) | Scroll predecible |
| Reintentos | Misma clave de página en retry | Idempotencia |
| Cache local | Reusar páginas para back/forward | UX suave |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Backend | Implementar cursor y next/prev tokens | Requiere soporte |
| Offset fallback | Solo si cursor no existe; ajustar UX | Contingencia |
| Saltos | Para saltar, usar índices server-side o search | Limitación |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Offset causa duplicados/saltos en datos dinámicos. |
| Opciones evaluadas | Offset simple; offset con orden fijo; cursor con orden estable. |
| Decisión | Cursor-based con orden estable y cache; offset solo si no hay cursor. |
| Consecuencias | Requiere cambios backend; UX de saltos limitada. |
| Riesgos aceptados | Backend sin cursor limita beneficio; datos muy dinámicos aún pueden mover items. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Duplicados/perdidos | 0 | Crítico si > 0 | UX consistente |
| Latencia de página | p95 competitiva | Warning si sube | Rendimiento |
| Consumo de datos | Menor vs offset | Alerta si no baja | Costos |
| Tickets por scroll errático | ↓ vs baseline | Alerta si no baja | Soporte |

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
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
