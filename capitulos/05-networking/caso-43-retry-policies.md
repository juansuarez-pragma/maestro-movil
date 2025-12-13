# Caso 43: El Timeout que Quebró el Banco
## Configurar Retry Policies Inteligentes

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | retry, timeout, resiliencia, políticas de reintento |
| **Patrón Técnico** | Exponential Backoff, Jitter, Circuit Breaker Light |
| **Stack Seleccionado** | Flutter + Dio interceptors + Riverpod para estado de red + backoff con jitter |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Retries sin backoff/jitter causan thundering herd y caídas; sin idempotencia, duplican cargos.
- Timeouts genéricos no se ajustan por operación; reintentos en red caída prolongada desperdician recursos.
- Sin telemetría de retries, no se detectan tormentas ni duplicados.

### Escenario de Negocio

> *"Como usuario, no quiero ver fallos al pagar por timeouts intermitentes; el sistema debe reintentar con inteligencia."*

### Incidentes reportados
- **SRE practices:** Backoff con jitter evita thundering herd.
- **Incidente bancario 2020:** Retries sin idempotencia generaron cargos duplicados.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Google/AWS SRE | Global | Backoff+jitter recomendado para resiliencia. |
| Incidentes fintech | Global | Retries sin idempotencia → doble cargo. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; manejo de errores/retries frecuente. |

**Resumen global**
- Retries inteligentes con idempotencia evitan duplicados y tormentas; requeridos en pagos críticos.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Doble cargo o caída de servicios por tormenta de retries |
| **UX** | Errores visibles en fallos transitorios |
| **Técnico** | Saturación de backend, timeouts en cascada |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Retries infinitos o sin backoff | **INADECUADO:** Thundering herd, más fallos. |
| **ACEPTABLE** | Retries fijos (pocos) | **MEJORA:** Menos riesgo, pero sin adaptación a latencia/errores. |
| **ENTERPRISE** | **Backoff exponencial con jitter + límites:** retries limitados, timeouts adecuados por operación, idempotencia garantizada, cancelación si sin red | **ÓPTIMO:** Resiliencia sin tormentas ni duplicados. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Configurar timeouts por tipo de operación. Retries con backoff y jitter. Cancelar si no hay red. Idempotency-Key para operaciones financieras. Telemetría de retries y motivos. |
| **Restricciones Duras (NO permite)** | **Sin idempotencia backend:** Riesgo de duplicados. **Operaciones no reintentar:** Algunas no son seguras (mutaciones sin idempotencia). **Red caída prolongada:** Retries deben detenerse. |
| **Criterio de Selección** | Interceptor en Dio para política uniforme; Riverpod para exponer estado de red; límites estrictos de intentos; idempotencia en operaciones críticas. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Backoff+jitter aplicado; límites respetados | Equipo móvil, CI |
| Integration (CI) | Reintentos con Idempotency-Key no duplican cargos | Móvil/Backend, CI |
| Observabilidad | Métricas `retry.*` (conteo, motivos, backoff) | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Estados | Mostrar “reintentando” y motivo; no spam de UI | Transparencia |
| Deshabilitar taps | Evitar múltiples envíos mientras reintenta | UX consistente |
| Red caída | Cortar retries si sin conectividad | Ahorra batería/datos |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Timeouts | Por operación (pago vs consulta) | Ajuste fino |
| Límites | Máx. intentos; evitar loops | Seguridad |
| Idempotencia | Requerida en mutaciones críticas | Evita duplicados |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Retries sin control duplican cargos o causan overload. |
| Opciones evaluadas | Retries ciegos; pocos retries; backoff+jitter+idempotencia. |
| Decisión | Backoff exponencial con jitter, límites, timeouts por operación, Idempotency-Key en críticas. |
| Consecuencias | Requiere soporte backend e instrumentación; tuning de timeouts. |
| Riesgos aceptados | Algunas operaciones no reintentables; dependencia de idempotencia server. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Cobros/pedidos duplicados | 0 | Crítico si > 0 | Confianza |
| Retries exitosos | Alta tasa sin overload | Alerta si fallan | Resiliencia |
| Overload/429 | Reducir eventos de herd | Alerta si suben | Estabilidad |
| Tickets por fallas transitorias | ↓ vs baseline | Alerta si no baja | Menos soporte |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Backoff exponencial | Incrementar el intervalo entre reintentos. |
| Jitter | Aleatorizar intervalos para evitar sincronía. |
| Idempotencia | Misma operación repetida no cambia el resultado final. |
| Thundering herd | Multiples clientes reintentando a la vez causando pico de carga. |
| Timeout | Tiempo límite para abortar una operación. |

---

## Referencias

- [Google SRE Book - Handling Overload](https://sre.google/sre-book/handling-overload/)
- [AWS Architecture Blog - Exponential Backoff](https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/)
