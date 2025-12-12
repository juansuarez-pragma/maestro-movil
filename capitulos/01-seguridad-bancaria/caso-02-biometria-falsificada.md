# Caso 2: Biometría Falsificada
## Cuando Face ID No Es Suficiente para Aprobar un Crédito

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | autenticación biométrica, Face ID, Touch ID, aprobación de crédito, fraude de identidad, liveness detection |
| **Patrón Técnico** | Multi-Layer Biometric Authentication, Step-up Authentication, Liveness Detection |
| **Stack Seleccionado** | local_auth + platform channels para SDK nativo (Facetec/iProov) + Cubit (BiometricCubit) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero aprobar mi solicitud de crédito de $50,000 usando Face ID para no tener que ir a una sucursal."*

El problema: Face ID/Touch ID del sistema operativo solo verifica que "un rostro/huella registrado en el dispositivo" coincide. NO verifica que sea el rostro del titular de la cuenta bancaria, ni que sea una persona viva frente a la cámara.

### Evidencia de Industria

**Caso Deepfake Banking Fraud (2023):** Un grupo criminal en Hong Kong usó deepfakes para engañar sistemas de verificación facial en múltiples bancos, obteniendo préstamos por más de $25 millones.

**Ataque de Presentation Attack (Spoofing):** En 2021, investigadores demostraron que el 20% de los sistemas de reconocimiento facial en apps bancarias europeas podían ser engañados con una foto impresa.

**Regulación PSD2/SCA:** La directiva requiere Strong Customer Authentication para transacciones > €30. La biometría DEBE incluir protección anti-spoofing según estándares EBA.

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

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Detección de ataques 2D (fotos, pantallas) y 3D (máscaras). Verificación de liveness con challenge-response. Comparación 1:1 selfie vs documento. FaceMap encriptado para auditoría. Funciona con cámara frontal estándar. Cumple ISO 30107-3 y NIST FRVT. |
| **Restricciones Duras (NO permite)** | **local_auth SOLO:** No distingue entre titular y otra persona del dispositivo. **Sin red:** Algunos SDKs requieren validación server-side. **Condiciones extremas:** Liveness falla con luz < 50 lux, lentes muy oscuros, barbas > 50% rostro. **Simulador:** Biometría no disponible. |
| **Criterio de Selección** | Se usa **Cubit** en lugar de BLoC porque el flujo es secuencial sin eventos complejos (captura→análisis→resultado). Cubit reduce boilerplate. Se usan **Platform Channels** para SDK nativo porque Facetec/iProov solo existen en Kotlin/Swift, procesamiento de imagen más eficiente nativo. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| local_auth | Plugin de biometría del sistema (Face ID/Touch ID/BiometricPrompt) sin verificación de identidad del titular. |
| Liveness Detection | Pruebas anti-spoofing (2D/3D) para asegurar que hay una persona viva frente a la cámara. |
| Platform Channels | Puente Flutter ↔ nativo para integrar SDKs que no existen en Dart. |
| Face Match 1:1 | Comparación de selfie vs foto de documento KYC con umbral de score mínimo. |
| ISO 30107-3 / PAD | Estándar de pruebas de Presentation Attack Detection para biometría. |
| Step-up Authentication | Escalar factores (OTP/PIN/biometría avanzada) según riesgo de la operación. |

---

## Referencias

- [ISO/IEC 30107-3:2017 - Biometric presentation attack detection](https://www.iso.org/standard/67381.html)
- [NIST FRVT - Face Recognition Vendor Test](https://www.nist.gov/programs-projects/face-recognition-vendor-test-frvt)
- [iBeta Quality Assurance - PAD Testing](https://www.ibeta.com/pad-testing/)
- [Apple Local Authentication Framework](https://developer.apple.com/documentation/localauthentication)
- [Android BiometricPrompt](https://developer.android.com/reference/android/hardware/biometrics/BiometricPrompt)

---

*Anterior: [Token que Nunca Expira](caso-01-token-nunca-expira.md) | Siguiente: [Certificate Pinning](caso-03-mitm-certificate-pinning.md)*
