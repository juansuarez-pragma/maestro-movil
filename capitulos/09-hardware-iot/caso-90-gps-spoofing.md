# Caso 90: GPS Spoofing y Mock Locations
## Evitar Fraude por Ubicación Falsa en Geofencing

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | GPS spoofing, mock location, antifraude, ubicación, geofencing |
| **Patrón Técnico** | Location Integrity, Signal Fusion, Risk-Based Step-up |
| **Stack Seleccionado** | Flutter + plugins de ubicación + backend de scoring + Play Integrity/App Attest (cuando aplique) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Geofencing basado solo en GPS puede ser burlado con **mock locations** o apps de spoofing.
- Sin señales adicionales (WiFi/celda/tiempo), el riesgo de falsos positivos y bypass aumenta.
- Si se bloquea sin alternativa, se afecta UX y se incrementa soporte.

### Escenario de Negocio

> *"Como banco, quiero usar ubicación como señal antifraude, pero evitando que un atacante falsifique el GPS para aprobar operaciones."*

### Incidentes reportados
- **Spoofing en fraude:** atacantes simulan estar “en zona” para evadir reglas.
- **Bloqueos falsos:** GPS impreciso en interiores disparó bloqueos a usuarios legítimos.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Android Location | Global | Existen flags y señales para detectar ubicaciones simuladas (según API/OS). |
| Prácticas antifraude | Global | “Signal fusion” (GPS + red + device integrity) reduce spoofing. |
| OWASP MAS | Global | Recomienda validar integridad del dispositivo y no confiar en una sola señal. |

**Resumen global**
- Ubicación debe tratarse como una señal probabilística: combinar múltiples señales y usar step-up/score, no reglas rígidas solo con GPS.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Seguridad** | Bypass de controles por GPS falso |
| **UX** | Bloqueos falsos por precisión variable |
| **Legal/Privacidad** | Uso de ubicación requiere consentimiento y minimización |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Geofence basado solo en GPS | **INADECUADO:** Vulnerable a spoofing y ruido. |
| **ACEPTABLE** | GPS + detección básica de mock location | **MEJORA:** Reduce algunos casos, pero no detiene spoofing avanzado. |
| **ENTERPRISE** | **Ubicación con integridad:** fusión de señales (GPS+WiFi+celda+tiempo), scoring backend, device integrity/attestation, step-up y auditoría | **ÓPTIMO:** Menos bypass y menos falsos positivos. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Detectar señales de ubicación simulada donde aplique. Enviar features de precisión/edad/consistencia al backend. Usar step-up (OTP/biometría) cuando el score lo requiera. |
| **Restricciones Duras (NO permite)** | **No existe señal perfecta:** spoofing avanzado puede evadir checks. **Privacidad:** minimización/consentimiento obligatorio. **Batería:** lecturas frecuentes consumen energía. |
| **Criterio de Selección** | Señales múltiples + scoring + step-up; evitar bloqueos rígidos; registrar auditoría sin PII en claro. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Seguridad | Resistencia a mock locations y bypass básicos | Seguridad/QA |
| Precisión | Control de falsos positivos (interiores/urban) | QA/Perf |
| Privacidad | Consentimiento y minimización de datos | Seguridad/Legal |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Step-up | Solicitar factor adicional ante sospecha, no bloquear siempre | UX/Seguridad |
| Mensajes | Explicar bloqueo/step-up sin revelar señales | Anti-abuso |
| Fallback | Alternativas si ubicación no está disponible | Inclusión |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Sampling | Captura solo en operaciones críticas | Batería |
| Observabilidad | Eventos `geo.integrity.*` con score/decisión | Trazabilidad |
| Gobernanza | Ajuste de thresholds basado en métricas reales | Control |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Ubicación falsificable afecta controles antifraude. |
| Opciones evaluadas | GPS-only; GPS + check básico; fusión de señales + scoring + step-up + integridad de dispositivo. |
| Decisión | Fusión de señales con scoring backend y step-up; controles de integridad donde aplique. |
| Consecuencias | Más complejidad y tuning; requiere métricas. |
| Riesgos aceptados | Spoofing avanzado; precisión variable en escenarios reales. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Bypass por GPS falso | ↓ vs baseline | Crítico si no baja | Seguridad |
| Falsos positivos | Controlados | Warning si suben | UX |
| Step-up triggers | Medidos y calibrados | Alerta si explotan | Operación |
| Tickets por geofence | Tendencia a la baja | Alerta si suben | Soporte |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Mock location | Ubicación simulada por el OS/app para pruebas o spoofing. |
| Spoofing | Suplantación de una señal (ej. GPS). |
| Signal fusion | Combinar señales para aumentar confiabilidad. |
| Scoring | Puntaje de riesgo basado en señales. |
| Step-up | Solicitar un factor adicional ante riesgo. |

---

## Referencias

- [Android Location (overview)](https://developer.android.com/develop/sensors-and-location/location)
- [OWASP MAS](https://mas.owasp.org/)
