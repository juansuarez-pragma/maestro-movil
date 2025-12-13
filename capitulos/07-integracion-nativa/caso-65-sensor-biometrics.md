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

### Problema detectado (técnico)
- `local_auth` no cubre liveness avanzado ni SDKs certificados; riesgo de bypass.
- Integraciones sin callbacks tipados rompen flujos (cámara/micrófono) y causan crashes.
- Sin manejo de permisos y performance, la UX se degrada y el SDK pierde precisión.

### Escenario de Negocio

> *"Como app, necesito combinar biometrías avanzadas (face/voice) con SDKs certificados sin romper Flutter."*

### Incidentes reportados
- **SDKs certificados (iBeta/NIST):** Integraciones incompletas fallaron en pruebas de liveness.
- **Apps financieras:** Sin step-up adecuado, se aprobaron usuarios fraudulentos o se bloquearon legítimos.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| iBeta/NIST PAD | Global | Exigen liveness robusto; integraciones deficientes fallan auditoría. |
| Banca/fintech | Global | Uso de biometrías múltiples para riesgo; necesitan flujos nativos completos. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; biometría mal integrada es hallazgo común. |

**Resumen global**
- Biometría avanzada requiere wrappers nativos tipados, manejo de permisos/performance y pruebas por plataforma; soluciones genéricas dejan huecos de seguridad y UX.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Integration (Android/iOS) | Flujos de face/voice con callbacks tipados y permisos | Móvil/QA |
| Seguridad | Liveness y resistencia a spoof | Seguridad |
| Performance | Latencia y uso de CPU/GPU en dispositivos target | QA/Perf |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Guías UI | Instrucciones claras para cámara/micrófono | Reduce abandono |
| Fallback | Step-up alternativo si sensor falla | Disponibilidad |
| Permisos | Solicitar en contexto y justificar | Confianza |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Licencias | Validar licencias por SDK y rotación | Cumplimiento |
| Telemetría | Eventos `bio.*` con factor/resultado/latencia | Observabilidad |
| Compatibilidad | Matriz de dispositivos soportados y exclusiones | Previene crashes |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Biometría avanzada sin liveness robusto ni estabilidad en Flutter. |
| Opciones evaluadas | `local_auth`; wrapper básico; wrapper tipado con liveness, permisos y pruebas por plataforma. |
| Decisión | Wrapper robusto por factor con canales tipados, liveness y telemetría. |
| Consecuencias | Mayor complejidad nativa y pruebas específicas; manejo de licencias. |
| Riesgos aceptados | Dependencia de SDKs y restricciones de dispositivos. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Tasa de éxito biométrico | > 98% en dispositivos soportados | Warning si baja | UX |
| Falsos positivos/negativos | Bajo según SDK | Crítico si sube | Seguridad |
| Crash rate en flujos bio | Tendencia a la baja | Crítico si sube | Estabilidad |
| Abandono en flujo bio | ↓ vs baseline | Warning si no baja | Conversión |

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
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
