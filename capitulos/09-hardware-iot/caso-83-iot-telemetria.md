# Caso 83: Telemetría IoT Confiable
## Enviar Datos de Dispositivos con Calidad y Seguridad

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | IoT, telemetría, mqtt, calidad de servicio, seguridad |
| **Patrón Técnico** | [MQTT](#term-mqtt "Protocolo ligero de mensajería para IoT.") [QoS](#term-qos "Calidad de servicio en entrega de mensajes."), Offline Buffering, Secure Telemetry |
| **Stack Seleccionado** | Flutter + MQTT/HTTPS + buffer local + TLS/[mTLS](#term-mtls "Autenticación mutua TLS.") |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- HTTP sin QoS ni reintento pierde telemetría y deja monitoreo incompleto.
- Sin buffer offline, los mensajes se pierden ante desconexión.
- Sin TLS/mTLS y [ACL](#term-acl "Lista de control de acceso por topic."), la telemetría puede ser interceptada o alterada.

### Escenario de Negocio

> *"Como app, debo enviar telemetría de dispositivos/IoT garantizando entrega y seguridad."*

### Incidentes reportados
- **Sistemas IoT:** Pérdida de datos por QoS 0/HTTP sin reintentos.
- **Seguridad:** Telemetría interceptada al no cifrar ni usar ACL.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Implementaciones IoT | Global | MQTT con QoS y buffer es estándar para confiabilidad. |
| Reportes de seguridad | Global | Falta de TLS/ACL expone datos; mTLS mitiga suplantación. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; comunicaciones inseguras son comunes. |

**Resumen global**
- MQTT con QoS adecuado, buffer y seguridad (TLS/mTLS, ACL) reduce pérdida y exposición de telemetría; HTTP sin garantías es insuficiente.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Fiabilidad | QoS 1/2 entrega sin pérdida y sin duplicados | QA/Móvil |
| Seguridad | TLS/mTLS y ACL aplicados | Seguridad |
| Offline | Buffer y reenvío en reconexión | Móvil/QA |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Retries | Backoff y límites para no agotar batería | UX/Recursos |
| Indicadores | Estado de conexión/cola visible para soporte | Operabilidad |
| Payload | Minimizar tamaño y evitar PII | Seguridad |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Certs | Rotación y gestión de mTLS | Seguridad |
| Monitoreo | Métricas `mqtt.*` (latencia, drop, buffer) | Observabilidad |
| Límites | Capacidad de buffer y expiración configuradas | Previene bloat |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Pérdida/exposición de telemetría IoT por falta de QoS y seguridad. |
| Opciones evaluadas | HTTP sin garantías; MQTT con QoS básico; MQTT con QoS, buffer, TLS/mTLS y ACL. |
| Decisión | MQTT con QoS adecuado, buffer offline y seguridad (TLS/mTLS, ACL). |
| Consecuencias | Necesita gestión de certs y monitoreo; overhead de QoS 2 en casos críticos. |
| Riesgos aceptados | Costo de infra y complejidad de certificados. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Pérdida de telemetría | 0 | Crítico si >0 | Confiabilidad |
| Retries/duplicados | Controlados según QoS | Alerta si crecen | Eficiencia |
| Fallos de TLS/mTLS | Tendencia a la baja | Crítico si sube | Seguridad |
| Tamaño de buffer | Bajo y controlado | Alerta si crece | Recursos |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-mqtt"></a>MQTT | Protocolo ligero de mensajería para IoT. |
| <a id="term-qos"></a>QoS | Calidad de servicio en entrega de mensajes. |
| <a id="term-mtls"></a>mTLS | Autenticación mutua TLS. |
| <a id="term-acl"></a>ACL | Lista de control de acceso por topic. |
| <a id="term-buffer-offline"></a>Buffer offline | Almacenamiento temporal de mensajes sin red. |

---

## Referencias

- [MQTT Specification](http://mqtt.org/)
- [TLS/mTLS](https://www.rfc-editor.org/rfc/rfc5246)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
