# Caso 49: gRPC en Flutter
## Comunicación de Alta Performance para Fintech

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | grpc, streaming, baja latencia, fintech |
| **Patrón Técnico** | Bidirectional Streaming, Proto Contracts, TLS mTLS |
| **Stack Seleccionado** | Flutter + gRPC/protobuf + interceptors + Riverpod para estados |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como app financiera, necesito streams de baja latencia y contratos tipados."*

REST puede ser pesado para streams; gRPC reduce overhead y provee contratos fuertes.

### Evidencia de Industria

- **Fintech/trading:** gRPC usado para streaming de mercado y operaciones rápidas.
- **Benchmarks:** gRPC supera REST en latencia y tamaño de payload.

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

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| gRPC | Framework RPC con HTTP/2 y protobuf. |
| Bidirectional Streaming | Cliente y servidor envían streams simultáneos. |
| mTLS | Mutual TLS; verifica cliente y servidor. |
| Proto | Archivo de definición de mensajes/servicios. |
| Interceptor | Hook para añadir auth/tracing/logging en gRPC. |

---

## Referencias

- [gRPC for Mobile](https://grpc.io/docs/platforms/mobile/)
- [Mutual TLS](https://cloud.google.com/load-balancing/docs/ssl-certificates/ssl-policies-concepts)
