# Caso 36: Cache Invalidation Hell
## Cuándo Confiar y Cuándo Descartar Datos Locales

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | cache, invalidación, frescura de datos, TTL |
| **Patrón Técnico** | Cache Invalidation Policies, Stale-While-Revalidate, E-Tag |
| **Stack Seleccionado** | Flutter + SQLite/Isar cache + E-Tag/If-None-Match + Riverpod |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero datos frescos sin esperar cargas pesadas, y sin ver información obsoleta peligrosa."*

Invalidar cache incorrectamente genera datos viejos o demasiado fetch, afectando UX y costos.

### Evidencia de Industria

- **CDNs y APIs:** E-Tag y cache policies son estándar para equilibrio entre frescura y costo.
- **Apps bancarias:** Datos stale pueden inducir errores financieros.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico/UX** | Datos viejos → decisiones erróneas; exceso de fetch → costo y batería |
| **Técnico** | Estados inconsistentes entre cache y servidor |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Cache sin TTL ni validación | **INADECUADO:** Datos stale sin control. |
| **ACEPTABLE** | TTL fijo y refresh en apertura | **MEJORA:** Control básico, pero no considera cambios en servidor. |
| **ENTERPRISE** | **Políticas adaptativas:** E-Tag/If-None-Match, stale-while-revalidate, TTL por tipo de dato, invalidación dirigida | **ÓPTIMO:** Balance frescura/costo, menor latencia, controlado. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Usar E-Tags para validar cambios sin descargar todo. TTL por recurso crítico vs no crítico. SWR: servir cache inmediato y refrescar en segundo plano. Invalidar selectivamente tras mutaciones. |
| **Restricciones Duras (NO permite)** | **Sin soporte backend:** E-Tag requiere servidor. **Mutaciones concurrentes:** Puede requerir re-fetch post-mutate. **Datos extremadamente dinámicos:** TTL corto o sin cache. |
| **Criterio de Selección** | Combinar E-Tag para validar, TTL por recurso, SWR para UX rápida, invalidación dirigida tras cambios locales. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| E-Tag | Hash/versión del recurso para validación condicional. |
| TTL | Tiempo que un recurso permanece válido en cache. |
| Stale-While-Revalidate | Servir datos cacheados y refrescar en background. |
| If-None-Match | Header para validar si un recurso cambió. |
| Invalidación dirigida | Borrar/actualizar solo recursos afectados por una mutación. |

---

## Referencias

- [HTTP Caching (RFC 7234)](https://httpwg.org/specs/rfc7234.html)
- [Stale-While-Revalidate Pattern](https://web.dev/stale-while-revalidate/)
