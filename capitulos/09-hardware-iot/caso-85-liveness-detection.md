# Caso 85: Liveness Detection
## Evitar Fotos de Fotos en Verificación Facial

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | liveness, spoofing, biometría facial, anti-fraude |
| **Patrón Técnico** | Presentation Attack Detection (PAD), Challenge-Response, Anti-Spoofing |
| **Stack Seleccionado** | Flutter + SDK nativo de liveness (iBeta/NIST) via Platform Channels + Riverpod estado |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como app financiera, debo asegurar que es una persona viva y no una foto/video."*

Sin PAD, los ataques de foto o video pueden pasar como legítimos.

### Evidencia de Industria

- **ISO 30107-3:** Estándar para PAD. SDKs certificados mitigan spoofing.
- **Incidentes de deepfake:** Casos reales en banca con fraude por liveness débil.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Seguridad/Legal** | Fraude de identidad, incumplimiento KYC/AML |
| **UX** | Falsos positivos si liveness falla con malas condiciones |
| **Reputacional** | Pérdida de confianza tras incidentes de spoofing |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Selfie sin liveness, solo match | **INADECUADO:** Vulnerable a fotos/videos. |
| **ACEPTABLE** | Liveness básico 2D | **MEJORA:** Bloquea fotos, pero no deepfakes/máscaras. |
| **ENTERPRISE** | **PAD certificado:** challenge-response, detección 2D/3D, evidencias firmadas, SDK iBeta/NIST | **ÓPTIMO:** Alta seguridad y auditoría. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Challenge (movimientos/expresiones). Detección de spoof 2D/3D. Evidencia encriptada y firmada. Callback con resultados y razones. |
| **Restricciones Duras (NO permite)** | **Condiciones de luz:** Afectan precisión. **Dispositivos sin buenas cámaras:** Mayor falsos positivos. **Licencias:** SDKs certificados requieren licenciamiento. |
| **Criterio de Selección** | SDK certificado, wrapper nativo robusto, manejo de permisos cámara, mensajes claros al usuario, reintentos limitados. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| PAD | Presentation Attack Detection. |
| Deepfake | Video/imagen generada para suplantación. |
| Challenge-Response | Solicitar acciones para probar liveness. |
| Evidencia firmada | Resultado con firma para auditoría. |
| Falsos positivos/negativos | Errores de detección. |

---

## Referencias

- [ISO 30107-3](https://www.iso.org/standard/67381.html)
- [iBeta PAD Testing](https://www.ibeta.com/pad-testing/)
