# Caso 45: Request Deduplication
## Evitar Doble Cobro por Doble Tap

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | deduplicación, doble tap, requests duplicados, idempotencia |
| **Patrón Técnico** | In-flight Request Deduplication, Idempotent Client, [Mutex](#term-mutex "Exclusión mutua para serializar requests iguales.") per Endpoint |
| **Stack Seleccionado** | Flutter + Dio interceptors + in-flight map + [Idempotency-Key](#term-idempotency-key "Token único que evita doble procesamiento en backend.") |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Doble tap o reintentos simultáneos generan requests duplicados → doble cobro si no hay dedup/idempotencia.
- Solo deshabilitar UI no cubre duplicados programáticos ni multi-hilos.
- Sin mapa de in-flight y hash de payload, se saturan backend y hay respuestas inconsistentes.

### Escenario de Negocio

> *"Como usuario, un doble tap no debe generar dos cobros."*

### Incidentes reportados
- **Pagos móviles:** Doble tap = doble cobro sin idempotencia.
- **PSPs:** Recomiendan Idempotency-Key y dedup cliente/servidor.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| PSPs | Global | Idempotencia/dedup obligatoria en pagos. |
| Casos móviles | Retail/finanzas | Duplicados por taps/retries sin dedup. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; manejo de requests es fuente de bugs. |

**Resumen global**
- Dedup + Idempotency-Key reducen duplicados y carga; UI sola no basta.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Cobros duplicados, reembolsos costosos |
| **UX** | Mensajes de error inconsistentes; confusión del usuario |
| **Técnico** | Requests redundantes saturan el backend |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Deshabilitar botón en UI solamente | **INADECUADO:** No cubre multi-requests programáticos ni reintentos. |
| **ACEPTABLE** | [Throttle/Debounce](#term-throttle-debounce "Limitar frecuencia de acciones/requests.") de acciones | **MEJORA:** Reduce taps, pero no elimina duplicados in-flight. |
| **ENTERPRISE** | **Dedup + idempotencia:** mantener mapa de requests en vuelo, rechazar duplicados, usar Idempotency-Key en pagos | **ÓPTIMO:** Previene cobros duplicados y reduce carga. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Identificar requests equivalentes por endpoint/payload. Reusar la misma future/respuesta para duplicados. Adjuntar Idempotency-Key para pagos. Cancelar duplicados opcionales. |
| **Restricciones Duras (NO permite)** | **Mutaciones no idempotentes:** Backend debe soportar. **Bindeo excesivo:** Dedup agresivo puede bloquear solicitudes legítimas si el payload difiere. **Estado compartido:** Dedup es local al cliente; múltiples dispositivos aún requieren idempotencia server-side. |
| **Criterio de Selección** | Interceptor central para dedup; claves basadas en método+path+payload hash; Idempotency-Key en pagos; UI también deshabilita acciones críticas. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Mapa de in-flight reusa future/respuesta | Equipo móvil, CI |
| Integration (CI) | Doble tap/reintento produce un solo efecto | Móvil/Backend, CI |
| Observabilidad | Eventos `request.dedup` con hash/key, duplicados evitados | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| UI | Deshabilitar acciones críticas mientras hay request | Menos taps duplicados |
| Feedback | Mostrar estado “procesando” y resultado claro | UX consistente |
| Errores | Mensajes coherentes al bloquear duplicados | Claridad |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Hash de payload | Incluir método+path+payload; considerar headers relevantes | Evita falsos duplicados |
| Idempotencia server | Requerida para pagos/órdenes | Defensa en profundidad |
| Alcance | Dedup local; multi-dispositivo depende del backend | Cobertura |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Requests duplicados por taps/reintentos causan doble cobro. |
| Opciones evaluadas | Solo UI throttle; dedup parcial; dedup + Idempotency-Key. |
| Decisión | Interceptor de dedup in-flight + Idempotency-Key en mutaciones críticas + UI deshabilitada. |
| Consecuencias | Complejidad de hashing/almacenamiento de in-flight; requiere soporte backend. |
| Riesgos aceptados | Falsos positivos si hash mal definido; cobertura local. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Cobros/pedidos duplicados | 0 | Crítico si > 0 | Confianza |
| Requests duplicados evitados | Alto ratio de dedup | Alerta si baja | Carga reducida |
| Tickets por doble tap | ↓ vs baseline | Alerta si no baja | Soporte controlado |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-in-flight-dedup"></a>In-flight dedup | Reutilizar la misma respuesta para requests concurrentes iguales. |
| <a id="term-idempotency-key"></a>Idempotency-Key | Token único que evita doble procesamiento en backend. |
| <a id="term-throttle-debounce"></a>Throttle/Debounce | Limitar frecuencia de acciones/requests. |
| <a id="term-payload-hash"></a>Payload hash | Hash del cuerpo/payload para identificar duplicados. |
| <a id="term-mutex"></a>Mutex | Exclusión mutua para serializar requests iguales. |

---

## Referencias

- [Stripe Idempotency](https://stripe.com/docs/idempotency)
- [Dio Interceptors](https://pub.dev/packages/dio)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
