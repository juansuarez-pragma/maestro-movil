# Caso 49: [gRPC](#term-grpc "Framework RPC con HTTP/2 y protobuf.") en Flutter
## Comunicación de Alta Performance para Fintech

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | grpc, streaming, baja latencia, fintech |
| **Patrón Técnico** | [Bidirectional Streaming](#term-bidirectional-streaming "Cliente y servidor envían streams simultáneos."), [Proto](#term-proto "Archivo de definición de mensajes/servicios.") Contracts, TLS [mTLS](#term-mtls "Mutual TLS; verifica cliente y servidor.") |
| **Stack Seleccionado** | Flutter + gRPC/protobuf + interceptors + Riverpod para estados |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- REST para streams genera overhead y mayor latencia; contratos menos estrictos.
- Sin reconexión/backoff/heartbeats, gRPC se desconecta y la UI no se entera.
- TLS/mTLS mal configurado o proxies HTTP/2 bloqueados afectan disponibilidad.

### Escenario de Negocio

> *"Como app financiera, necesito streams de baja latencia y contratos tipados."*

### Incidentes reportados
- **Fintech/trading:** gRPC usado para streaming de mercado y operaciones rápidas.
- **Benchmarks:** gRPC supera REST en latencia y tamaño de payload.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Fintech/trading | Global | gRPC preferido para baja latencia y streams. |
| Benchmarks HTTP/2/gRPC | Global | Menor latencia/payload que REST. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; redes/seguridad son críticas. |

**Resumen global**
- gRPC reduce overhead/latencia para streams, pero requiere manejo de reconexión/seguridad y compatibilidad de red.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Técnico** | Setup TLS/mTLS más complejo, compatibilidad limitada en algunos proxies |
| **Operacional** | Monitoreo y errores menos visibles que REST si no se instrumenta |
| **UX** | Desconexiones silenciosas si no se manejan heartbeats/retries |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Usar REST para todo, incluido streaming | **INADECUADO:** Overhead, peor latencia. |
| **ACEPTABLE** | gRPC unidireccional sin manejo de desconexiones | **MEJORA:** Mejor latencia, pero frágil ante cortes. |
| **ENTERPRISE** | **gRPC robusto:** TLS/mTLS, reconexión con backoff, heartbeats, retries seguros, contratos versionados | **ÓPTIMO:** Baja latencia confiable y segura. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Streaming bidi/unidireccional de mercado. Contratos proto tipados. Interceptores para auth/tracing. Heartbeats y reconexión. mTLS para seguridad. |
| **Restricciones Duras (NO permite)** | **Compatibilidad proxies/redes:** Algunos entornos bloquean HTTP/2. **Payloads binarios:** Debug más difícil. **Generación de código:** Requiere build_runner/protoc. |
| **Criterio de Selección** | gRPC para casos de baja latencia/streaming; REST para compatibilidad amplia. Configurar seguridad y reconexión desde el inicio. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Integration (CI) | Reconexión/backoff/heartbeat operan; streams continúan | Móvil/Backend, CI |
| Seguridad | TLS/mTLS configurado correctamente | QA/Seguridad |
| Observabilidad | Métricas `grpc.*` (latencia, retries, disconnects) | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Fallback | Detectar bloqueo HTTP/2 → fallback a REST cuando aplique | Disponibilidad |
| Heartbeats | Ajustar frecuencia para balance perf/batería | Estabilidad |
| Errores | Mensajes claros en desconexión y reintento | UX transparente |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Generación de código | Automatizar con build_runner; revisar cambios de proto | Cohesión |
| Compatibilidad | Probar en redes/proxies variados | Robustez |
| Versionado | Contratos proto versionados y backward-compatible | Evita rupturas |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | REST/streams lentos; gRPC sin manejo de reconexión/seguridad es frágil. |
| Opciones evaluadas | REST; gRPC básico; gRPC con reconexión, mTLS y fallback. |
| Decisión | gRPC con reconexión/backoff/heartbeats, mTLS, interceptores; fallback a REST en bloqueo. |
| Consecuencias | Mayor complejidad de red/seguridad; requiere tooling de proto. |
| Riesgos aceptados | Entornos bloquean HTTP/2; binarios más difíciles de debug. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Latencia de streams | p95 menor que REST baseline | Warning si sube | UX ágil |
| Disponibilidad en red adversa | Fallback correcto | Alerta si falla | Resiliencia |
| Desconexiones/retries | Controlados con backoff | Alerta si explotan | Estabilidad |
| Tickets “se cae el stream” | ↓ vs baseline | Alerta si no baja | Soporte |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-grpc"></a>gRPC | Framework RPC con HTTP/2 y protobuf. |
| <a id="term-bidirectional-streaming"></a>Bidirectional Streaming | Cliente y servidor envían streams simultáneos. |
| <a id="term-mtls"></a>mTLS | Mutual TLS; verifica cliente y servidor. |
| <a id="term-proto"></a>Proto | Archivo de definición de mensajes/servicios. |
| <a id="term-interceptor"></a>Interceptor | Hook para añadir auth/tracing/logging en gRPC. |

---

## Referencias

- [gRPC for Mobile](https://grpc.io/docs/platforms/mobile/)
- [Mutual TLS](https://cloud.google.com/load-balancing/docs/ssl-certificates/ssl-policies-concepts)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
