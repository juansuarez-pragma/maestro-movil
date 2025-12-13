# Caso 86: Voice Biometrics
## "Mi Voz es mi Contraseña" para Banca Telefónica

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | biometría de voz, autenticación, liveness, fraude |
| **Patrón Técnico** | Voiceprint Enrollment/Verification, Anti-Spoofing, Challenge Phrases |
| **Stack Seleccionado** | Flutter + Platform Channels a SDK de voz + mic permissions + cifrado de templates |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Voiceprint estático sin liveness permite replay/deepfake.
- Frase fija puede ser grabada y reutilizada.
- Sin consentimiento y cifrado, los datos biométricos violan privacidad/regulación.

### Escenario de Negocio

> *"Como usuario, quiero autenticarme por voz en canales telefónicos sin ser suplantado."*

### Incidentes reportados
- **Fraude de voz:** Deepfakes y grabaciones han burlado autenticaciones estáticas.
- **Auditorías:** Falta de consentimiento/borrado de biometría generó hallazgos.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Contact centers | Global | Biometría de voz con anti-spoofing y challenge dinámico adopción creciente. |
| ASVspoof benchmarks | Global | Deepfakes requieren detección avanzada de spoof. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; manejo de datos sensibles es brecha común. |

**Resumen global**
- Biometría de voz segura requiere anti-spoof, challenge dinámico y templates cifrados; enfoques estáticos son vulnerables a replay/deepfake.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Seguridad** | Suplantación con grabaciones/deepfakes |
| **Regulatorio** | Consentimiento y protección de datos biométricos |
| **Técnico/UX** | Falsos rechazos en ambientes ruidosos |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Voiceprint estático sin liveness | **INADECUADO:** Vulnerable a replay. |
| **ACEPTABLE** | Liveness básico y frase fija | **MEJORA:** Más seguro, pero susceptible a grabaciones de esa frase. |
| **ENTERPRISE** | **Anti-spoof + frases dinámicas:** challenge aleatorio, detección de spoof, templates cifrados, umbrales ajustables | **ÓPTIMO:** Seguridad alta y adaptable. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Enrollment con consentimiento. Verificación con challenge aleatorio. Detección de spoof (replay/deepfake). Templates cifrados. Ajuste de umbrales y re-entrenamiento. |
| **Restricciones Duras (NO permite)** | **Ruido:** Afecta precisión. **Privacidad:** Datos biométricos regulados. **Hardware:** Calidad de micrófono impacta FAR/FRR. |
| **Criterio de Selección** | SDK con anti-spoof, challenge dinámico, templates cifrados; políticas de consentimiento/borrado; métricas FAR/FRR. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Anti-spoof | Bloqueo de replay/deepfake | Seguridad/QA |
| UX/ruido | Tasa de rechazo/aceptación en ambientes ruidosos | QA/Móvil |
| Privacidad | Consentimiento y cifrado de templates | Legal/Seguridad |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Frases | Challenge aleatorio; evitar frases fijas | Seguridad |
| Feedback | Mensajes claros en rechazo y reintentos limitados | UX |
| Consentimiento | Opt-in y borrado a solicitud | Cumplimiento |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Almacenamiento | Templates cifrados, control de acceso | Seguridad |
| Métricas | FAR/FRR monitorizados; tuning continuo | Calidad |
| Observabilidad | Eventos `voice.auth.*` con resultado/razón | Trazabilidad |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Suplantación por replay/deepfake en biometría de voz. |
| Opciones evaluadas | Voiceprint estático; liveness básico; anti-spoof + challenge dinámico + templates cifrados. |
| Decisión | Anti-spoof, challenge dinámico, templates cifrados y políticas de consentimiento. |
| Consecuencias | Requiere UX clara y pruebas en ambientes ruidosos; gestión de datos biométricos. |
| Riesgos aceptados | Falsos rechazos en ruido; dependencia de calidad de micrófono. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| FAR (False Accept Rate) | Bajo y monitoreado | Crítico si sube | Seguridad |
| FRR (False Reject Rate) | Controlado | Warning si sube | UX |
| Incidentes de spoof | 0 | Crítico si >0 | Protección |
| Consentimientos registrados | 100% enrolados | Alerta si faltan | Cumplimiento |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Voiceprint | Plantilla biométrica de voz. |
| FAR/FRR | False Accept/Reject Rate. |
| Challenge aleatorio | Frase/código distinto en cada verificación. |
| Anti-spoofing | Técnicas para detectar audio falso/reproducido. |
| Template cifrado | Voiceprint almacenada con cifrado fuerte. |

---

## Referencias

- [Voice Biometrics Standards](https://pages.nist.gov)
- [Anti-spoofing Research](https://www.asvspoof.org/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
