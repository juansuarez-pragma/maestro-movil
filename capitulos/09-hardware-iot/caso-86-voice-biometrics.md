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

### Escenario de Negocio

> *"Como usuario, quiero autenticarme por voz en canales telefónicos sin ser suplantado."*

Biometría de voz es vulnerable a replay y deepfakes si no se implementa con liveness y desafíos dinámicos.

### Evidencia de Industria

- **Contact centers:** Adoptan biometría de voz con detección de spoofing.
- **Fraudes de voz:** Deepfakes de voz aumentan; requieren contramedidas.

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
| **Capacidades (SÍ permite)** | Enrollment de voiceprint con consentimiento. Verificación con challenge aleatorio. Detección de spoof (replay/deepfake). Almacenamiento cifrado de templates. Ajuste de umbrales y re-entrenamiento. |
| **Restricciones Duras (NO permite)** | **Ruido ambiental:** Afecta precisión. **Consentimiento/privacidad:** Datos biométricos requieren cumplimiento legal. **Compatibilidad:** Depende de calidad de micrófono. |
| **Criterio de Selección** | SDK con anti-spoof; challenge dinámico; templates cifrados; políticas de consentimiento y borrado; métricas de FAR/FRR. |

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
