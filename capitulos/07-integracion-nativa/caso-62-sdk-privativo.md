# Caso 62: SDK Privativo con Licenciamiento
## Integrar SDKs con Keys, Obfuscation y Auditoría

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | sdk privativo, licenciamiento, obfuscation, seguridad |
| **Patrón Técnico** | Secure SDK Integration, Key Management, Obfuscation |
| **Stack Seleccionado** | Flutter + nativo (Kotlin/Swift) + ProGuard/R8 + obfuscation Dart + secure storage de keys |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo, debemos integrar un SDK comercial con licencias y claves que no pueden filtrarse."*

Claves expuestas o SDK mal ofuscado pueden ser pirateados o bloqueados por el proveedor.

### Evidencia de Industria

- **SDKs biométricos/pagos:** Requieren claves y licencias; filtrarlas puede invalidar contratos.
- **Prácticas de seguridad:** Ofuscación y manejo seguro de keys son obligatorios.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Seguridad/Legal** | Revocación de licencias, pérdida de contrato, fraude |
| **Técnico** | Crashes por mal manejo de versiones; incompatibilidad en CI |
| **Reputacional** | Pérdida de confianza con proveedor |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Claves en código fuente, sin ofuscación | **INADECUADO:** Claves filtrables; riesgo máximo. |
| **ACEPTABLE** | Claves en variables de entorno, ofuscación nativa básica | **MEJORA:** Menos riesgo, pero puede filtrar en apps decompiladas. |
| **ENTERPRISE** | **Gestión segura:** keys en backend o storage seguro, ofuscación Dart/nativo, validación de licencias, attestation de integridad | **ÓPTIMO:** Minimiza filtración y asegura cumplimiento. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Cargar claves desde backend con binding a device/user. Guardar tokens de licencia en secure storage. Ofuscar código Dart (dart2js obfuscation) y nativo (R8/ProGuard). Validar integridad (Play Integrity/App Attest). |
| **Restricciones Duras (NO permite)** | **Keys en cliente:** Siempre hay riesgo; mejor mantener en backend cuando sea posible. **Emuladores/root:** SDK puede bloquearse; manejar mensajes claros. **CI/CD:** Debe gestionar licencias sin exponerlas en logs. |
| **Criterio de Selección** | Minimizar exposición de claves; ofuscar; attestation; manejar licencias de forma centralizada cuando sea viable. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Licenciamiento | Validación de uso autorizado del SDK. |
| Ofuscación | Transformar código para dificultar ingeniería inversa. |
| Secure Storage | Almacenamiento cifrado en dispositivo. |
| Attestation | Verificación de integridad del dispositivo/app. |
| Key rotation | Rotación periódica de claves/licencias. |

---

## Referencias

- [Android R8/ProGuard](https://developer.android.com/studio/build/shrink-code)
- [iOS Obfuscation Guidelines](https://developer.apple.com/documentation/)
- [Play Integrity / App Attest](https://developer.android.com/google/play/integrity)
