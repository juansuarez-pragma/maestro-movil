# Caso 88: Geofencing Inteligente
## Bloquear Transacciones Fuera de Zona

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | geofencing, ubicación, antifraude, transacciones |
| **Patrón Técnico** | Geofence Rules, Risk Scoring, Location Verification |
| **Stack Seleccionado** | Flutter + plugins de ubicación + backend de reglas + Riverpod estado de riesgo |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero que bloqueen transacciones fuera de mi zona habitual para evitar fraude."*

Sin geofencing, transacciones en ubicaciones inusuales pueden pasar sin control.

### Evidencia de Industria

- **Bancos/fintech:** Usan ubicación como señal de riesgo adicional.
- **Fraude:** Transacciones desde ubicaciones atípicas elevan probabilidad de fraude.

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
| **ENTERPRISE** | **Geofencing con riesgo:** reglas en backend, señal de ubicación + device binding, step-up/ bloqueo, privacidad y consentimientos | **ÓPTIMO:** Seguridad con control y menos falsos positivos. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Obtener ubicación con consentimiento. Enviar hash/zonas al backend para scoring. Aplicar step-up (OTP/biometría) o bloqueo según riesgo. Manejar precisión y tiempo de captura. |
| **Restricciones Duras (NO permite)** | **Privacidad:** Requiere consentimientos y minimización. **Precisión variable:** GPS puede ser impreciso en interiores; usar múltiples señales. **Batería:** Ubicación frecuente consume energía; usar triggers. |
| **Criterio de Selección** | Reglas en backend; uso de ubicación como señal adicional; consentimiento y minimización de datos; step-up en casos de riesgo. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Geofence | Área geográfica definida para reglas de entrada/salida. |
| Step-up | Solicitar factor adicional ante riesgo. |
| Risk scoring | Puntaje de riesgo basado en señales (ubicación, device, patrón). |
| Consentimiento | Aprobación del usuario para usar ubicación. |
| Minimización de datos | Guardar solo lo necesario, posiblemente hash/zonas. |

---

## Referencias

- [Android Geofencing](https://developer.android.com/training/location/geofencing)
- [iOS Region Monitoring](https://developer.apple.com/documentation/corelocation/monitoring_the_user_s_proximity_to_geographic_regions)
