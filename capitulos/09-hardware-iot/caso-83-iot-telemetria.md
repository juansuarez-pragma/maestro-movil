# Caso 83: Telemetría IoT Confiable
## Enviar Datos de Dispositivos con Calidad y Seguridad

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | IoT, telemetría, mqtt, calidad de servicio, seguridad |
| **Patrón Técnico** | MQTT QoS, Offline Buffering, Secure Telemetry |
| **Stack Seleccionado** | Flutter + MQTT/HTTPS + buffer local + TLS/mTLS |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como app, debo enviar telemetría de dispositivos/IoT garantizando entrega y seguridad."*

Sin QoS y seguridad, se pierden datos o se exponen, afectando monitoreo y cumplimiento.

### Evidencia de Industria

- **Sistemas IoT:** MQTT con QoS y buffering es estándar.
- **Seguridad:** TLS/mTLS y auth son obligatorios para datos sensibles.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Técnico** | Pérdida de mensajes, datos corruptos |
| **Seguridad** | Interceptación/manipulación de telemetría |
| **Operacional** | Monitoreo incorrecto y decisiones erróneas |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Enviar telemetría por HTTP sin reintento/QoS | **INADECUADO:** Pérdida de datos, sin garantías. |
| **ACEPTABLE** | MQTT QoS 0/1 con reconexión básica | **MEJORA:** Mejor que nada, pero sin buffer ni seguridad fuerte. |
| **ENTERPRISE** | **MQTT QoS + buffer + seguridad:** QoS 1/2 según necesidad, buffer offline, TLS/mTLS, autenticación y topics autorizados | **ÓPTIMO:** Entrega confiable y segura. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | QoS adecuado (1/2) para delivery garantizado. Buffer local de mensajes y reenvío al reconectar. TLS/mTLS para transporte seguro. Auth y ACL por topic. Heartbeats/keep-alive configurables. |
| **Restricciones Duras (NO permite)** | **QoS 2 overhead:** Más latencia; usar solo cuando necesario. **Offline largo:** Buffer puede crecer; requiere límites y expiración. **Certs:** mTLS requiere gestión de certificados. |
| **Criterio de Selección** | MQTT para telemetría; QoS según criticidad; buffer con límites; seguridad con TLS/mTLS y ACL; monitoreo de conexión. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| MQTT | Protocolo ligero de mensajería para IoT. |
| QoS | Calidad de servicio en entrega de mensajes. |
| mTLS | Autenticación mutua TLS. |
| ACL | Lista de control de acceso por topic. |
| Buffer offline | Almacenamiento temporal de mensajes sin red. |

---

## Referencias

- [MQTT Specification](http://mqtt.org/)
- [TLS/mTLS](https://www.rfc-editor.org/rfc/rfc5246)
