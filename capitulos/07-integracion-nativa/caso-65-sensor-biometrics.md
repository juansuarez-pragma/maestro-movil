# Caso 65: Sensores Biométricos Avanzados
## Uso de Face/Touch/Voice con SDKs Nativos

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | biometría avanzada, sensores, sdk nativo, voz |
| **Patrón Técnico** | Multi-factor Biometrics, Native SDK Integration, Liveness |
| **Stack Seleccionado** | Flutter + Platform Channels + SDKs nativos biométricos (Face/Voice) + Riverpod estado |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como app, necesito combinar biometrías avanzadas (face/voice) con SDKs certificados sin romper Flutter."*

Biometrías avanzadas requieren flujos nativos, liveness y callbacks complejos.

### Evidencia de Industria

- **SDKs certificados (iBeta/NIST):** Necesitan integración nativa controlada.
- **Apps financieras:** Usan biometrías múltiples para step-up de riesgo.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Seguridad** | Implementación incorrecta = bypass biométrico |
| **UX** | Flujos frágiles y crashes por mala integración |
| **Legal** | Incumplimiento si SDK no se integra según requisitos |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Usar `local_auth` para todo | **INADECUADO:** No cubre liveness avanzado ni SDKs certificados. |
| **ACEPTABLE** | Integrar SDK nativo básico sin callbacks tipados | **MEJORA:** Pero frágil y difícil de testear. |
| **ENTERPRISE** | **Wrapper robusto por factor:** canales tipados, mapeo de errores, liveness, evidencia auditada, pruebas por plataforma | **ÓPTIMO:** Seguridad y estabilidad, cumplimiento de certificación. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Encapsular flujos de face/voice/fingerprint avanzados. Recepción de callbacks/eventos con estados. Recolección de evidencia (FaceMap/voz) según SDK. Manejo de permisos cámara/micrófono. |
| **Restricciones Duras (NO permite)** | **Dependencia del SDK:** APIs y licencias específicas. **Performance:** Procesamiento pesado puede requerir optimizaciones. **Compatibilidad:** No todos los dispositivos soportan sensores avanzados. |
| **Criterio de Selección** | Wrapper por factor, contratos claros, pruebas E2E nativas, manejo de errores y permisos explícitos. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Liveness | Verificación de que hay una persona viva, no spoof. |
| FaceMap | Representación biométrica generada por SDKs de rostro. |
| Callback tipado | Evento con estructura definida para resultados/errores. |
| Step-up | Aumentar el factor de autenticación según riesgo. |
| SDK certificado | SDK con validación/regulación (iBeta/NIST). |

---

## Referencias

- [local_auth](https://pub.dev/packages/local_auth)
- [iBeta PAD](https://www.ibeta.com/pad-testing/)
