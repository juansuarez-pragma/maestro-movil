# Caso 64: FIDO2/Passkeys en Flutter
## Autenticación Sin Contraseña con WebAuthn

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | fido2, passkeys, weauthn, autenticación sin contraseña |
| **Patrón Técnico** | WebAuthn, Public Key Credentials, Device-bound Keys |
| **Stack Seleccionado** | Flutter + Platform Channels (FIDO2/Passkeys APIs) + secure storage de credenciales |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Contraseñas/OTP expuestos a phishing y credential stuffing.
- Integrar Passkeys sin guías claras causa fallos de compatibilidad y UX confusa.
- Sin backend WebAuthn adecuado, el flujo falla o se degrada a secretos débiles.

### Escenario de Negocio

> *"Como usuario, quiero autenticación segura sin contraseñas vulnerables a phishing."*

### Incidentes reportados
- **Credential stuffing:** Ataques masivos por reuse de contraseñas.
- **Implementaciones parciales:** Apps con fallbacks inseguros que mantienen riesgo de phishing.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| FIDO Alliance | Global | Passkeys reducen phishing y reuse de contraseñas. |
| Breach reports | Global | Credential stuffing sigue siendo vector principal en banca. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; autenticación débil es hallazgo típico. |

**Resumen global**
- Passkeys eliminan contraseñas y reducen phishing; requieren compatibilidad por SO, backend WebAuthn y UX clara con fallback seguro.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Seguridad** | Phishing y credential stuffing se reducen, pero requieren UX clara |
| **Compatibilidad** | Soporte varía por SO/versión; fallback necesario |
| **Técnico** | Integración nativa y manejo de claves requiere cuidado |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Solo contraseñas/OTP | **INADECUADO:** Riesgo alto de phishing. |
| **ACEPTABLE** | OTP + biometría local_auth | **MEJORA:** Mejor, pero aún depende de secrets débiles. |
| **ENTERPRISE** | **Passkeys/WebAuthn:** registro y autenticación con claves públicas, challenge-response, fallback controlado | **ÓPTIMO:** Sin contraseñas, resistente a phishing, UX moderna. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Generar credenciales FIDO2 ligadas al dispositivo. Autenticación con challenge-response y biometría. Sin almacenar contraseñas. Fallback a OTP solo cuando necesario. |
| **Restricciones Duras (NO permite)** | **Compatibilidad parcial:** Versiones antiguas pueden no soportar. **Multi-dispositivo:** Sin sync de passkeys se requiere registro por dispositivo. **Errores UX:** Necesario guiar al usuario en prompts nativos. |
| **Criterio de Selección** | Usar APIs nativas FIDO2/Passkeys via Channels; backend con WebAuthn; fallback seguro; almacenamiento de credenciales en secure storage. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Integration (Android/iOS) | Registro/autenticación FIDO2/Passkeys y fallbacks | Móvil/QA |
| Seguridad | Protección contra phishing y reuse; sin secretos guardados | Seguridad |
| Compatibilidad | Matriz de SO y dispositivos soportados | QA/Móvil |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Mensajes | Guiar prompts nativos y fallback seguro | Reduce abandono |
| Fallback | OTP solo si Passkeys no disponible | Riesgo acotado |
| Recuperación | Re-registro simple en nuevos dispositivos | Continuidad |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Backend | Soporte WebAuthn con challenge/attestation | Integridad |
| Storage | Credential IDs en secure storage | Seguridad |
| Auditoría | Eventos `auth.*` con método y resultado | Trazabilidad |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Autenticación vulnerable a phishing y reuse de contraseñas. |
| Opciones evaluadas | Password/OTP; OTP+biometría; Passkeys/WebAuthn con fallback seguro. |
| Decisión | Implementar Passkeys/WebAuthn, fallback controlado y backend compatible. |
| Consecuencias | Requiere coordinación backend y UX nativa; matriz de compatibilidad. |
| Riesgos aceptados | Dependencia de soporte de plataforma; fricción inicial de adopción. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Incidentes de phishing | ↓ vs baseline | Crítico si no baja | Seguridad |
| Éxito de login con Passkey | > 99% en dispositivos soportados | Warning si baja | UX |
| Uso de fallback (OTP) | ↓ progresivo | Alerta si alto | Adopción |
| Abandono en login | Tendencia a la baja | Warning si sube | Conversión |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Passkey | Credencial FIDO2 ligada a dispositivo/biometría. |
| WebAuthn | Estándar de autenticación basada en claves públicas. |
| Challenge | Valor único enviado por el servidor para validar autenticación. |
| Credential ID | Identificador de la credencial almacenada. |
| Relying Party | Servicio que solicita autenticación WebAuthn. |

---

## Referencias

- [FIDO2 / WebAuthn](https://fidoalliance.org/specifications/)
- [Passkeys Platform Docs](https://developer.apple.com/passkeys/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
