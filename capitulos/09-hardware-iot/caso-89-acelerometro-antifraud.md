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

### Escenario de Negocio

> *"Como app, quiero detectar bots o automatizaciones usando patrones de movimiento/dispositivo."*

Bots pueden emular taps, pero carecen de micro-movimientos humanos capturables por sensores.

### Evidencia de Industria

- **Behavioral biometrics:** Usan acelerómetro/giroscopio para detectar bots.
- **Fraude móvil:** Herramientas anti-bot incorporan señales de sensores.

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
| **Restricciones Duras (NO permite)** | **Privacidad:** Requiere consentimiento y minimización. **Costo battery:** Sampling debe ser limitado. **Modelos:** Necesita entrenamiento y actualización; riesgo de falsos positivos. |
| **Criterio de Selección** | Recolección mínima necesaria, features calculadas localmente, consentimientos claros, sampling y límites de battery, backend de scoring. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Behavioral biometrics | Identificación por patrones de uso/movimiento. |
| Feature | Métrica derivada de datos brutos de sensores. |
| Scoring | Puntaje de riesgo basado en features. |
| Sampling | Captura de datos a intervalos controlados. |
| Anonimización | Remover PII de datos antes de enviarlos. |

---

## Referencias

- [Mobile Sensor Fraud Detection Research](https://scholar.google.com/)
- [Behavioral Biometrics Overview](https://www.nist.gov/)
