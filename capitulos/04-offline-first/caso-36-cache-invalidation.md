# Caso 36: Cache Invalidation Hell
## Cuándo Confiar y Cuándo Descartar Datos Locales

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | cache, invalidación, frescura de datos, [TTL](#term-ttl "Tiempo que un recurso permanece válido en cache.") |
| **Patrón Técnico** | Cache Invalidation Policies, [Stale-While-Revalidate](#term-stale-while-revalidate "Servir datos cacheados y refrescar en background."), [E-Tag](#term-e-tag "Hash/versión del recurso para validación condicional.") |
| **Stack Seleccionado** | Flutter + SQLite/Isar cache + E-Tag/[If-None-Match](#term-if-none-match "Header para validar si un recurso cambió.") + Riverpod |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Cache sin TTL ni validación sirve datos stale; TTL fijo sin E-Tag fuerza fetch innecesario.
- Sin SWR ni invalidación dirigida, UX sufre (datos viejos) o se dispara el consumo/red.
- Mutaciones sin invalidación correcta dejan cache inconsistente con backend.

### Escenario de Negocio

> *"Como usuario, quiero datos frescos sin esperar cargas pesadas, y sin ver información obsoleta peligrosa."*

### Incidentes reportados
- **CDNs/APIs:** E-Tag y políticas de cache son estándar para balance frescura/costo.
- **Apps bancarias:** Datos stale inducen errores financieros.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| CDNs/APIs | Global | E-Tag/If-None-Match reduce ancho de banda y mantiene frescura. |
| Apps bancarias | Global | Datos stale generan reclamos y riesgo operativo. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; cache/frescura es hallazgo frecuente. |

**Resumen global**
- Invalidación pobre causa datos stale o exceso de fetch; E-Tag/SWR balancean frescura y costo.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Manejo de E-Tag/If-None-Match y SWR | Equipo móvil, CI |
| Integration (CI) | Invalidación tras mutaciones; TTL por recurso se respeta | Móvil/Backend, CI |
| Observabilidad | Eventos `cache.*` con hit/miss, staleness, invalidaciones | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| SWR | Servir cache inmediato, refrescar en background | UX rápida |
| TTL | Diferenciar críticos (TTL corto) vs no críticos (TTL largo) | Balance costo/frescura |
| Errores | Manejar 304/not modified y mostrar estado de frescura | Transparencia |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| [Invalidación dirigida](#term-invalidacion-dirigida "Borrar/actualizar solo recursos afectados por una mutación.") | Borrar/actualizar recursos afectados por mutaciones | Consistencia |
| Fallback | Full fetch si E-Tag falla o backend no soporta | Robustez |
| Auditoría | Log de invalidaciones y staleness para diagnóstico | Soporte |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Cache stale o fetch excesivo por invalidación deficiente. |
| Opciones evaluadas | Cache sin TTL; TTL fijo; E-Tag + SWR + TTL por recurso + invalidación dirigida. |
| Decisión | E-Tag/If-None-Match + TTL por recurso + SWR + invalidación dirigida tras mutaciones. |
| Consecuencias | Depende de soporte backend; más lógica de cache. |
| Riesgos aceptados | Backend sin E-Tag reduce beneficio; datos muy dinámicos exigen TTL corto. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Hit rate de cache | Alto con datos frescos | Alerta si baja | Menor latencia/costos |
| [Staleness](#term-staleness "Grado de antigüedad aceptable de los datos.") | Controlada según TTL | Alerta si datos stale | UX confiable |
| Uso de red | Reducido vs baseline | Alerta si sube | Eficiencia |
| Errores 304/invalidación | Bajo y manejado | Alerta si repetidos | Consistencia |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-e-tag"></a>E-Tag | Hash/versión del recurso para validación condicional. |
| <a id="term-ttl"></a>TTL | Tiempo que un recurso permanece válido en cache. |
| <a id="term-stale-while-revalidate"></a>Stale-While-Revalidate | Servir datos cacheados y refrescar en background. |
| <a id="term-if-none-match"></a>If-None-Match | Header para validar si un recurso cambió. |
| <a id="term-invalidacion-dirigida"></a>Invalidación dirigida | Borrar/actualizar solo recursos afectados por una mutación. |
| <a id="term-staleness"></a>Staleness | Grado de antigüedad aceptable de los datos. |

---

## Referencias

- [HTTP Caching (RFC 7234)](https://httpwg.org/specs/rfc7234.html)
- [Stale-While-Revalidate Pattern](https://web.dev/stale-while-revalidate/)
- [Cache Invalidation](https://martinfowler.com/bliki/CacheInvalidation.html)
