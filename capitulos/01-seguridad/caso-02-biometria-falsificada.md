# Caso 2: Biometría Falsificada
## Cuando Face ID No Es Suficiente para Aprobar un Crédito

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | autenticación biométrica, Face ID, Touch ID, aprobación de crédito, fraude de identidad, liveness detection |
| **Patrón Técnico** | Multi-Layer Biometric Authentication, [Step-up Authentication](#term-step-up "Escalar factores (OTP/PIN/biometría avanzada) según riesgo de la operación."), Liveness Detection |
| **Stack Seleccionado** | local_auth + platform channels para SDK nativo (Facetec/iProov) + Cubit (BiometricCubit) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Biometría del sistema ([local_auth](#term-local_auth)) solo valida que el rostro/huella está inscrito en el dispositivo, no la identidad real del titular ni liveness.
- Sin PAD ([Liveness Detection](#term-liveness-detection)) robusto, fotos/videos/deepfakes pasan como válidos; riesgo alto de fraude en aprobaciones de crédito.

### Escenario de negocio
> *"Como usuario, quiero aprobar mi solicitud de crédito de $50,000 usando Face ID para no tener que ir a una sucursal."*

**Incidente reportado (ejemplo realista):**
- Solicitud de crédito aprobada con deepfake (foto de foto); el sistema aceptó el Face ID del dispositivo.
- Monto fraudulento: > $50K; reclamaciones y revisión manual costosa.
- Regulación PSD2/SCA exige SCA con biometría anti-spoofing; el flujo actual no cumple.

### Incidentes reportados

- **Deepfake Banking Fraud (2023):** grupo criminal en Hong Kong usó deepfakes para engañar verificación facial en varios bancos, obteniendo > $25M en préstamos.
- **Presentation attacks (2021):** 20% de sistemas de reconocimiento facial en apps bancarias europeas fueron engañados con fotos impresas.
- **PSD2/SCA:** exige liveness y anti-spoofing; no cumplir expone a multas KYC/AML y denegación de SCA.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| NIST FRVT PAD (2022) | Proveedores globales | Sistemas sin PAD robusto mostraron tasas de aceptación de ataque (PAI) > 10%; top proveedores con PAD certificada < 0.1% FAR. |
| iBeta PAD listings (2023) | Vendors certificados iBeta | Solo SDKs con certificación ISO 30107-3 mantienen FAR < 0.2% en pruebas de máscaras/fotos. |
| NowSecure State of Mobile App Security (2024) | 1,000+ apps móviles (US/EU) | Biometría sin liveness aparece en el top de fallos MASVS; 85% de apps fallan ≥1 control MASVS. |
| Kaspersky Mobile Threats (2023) | LATAM y APAC | Incremento de malware que abusa accesibilidad/cámara para capturar video y suplantar biometría. |

**Resumen global de hallazgos**
- Entre 10% y 20% de implementaciones sin PAD son vulnerables a fotos/máscaras (NIST FRVT PAD 2022).
- Solo SDKs con certificación iBeta ISO 30107-3 alcanzan FAR < 0.2% frente a ataques de presentación.
- 85% de apps móviles fallan ≥1 control MASVS; biometría sin liveness es un fallo recurrente (NowSecure 2024).
- LATAM/APAC presentan mayor actividad de malware que captura imágenes y abusa accesibilidad (Kaspersky 2023).

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Pérdidas por fraude en préstamos ($50K-500K por incidente) |
| **Regulatorio** | Incumplimiento KYC/AML: multas de $1M-100M |
| **Reputacional** | Un caso viral de deepfake fraud destruye confianza digital |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | `local_auth` para Face ID/Touch ID, si pasa → aprobado | **INADECUADO:** Solo verifica biometría del dispositivo. No verifica que sea el titular. No detecta fotos/videos. Cualquier persona con acceso al dispositivo puede aprobar. |
| **ACEPTABLE** | `local_auth` como primer factor + captura de selfie + envío a backend para comparación con foto de documento | **CUMPLE MÍNIMOS:** Agrega verificación de identidad. Pero: vulnerable a fotos impresas, latencia alta, dependencia de conectividad. |
| **ENTERPRISE** | **Autenticación en capas:** 1) `local_auth` gate inicial, 2) SDK certificado Liveness (Facetec/iProov) via Platform Channels, 3) Comparación 1:1 contra KYC, 4) Challenge-Response, 5) Cubit para orquestar | **ÓPTIMO PARA BANCA:** Detecta spoofing 2D/3D. Certificación iBeta/NIST. Evidencia auditable con firma criptográfica. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Cubit de biometría transita estados secuenciales y maneja errores | Equipo móvil, CI |
| Integration (CI) | SDK de liveness responde spoof → app rechaza y no aprueba; fallback a PIN/OTP cuando biometría falla/enroll no disponible | Equipo móvil/backend, CI |
| Seguridad manual | Ataque de foto impresa/video grabado debe ser rechazado; permisos de cámara manejados correctamente | Seguridad/QE, dispositivos reales |
| Observabilidad (CI) | Eventos `biometric.*` con motivo de fallo (low_light, spoof_detected, user_abort) y métricas de score/liveness | Equipo móvil, CI |

