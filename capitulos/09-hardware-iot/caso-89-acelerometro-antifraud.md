# Caso 89: Acelerómetro Anti-Fraude
## Detectar Comportamiento de Bot con Sensores

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | acelerómetro, antifraude, bot detection, sensores |
| **Patrón Técnico** | Sensor-based Fraud Detection, Behavioral Biometrics, Anomaly Detection |
| **Stack Seleccionado** | Flutter + sensores nativos + análisis local + envío de features a backend |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Sin señales de sensores, bots/automatizaciones no se distinguen de usuarios.
- Colecta sin control puede violar privacidad o drenar batería.
- Sin modelos/umbrales, los falsos positivos disparan bloqueos injustos.

### Escenario de Negocio

> *"Como app, quiero detectar bots o automatizaciones usando patrones de movimiento/dispositivo."*

### Incidentes reportados
- **Fraude móvil:** Bots emulan taps; señales de sensores ayudan a detectar patrones no humanos.
- **Privacidad/batería:** Colectas agresivas generaron quejas y desinstalaciones.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| [Behavioral biometrics](#term-behavioral-biometrics "Identificación por patrones de uso/movimiento.") | Global | Acelerómetro/giroscopio se usan para señales anti-bot. |
| Investigación anti-fraude | Global | Modelos requieren datos limpios y minimizados para reducir falsos positivos. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; manejo de datos/sensores es brecha común. |

**Resumen global**
- Señales de sensores con features y scoring controlado ayudan a detectar bots; se necesitan consentimiento, minimización y límites de batería para evitar efectos negativos.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Seguridad** | Automatizaciones que ejecutan fraude sin ser detectadas |
| **UX/Privacidad** | Recolección de sensores debe ser transparente y limitada |
| **Operacional** | Falsos positivos si el modelo es deficiente |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Sin señales de sensores | **INADECUADO:** Menor capacidad anti-bot. |
| **ACEPTABLE** | Colecta básica de acelerómetro | **MEJORA:** Señales, pero sin análisis ni modelos robustos. |
| **ENTERPRISE** | **Features + modelo:** colecta controlada, extracción de features locales, envío a backend para scoring, privacidad y sampling | **ÓPTIMO:** Señal robusta con menor riesgo de falsos positivos. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Capturar movimientos durante interacciones críticas. Extraer features (varianza, frecuencia). Enviar features anonimizadas para scoring. Aplicar step-up/bloqueo ante riesgo. |
| **Restricciones Duras (NO permite)** | **Privacidad:** Requiere consentimiento y minimización. **Costo battery:** [Sampling](#term-sampling "Captura de datos a intervalos controlados.") debe ser limitado. **Modelos:** Necesita entrenamiento y actualización; riesgo de falsos positivos. |
| **Criterio de Selección** | Recolección mínima necesaria, features calculadas localmente, consentimientos claros, sampling y límites de battery, backend de scoring. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Datos | Consentimiento y minimización de sensores | Seguridad/Legal |
| Modelo | Tasa de falsos positivos/negativos | Data/QA |
| Batería | Consumo dentro de budget | QA/Perf |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Transparencia | Explicar uso de sensores y permitir opt-out | Confianza |
| Sampling | Solo en eventos críticos; limitar duración | Batería |
| Step-up | OTP/bio si score alto | Riesgo controlado |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Retención | Guardar features anonimizadas por tiempo limitado | Privacidad |
| Observabilidad | Eventos `sensor.fraud.*` con score/acción | Trazabilidad |
| Modelo | Retrain periódico con datos limpios | Precisión |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Bots no detectados por falta de señales de sensores. |
| Opciones evaluadas | Sin sensores; colecta básica; features locales + scoring backend con privacidad/sampling. |
| Decisión | Colecta mínima con features locales, scoring backend y privacidad/sampling. |
| Consecuencias | Requiere modelo y gobernanza de datos; tuning de batería/privacidad. |
| Riesgos aceptados | Falsos positivos iniciales; impacto de batería si mal configurado. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Detección de bots | ↑ detección con baja tasa FP | Warning si FP sube | Seguridad |
| Consentimiento opt-in | 100% donde aplique | Alerta si falta | Cumplimiento |
| Consumo batería | Dentro de budget | Alerta si sube | UX |
| Incidentes de fraude | ↓ vs baseline | Crítico si no baja | Protección |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-behavioral-biometrics"></a>Behavioral biometrics | Identificación por patrones de uso/movimiento. |
| <a id="term-feature"></a>Feature | Métrica derivada de datos brutos de sensores. |
| <a id="term-scoring"></a>Scoring | Puntaje de riesgo basado en features. |
| <a id="term-sampling"></a>Sampling | Captura de datos a intervalos controlados. |
| <a id="term-anonimizacion"></a>Anonimización | Remover PII de datos antes de enviarlos. |

---

## Referencias

- [Mobile Sensor Fraud Detection Research](https://scholar.google.com/)
- [Behavioral Biometrics Overview](https://www.nist.gov/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
