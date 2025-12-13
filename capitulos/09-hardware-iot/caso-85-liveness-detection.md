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

### Problema detectado (técnico)
- Selfie sin liveness permite ataques de foto/video/deepfake.
- Liveness básico 2D no detiene máscaras o deepfakes avanzados.
- Sin evidencia firmada y manejo de licencias, se compromete auditoría y cumplimiento.

### Escenario de Negocio

> *"Como app financiera, debo asegurar que es una persona viva y no una foto/video."*

### Incidentes reportados
- **Fraude con deepfakes:** Banca ha sufrido suplantaciones por liveness débil.
- **KYC/AML:** Auditorías fallaron por falta de evidencias firmadas.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| ISO 30107-3 / iBeta | Global | SDKs certificados bloquean spoofing 2D/3D. |
| Casos de fraude | Global | Deepfakes y máscaras eluden liveness básico. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; biometría mal integrada es frecuente. |

**Resumen global**
- PAD certificado con challenge-response y evidencia firmada reduce fraude; liveness básico deja vectores abiertos a deepfakes y spoofing.

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
| **Restricciones Duras (NO permite)** | **Condiciones de luz:** Afectan precisión. **Dispositivos sin buenas cámaras:** Más falsos positivos. **Licencias:** SDKs certificados requieren licenciamiento. |
| **Criterio de Selección** | SDK certificado, wrapper nativo robusto, manejo de permisos cámara, mensajes claros al usuario, reintentos limitados. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Seguridad | PAD bloquea fotos/videos/deepfakes | Seguridad/QA |
| Evidencia | Payload firmado y cifrado | Seguridad |
| UX | Mensajes claros y reintentos limitados | Móvil/QA |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Iluminación | Guiar al usuario y validar condiciones | Reduce falsos positivos |
| Reintentos | Límite y cooldown | Evita abuso |
| Soporte | Mensajes de error con causa | Transparencia |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Licencias | Gestión y expiración del SDK | Cumplimiento |
| Evidencias | Almacenamiento seguro y cifrado | Auditoría |
| Observabilidad | Eventos `liveness.*` con resultado/razón | Monitoreo |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Spoofing (foto/video/deepfake) en verificación facial. |
| Opciones evaluadas | Selfie simple; liveness básico 2D; PAD certificado con challenge-response y evidencia firmada. |
| Decisión | PAD certificado (iBeta/NIST) con challenge-response, evidencia firmada y UX guiada. |
| Consecuencias | Mayor costo/licencias; requiere UX y pruebas en dispositivos variados. |
| Riesgos aceptados | Condiciones de luz/dispositivos limitan precisión. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Spoofings detectados | 100% | Crítico si se escapa alguno | Seguridad |
| Falsos positivos | Mantener bajo | Warning si sube | UX |
| Tiempo de verificación | Estable | Alerta si crece | Experiencia |
| Incidentes de auditoría | 0 | Crítico si >0 | Cumplimiento |

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
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
