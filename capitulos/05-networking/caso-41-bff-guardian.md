# Caso 41: [BFF](#term-bff "Backend for Frontend; capa dedicada para un cliente (móvil/web).") — El Guardián del Móvil
## Orquestar 7 Microservicios en Una Llamada

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | BFF, orquestación, microservicios, latencia, agregación |
| **Patrón Técnico** | Backend for Frontend (BFF), [Aggregation](#term-aggregation "Combinar respuestas de múltiples servicios en una sola."), [Response Shaping](#term-response-shaping "Ajustar payloads a las necesidades del cliente.") |
| **Stack Seleccionado** | Flutter + Dio + BFF REST/GraphQL + Riverpod para cache |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- El cliente móvil orquesta 7 microservicios: latencia alta, errores parciales y lógica duplicada.
- Cambios en contratos de dominio rompen el cliente; sin BFF, el mobile debe versionar cada servicio.
- Falta de cache/response shaping aumenta payload y roundtrips.

### Escenario de Negocio

> *"Como usuario móvil, quiero ver mi dashboard sin esperar a 7 llamadas secuenciales."*

### Incidentes reportados
- **Netflix/Shopify:** BFFs móviles reducen latencia/payload en dispositivos limitados.
- **Banca:** Dashboards con múltiples agregaciones requieren capa dedicada.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Netflix/Shopify tech blogs | Global | BFF reduce roundtrips y payload; contratos por cliente. |
| Equipos banca | LATAM/EU | Dashboards multi-servicio generan latencia/errores sin BFF. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; payload/latencia es hallazgo común. |

**Resumen global**
- Sin BFF, la latencia y la fragilidad de contratos escalan; BFF reduce roundtrips, versiona contratos y cachea.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Pantallas lentas, timeouts |
| **Técnico** | Lógica duplicada en cliente, mayor complejidad y bugs |
| **Reputacional/Económico** | Menor conversión si el dashboard tarda |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Cliente orquesta múltiples APIs directamente | **INADECUADO:** Latencia alta, acoplamiento, cambios frecuentes. |
| **ACEPTABLE** | Proxy simple sin agregación | **MEJORA:** Un endpoint, pero sin reducir payload ni complejidad. |
| **ENTERPRISE** | **BFF completo:** agregación, cache server-side, response shaping, contratos específicos por pantalla | **ÓPTIMO:** Menos roundtrips, contratos estables, menor lógica en cliente. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Reducir llamadas a una sola por pantalla. Adaptar payloads al modelo de UI. Aplicar cache server-side y e-tags. Versionar contratos sin romper cliente. |
| **Restricciones Duras (NO permite)** | **Sin BFF evolutivo:** Cambios de dominio requieren actualizar BFF. **Escalabilidad:** BFF puede ser cuello de botella sin autoscaling. **Consistencia:** Datos pueden estar stale si cache no se invalida bien. |
| **Criterio de Selección** | BFF para reducir latencia y acoplamiento; GraphQL opcional para response shaping; Riverpod para cache local y estados claros. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Integration (CI) | Dashboard usa 1 llamada al BFF; errores parciales manejados | Móvil/Backend, CI |
| Performance | Latencia y tamaño de payload vs baseline | QA/Perf |
| Observabilidad | Eventos `bff.*` con latencia, cache hit/miss | Móvil/SRE |
| Contratos | Versiones coexistentes; adapters funcionan | Móvil/QA |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Loading parcial | Placeholder por sección; degradar si faltan datos | UX estable |
| Cache server-side | [E-Tag](#term-e-tag "Hash de recurso para validar/inutilizar cache.")/stale-while-revalidate donde aplique | Menos latencia |
| Versionado | Headers de versión; flags para activar nuevos contratos | Migraciones seguras |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Escalabilidad | Autoscaling del BFF; circuit breaker hacia downstream | Resiliencia |
| Observabilidad | Trazas por agregación; detectar servicios lentos | MTTR menor |
| Caducidad | Invalidation dirigida tras mutaciones | Consistencia |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Latencia/fragilidad por orquestar 7 servicios desde el cliente. |
| Opciones evaluadas | Cliente orquesta; proxy simple; BFF con agregación/cache/versionado. |
| Decisión | BFF completo con contratos por pantalla, cache, versionado y observabilidad. |
| Consecuencias | Operar y escalar el BFF; mayor responsabilidad de backend. |
| Riesgos aceptados | Stale si cache mal invalidado; BFF como punto único crítico. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Latencia de dashboard | p95 reducida vs baseline | Warning si no mejora | UX rápida |
| Roundtrips | 1 llamada por pantalla principal | Alerta si sube | Menos fallos |
| Tamaño de payload | ↓ con response shaping/cache | Alerta si crece | Costos/red |
| Errores parciales | Manejados con degradación | Crítico si rompen UI | Estabilidad |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-bff"></a>BFF | Backend for Frontend; capa dedicada para un cliente (móvil/web). |
| <a id="term-aggregation"></a>Aggregation | Combinar respuestas de múltiples servicios en una sola. |
| <a id="term-response-shaping"></a>Response Shaping | Ajustar payloads a las necesidades del cliente. |
| <a id="term-e-tag"></a>E-Tag | Hash de recurso para validar/inutilizar cache. |
| <a id="term-latencia-percibida"></a>Latencia percibida | Tiempo que el usuario percibe antes de ver datos útiles. |

---

## Referencias

- [Pattern: Backend for Frontend](https://samnewman.io/patterns/architectural/bff/)
- [Netflix BFF Lessons](https://netflixtechblog.com/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
