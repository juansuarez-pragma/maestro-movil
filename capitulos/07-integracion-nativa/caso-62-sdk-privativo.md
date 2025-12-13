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

### Problema detectado (técnico)
- Claves/licencias en código se filtran al decompilar; riesgo de revocación y fraude.
- Sin ofuscación ni attestation, el SDK puede usarse en apps no autorizadas.
- Manejo pobre de versiones/licencias en CI rompe builds y soporte.

### Escenario de Negocio

> *"Como equipo, debemos integrar un SDK comercial con licencias y claves que no pueden filtrarse."*

### Incidentes reportados
- **SDKs biométricos/pagos:** Claves expuestas en repositorios/apks resultaron en revocación de licencias.
- **Fraude:** Uso del SDK en apps clonadas al no validar integridad/entorno.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Incidentes de filtración de keys | Global | Repos y APK decompilados revelan licencias; proveedores revocan acceso. |
| Seguridad móvil | Global | Integridad/attestation reduce uso en dispositivos rooteados/clonados. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; exposición de secretos y configuración es común. |

**Resumen global**
- Guardar claves en cliente sin ofuscación ni attestation expone licencias; mover claves al backend o protegerlas con storage seguro/rotación es obligatorio.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Seguridad | Secrets no están en repos/APK; attestation habilitada | Seguridad |
| Integration (CI) | Licencias cargan y no se exponen en logs | Móvil/CI |
| Observabilidad | Eventos `license.*` y fallos de integridad | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Mensajes | Errores claros si falla licencia/integridad | Evita confusión |
| Cache | Token de licencia en secure storage con rotación | Seguridad |
| Soporte | Matriz de versiones/licencias soportadas | Previsibilidad |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Rotación | Rotar keys/licencias periódicamente | Reduce exposición |
| Build/CI | Inyectar credenciales vía vault/secrets | Cumplimiento |
| Attestation | Bloquear root/emulador según política | Defensa |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Integrar SDK privativo sin filtrar licencias ni romper builds. |
| Opciones evaluadas | Claves en código; variables de entorno + ofuscación básica; gestión segura con storage/attestation/rotación. |
| Decisión | Gestionar claves desde backend/secure storage, ofuscar Dart/nativo, attestation y rotación. |
| Consecuencias | Mayor complejidad operativa y dependencias de seguridad/CI. |
| Riesgos aceptados | Operación de rotación; dependencia del proveedor para validaciones. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Filtración de licencias | 0 | Crítico si >0 | Cumplimiento |
| Éxito de licenciamiento | > 99% | Warning si baja | Disponibilidad |
| Incidentes por integridad | Tendencia a la baja | Crítico si sube | Seguridad |
| Tiempo de rotación de keys | Predecible | Alerta si se retrasa | Riesgo acotado |

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
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
