# Caso 68: Push Notifications Seguras
## Acciones Sensibles desde Notificaciones sin Exponer PII

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | push, notificaciones, MFA, aprobación, PII, phishing |
| **Patrón Técnico** | Secure Push, Payload Minimization, Signed Actions |
| **Stack Seleccionado** | Flutter + FCM/APNs + acciones nativas + validación backend + observabilidad |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Payload con datos sensibles (PII/montos) en push se filtra en lockscreen o logs.
- Acciones “approve/deny” sin validación server-side permiten spoofing o replay.
- Sin tracking end-to-end (push_id/trace_id), no se diagnostican fallos de entrega.

### Escenario de Negocio

> *"Como usuario, quiero aprobar operaciones y alertas críticas desde notificaciones sin comprometer mi seguridad."*

### Incidentes reportados
- **Exposición en lockscreen:** notificaciones mostraron montos/datos personales sin control de privacidad.
- **Spoofing:** acciones basadas en payload sin verificación permitieron aprobaciones indebidas.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Apple (UserNotifications) | Global | Recomendación: minimizar payload y controlar presentación en pantalla bloqueada. |
| Android notifications | Global | Buenas prácticas: canales, visibilidad y acciones que requieren confirmación/servidor. |
| OWASP MAS | Global | Minimización de datos y validación server-side para flujos críticos. |

**Resumen global**
- Push debe transportar “señales”, no datos sensibles: payload mínimo + fetch seguro + validación en backend + expiración.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Seguridad/Privacidad** | PII en lockscreen, spoofing de acciones |
| **UX** | Notificaciones irrelevantes o rotas elevan churn |
| **Operacional** | Sin trazas, debugging de entrega es lento |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Payload con PII + acción local sin verificación | **INADECUADO:** Exposición y spoofing. |
| **ACEPTABLE** | [Payload mínimo](#term-payload-minimo "Notificación con identificador, sin datos sensibles.") pero sin expiración/telemetría | **MEJORA:** Menos PII, pero aún sin control operacional. |
| **ENTERPRISE** | **Push seguro:** payload mínimo (id), fetch de detalle autenticado, acciones firmadas/one-time, expiración, controles de lockscreen, telemetría end-to-end | **ÓPTIMO:** Seguridad y operabilidad. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Acciones rápidas con confirmación server-side. Minimizar PII en notificaciones. Controlar visibilidad (public/private). [Correlación](#term-correlacion "Enlazar push_id con trace_id para diagnóstico.") `push_id` ↔ `trace_id`. Expiración de acciones. |
| **Restricciones Duras (NO permite)** | **Entrega no garantizada:** push puede retrasarse o perderse. **Offline:** acciones requieren conectividad. **Políticas de OS:** iOS/Android limitan background. |
| **Criterio de Selección** | Payload mínimo + fetch autenticado; acciones siempre validadas por backend; políticas de visibilidad y telemetría. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Seguridad | No PII en payload; acciones one-time | Seguridad/QA |
| Integration (Android/iOS) | Acciones y canales/visibilidad operan | Móvil/QA |
| Observabilidad | Métricas `push.delivery` y `push.action` | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Privacidad | “Contenido sensible” oculto en lockscreen | Confianza |
| Offline | Si no hay red, abrir app y revalidar | Seguridad |
| Relevancia | Categorías, quiet hours, rate limit | Menos spam |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Expiración | Acciones vencen rápido (minutos) | Anti-replay |
| Telemetría | `push_id`, `device_id`, `app_version` | Diagnóstico |
| Fallback | SMS/Email solo si es seguro y necesario | Continuidad |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | PII expuesta y spoofing de acciones via push. |
| Opciones evaluadas | Payload con PII; payload mínimo sin validación; push seguro con fetch + validación + expiración. |
| Decisión | Payload mínimo (id) + fetch autenticado + acciones one-time validadas en backend + control lockscreen. |
| Consecuencias | Requiere backend para “acciones” y trazabilidad; más llamadas. |
| Riesgos aceptados | Entrega tardía; dependencia de conectividad. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| PII en push | 0 | Crítico si >0 | Privacidad |
| Éxito de acciones | > 99% | Warning si baja | UX |
| Spoof/replay | 0 | Crítico si >0 | Seguridad |
| Entrega p95 | Medida por cohorte | Alerta si empeora | Operación |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-payload-minimo"></a>Payload mínimo | Notificación con identificador, sin datos sensibles. |
| <a id="term-lockscreen-privacy"></a>Lockscreen privacy | Política para ocultar contenido sensible en pantalla bloqueada. |
| <a id="term-accion-one-time"></a>Acción one-time | Acción válida una vez, con expiración. |
| <a id="term-correlacion"></a>Correlación | Enlazar `push_id` con `trace_id` para diagnóstico. |
| <a id="term-fetch-autenticado"></a>Fetch autenticado | Obtener detalle desde backend con auth. |

---

## Referencias

- [Apple UserNotifications](https://developer.apple.com/documentation/usernotifications)
- [Android Notifications](https://developer.android.com/develop/ui/views/notifications)
- [OWASP MAS](https://mas.owasp.org/)