### 3.2 Capacidades, límites y restricciones — tabla
| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Detección de ataques 2D (fotos, pantallas) y 3D (máscaras). Verificación de liveness con challenge-response. Comparación [1:1](#term-face-match) selfie vs documento. FaceMap encriptado para auditoría. Funciona con cámara frontal estándar. Cumple ISO 30107-3 y NIST FRVT. |
| **Restricciones Duras (NO permite)** | **local_auth SOLO:** No distingue entre titular y otra persona del dispositivo. **Sin red:** Algunos SDKs requieren validación server-side. **Condiciones extremas:** Liveness falla con luz < 50 lux, lentes muy oscuros, barbas > 50% rostro. **Simulador:** Biometría no disponible. |
| **Criterio de Selección** | Se usa **[Cubit](#term-cubit)** en lugar de BLoC porque el flujo es secuencial sin eventos complejos (captura→análisis→resultado). Cubit reduce boilerplate. Se usan **[Platform Channels](#term-platform-channels)** para SDK nativo porque Facetec/iProov solo existen en Kotlin/Swift, procesamiento de imagen más eficiente nativo. |

### 3.3 Privacidad y custodia de datos — tabla sintética
| Tema | Requisito | Notas |
|:-----|:----------|:------|
| Almacenamiento | No persistir biometría en app; FaceMap/en evidencias solo cifradas y efímeras | Uso exclusivo para verificación; limpiar tras decisión |
| Transmisión | Encriptar evidencias (TLS/mTLS) y firmar payloads | Validar integridad en backend |
| Retención | Mínima necesaria para auditoría; cumplir regulaciones de datos biométricos | Borrado/expurgo programado |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Biometría del dispositivo sin liveness no valida titular ni detecta spoof (fotos/deepfakes). |
| Opciones evaluadas | Solo `local_auth`; `local_auth` + selfie backend; SDK PAD certificado + comparación 1:1 + fallback. |
| Decisión | SDK con PAD certificado (ISO 30107-3) via Platform Channels + comparación 1:1 KYC + fallback seguro (PIN/OTP). |
| Consecuencias | Dependencia de SDK nativo y red; mayor latencia que `local_auth` solo; costo de licenciamiento. |
| Riesgos aceptados | Fallos en baja luz/oclusiones; dependencia de conectividad para validación server-side. |

### 3.5 UX, accesibilidad y condiciones — tabla
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Fallback | Si no hay biometría enrollada o error (`LAError/BIOMETRIC_ERROR`) → PIN/OTP con mensajes claros | Evita bloqueos y mantiene seguridad |
| Permisos de cámara | CTA a Settings si se deniega; explicar motivo y uso | Transparencia y reducción de abandono |
| Condiciones de captura | Luz > 50 lux, sin lentes oscuros, rostro estable; reintentos limitados | Mejora tasas de éxito y reduce falsos rechazos |
| Accesibilidad | Mensajes compatibles con lectores de pantalla; instrucciones concisas | Inclusión y reducción de fricción |

### 3.6 Contratos y evidencias
| Componente | Request/Evidencia | Validación | Nota |
|:-----------|:------------------|:-----------|:-----|
| Captura selfie liveness | Frame + metadata (luz, movimiento) | SDK PAD certificado iBeta/NIST | No persistir en app; envío cifrado |
| Comparación 1:1 | Selfie vs foto KYC | Score ≥ umbral; firma del proveedor | Loguear motivo de fallo (spoof, low_light) |
| Evento de resultado | `biometric.result` con score, liveness, motivo | Observabilidad en CI/producción | No registrar imágenes; solo metadatos |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Detección de spoof (PAD 2D/3D) | > 99% | FAR < 0.1% | Minimiza fraude por fotos/máscaras/deepfakes |
| Tiempo de verificación p95 | < 10 s | Alerta si supera meta | UX ágil; menos abandonos |
| Reintentos promedio | ≤ 1 | Alerta si > 1 | Reduce fricción y soporte |
| % sesiones aprobadas sin PAD | 0 | Alerta si > 0 | Elimina exposiciones por flujo débil |
| Incidentes/1M verificaciones (deepfake/spoof aceptado) | 0 | Cualquier > 0 es crítico | Tolerancia cero a ataques de presentación |
| Reducción de fraude en onboarding/crédito | ≥ 80% vs baseline | Alertar si no mejora tras rollout | Beneficio de negocio tangible |
| Tickets por fallas de biometría | ↓ 25% en 4 semanas | Alertar si no baja | Mejora NPS y costo operativo |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-local_auth"></a>local_auth | Plugin de biometría del sistema (Face ID/Touch ID/BiometricPrompt) sin verificación de identidad del titular. |
| <a id="term-liveness-detection"></a>Liveness Detection | Pruebas anti-spoofing (2D/3D) para asegurar que hay una persona viva frente a la cámara. |
| <a id="term-platform-channels"></a>Platform Channels | Puente Flutter ↔ nativo para integrar SDKs que no existen en Dart. |
| <a id="term-face-match"></a>Face Match 1:1 | Comparación de selfie vs foto de documento KYC con umbral de score mínimo. |
| <a id="term-iso-pad"></a>ISO 30107-3 / PAD | Estándar de pruebas de Presentation Attack Detection para biometría. |
| <a id="term-step-up"></a>Step-up Authentication | Escalar factores (OTP/PIN/biometría avanzada) según riesgo de la operación. |
| <a id="term-cubit"></a>Cubit | Patrón de estado ligero basado en flujos secuenciales sin eventos complejos. |

---

## Referencias

- [ISO/IEC 30107-3:2017 - Biometric presentation attack detection](https://www.iso.org/standard/67381.html)
- [NIST FRVT - Face Recognition Vendor Test](https://www.nist.gov/programs-projects/face-recognition-vendor-test-frvt)
- [iBeta Quality Assurance - PAD Testing](https://www.ibeta.com/pad-testing/)
- [Apple Local Authentication Framework](https://developer.apple.com/documentation/localauthentication)
- [Android BiometricPrompt](https://developer.android.com/reference/android/hardware/biometrics/BiometricPrompt)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
- [Kaspersky - Mobile Threats 2023](https://securelist.com/mobile-malware-evolution-2023/111742/)

---

*Anterior: [Token que Nunca Expira](caso-01-token-nunca-expira.md) | Siguiente: [Certificate Pinning](caso-03-mitm-certificate-pinning.md)*
