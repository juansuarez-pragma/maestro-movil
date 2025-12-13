# Caso 88: Geofencing Inteligente
## Bloquear Transacciones Fuera de Zona

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | geofencing, ubicación, antifraude, transacciones |
| **Patrón Técnico** | [Geofence](#term-geofence "Área geográfica definida para reglas de entrada/salida.") Rules, Risk Scoring, Location Verification |
| **Stack Seleccionado** | Flutter + plugins de ubicación + backend de reglas + Riverpod estado de riesgo |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Sin ubicación ni reglas, transacciones inusuales pasan sin control.
- Geofencing on-device sin backend genera falsos positivos y poca trazabilidad.
- Sin consentimiento/privacidad, el uso de ubicación puede incumplir normativas.

### Escenario de Negocio

> *"Como usuario, quiero que bloqueen transacciones fuera de mi zona habitual para evitar fraude."*

### Incidentes reportados
- **Bancos/fintech:** Ubicaciones atípicas elevan probabilidad de fraude.
- **Falsos positivos:** Ubicación imprecisa bloqueó transacciones legítimas.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Fraude por ubicación | Global | Ubicaciones inusuales son señal clave; se combinan con device/IP. |
| Geofencing móvil | Global | Precisión variable; se requieren múltiples señales y backend. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; manejo de datos/ubicación es brecha común. |

**Resumen global**
- Ubicación como señal de riesgo requiere reglas en backend, consentimientos y step-up; on-device simple es insuficiente y ruidoso.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Seguridad** | Fraude por uso en otras regiones/países |
| **UX** | Bloqueos falsos si ubicación es imprecisa |
| **Legal** | Regulaciones de geolocalización y privacidad |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Sin ubicación ni reglas | **INADECUADO:** No detecta anomalías. |
| **ACEPTABLE** | Geofencing simple on-device | **MEJORA:** Mejor, pero sin correlación con backend ni señales adicionales. |
| **ENTERPRISE** | **Geofencing con riesgo:** reglas en backend, señal de ubicación + device binding, step-up/bloqueo, privacidad y consentimientos | **ÓPTIMO:** Seguridad con control y menos falsos positivos. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Obtener ubicación con consentimiento. Enviar hash/zonas al backend para scoring. Aplicar step-up (OTP/biometría) o bloqueo según riesgo. Manejar precisión y tiempo de captura. |
| **Restricciones Duras (NO permite)** | **Privacidad:** Requiere consentimientos y minimización. **Precisión variable:** GPS puede fallar en interiores; usar múltiples señales. **Batería:** Ubicación frecuente consume energía; usar triggers. |
| **Criterio de Selección** | Reglas en backend; ubicación como señal adicional; consentimiento y minimización; step-up en riesgo. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Reglas | Scoring y decisiones (step-up/bloqueo) correctas | Backend/QA |
| Precisión | Manejo de señales múltiples y thresholds | Móvil/QA |
| Privacidad | [Consentimiento](#term-consentimiento "Aprobación del usuario para usar ubicación.") y minimización de datos | Seguridad/Legal |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Consentimiento | Claro y revocable; mostrar uso de ubicación | Transparencia |
| [Step-up](#term-step-up "Solicitar factor adicional ante riesgo.") | Solicitar OTP/biometría según riesgo | UX/Seguridad |
| Mensajes | Explicar bloqueos y cómo resolver | Soporte |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Frecuencia | Triggers de ubicación para ahorrar batería | Eficiencia |
| Datos | Hash/zonas en vez de coordenadas exactas | Privacidad |
| Observabilidad | Eventos `geo.risk.*` con decisión | Trazabilidad |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Transacciones fuera de zona sin control de riesgo. |
| Opciones evaluadas | Sin ubicación; geofencing on-device simple; reglas en backend con scoring/step-up y privacidad. |
| Decisión | Geofencing con reglas backend, señal de ubicación + device binding, step-up/bloqueo y privacidad. |
| Consecuencias | Requiere consentimientos y tuning de precisión; coordinación backend-móvil. |
| Riesgos aceptados | Falsos positivos si señal es mala; consumo de batería si se abusa. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Fraude por ubicación | ↓ vs baseline | Crítico si no baja | Seguridad |
| Falsos positivos | Controlados | Warning si suben | UX |
| Consentimientos activos | 100% usuarios con geofence | Alerta si faltan | Cumplimiento |
| Consumo de batería | Dentro de budget | Alerta si sube | Experiencia |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-geofence"></a>Geofence | Área geográfica definida para reglas de entrada/salida. |
| <a id="term-step-up"></a>Step-up | Solicitar factor adicional ante riesgo. |
| <a id="term-risk-scoring"></a>Risk scoring | Puntaje de riesgo basado en señales (ubicación, device, patrón). |
| <a id="term-consentimiento"></a>Consentimiento | Aprobación del usuario para usar ubicación. |
| <a id="term-minimizacion-de-datos"></a>Minimización de datos | Guardar solo lo necesario, posiblemente hash/zonas. |

---

## Referencias

- [Android Geofencing](https://developer.android.com/training/location/geofencing)
- [iOS Region Monitoring](https://developer.apple.com/documentation/corelocation/monitoring_the_user_s_proximity_to_geographic_regions)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
