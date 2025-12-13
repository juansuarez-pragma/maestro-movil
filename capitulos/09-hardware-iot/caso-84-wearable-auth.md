# Caso 84: El Wearable que Autoriza
## Apple Watch como Segundo Factor

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | wearable, apple watch, segundo factor, mfa |
| **Patrón Técnico** | Wearable MFA, Device Binding, Secure Notifications |
| **Stack Seleccionado** | Flutter + Platform Channels (WatchConnectivity/Companion) + push seguro + binding a dispositivo |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Notificaciones simples sin binding permiten approvals en relojes no autorizados.
- Sin payload firmado, se puede spoofear la transacción.
- Dependencia de conectividad con el teléfono rompe la UX si no se gestiona.

### Escenario de Negocio

> *"Como usuario, quiero aprobar operaciones desde mi reloj sin sacar el teléfono."*

### Incidentes reportados
- **MFA en wearables:** Implementaciones sin binding permitieron aprobaciones no autorizadas.
- **Payload inseguro:** Mensajes sin firma fueron interceptados/spoofeados.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| MFA en banca | Global | Relojes usados como segundo factor con binding y firma. |
| Seguridad en notificaciones | Global | Payload sin firma expone a spoofing. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; controles de autenticación son críticos. |

**Resumen global**
- MFA en wearables exige binding fuerte, payload firmado y revocación; sin ello, spoofing y approvals indebidos son probables.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Seguridad** | Aprobaciones en dispositivos no vinculados |
| **UX** | Flujos rotos si no hay conectividad con el teléfono |
| **Regulatorio** | MFA debe cumplir SCA/PSD2 en pagos |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Notificaciones simples sin binding | **INADECUADO:** Riesgo de aprobación indebida. |
| **ACEPTABLE** | Aprobación con app compendio, sin binding fuerte | **MEJORA:** Menor riesgo, pero aún vulnerable a spoofing. |
| **ENTERPRISE** | **MFA con binding:** vincular wearable a cuenta/device, push seguro con payload firmado, biometría del reloj si disponible, revocación remota | **ÓPTIMO:** Fricción baja y seguridad alta. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Vincular reloj con ID de dispositivo y cuenta. Validar payload firmado en reloj. Requerir biometría del reloj (passcode/bio) si disponible. Sincronizar estado con teléfono. Revocar binding desde backend. |
| **Restricciones Duras (NO permite)** | **Dependencia de teléfono:** Muchas acciones requieren el teléfono cerca. **Compatibilidad:** APIs varían (WatchOS vs WearOS). **Seguridad física:** Reloj desbloqueado puede aprobar si no se exige bio. |
| **Criterio de Selección** | Binding fuerte, firmas en payload, biometría/lock del reloj, revocación y sync consistente. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Seguridad | Binding y firma de payload | Seguridad/QA |
| Integration (WatchOS/WearOS) | Flujos de aprobación y revocación | Móvil/QA |
| Observabilidad | Eventos `wearable.auth.*` con resultado y dispositivo | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Mensajes | Mostrar datos de la operación y expiración | Claridad |
| Conectividad | Manejo offline/reenvío desde teléfono | Continuidad |
| Revocación | Opción clara para desvincular reloj | Control usuario |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Binding | Asociar a cuenta y device ID, no reutilizar | Seguridad |
| Biométrico | Usar passcode/bio del reloj si existe | Protección |
| Cumplimiento | SCA/PSD2 donde aplique | Regulación |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Aprobar operaciones desde reloj sin spoofing ni uso indebido. |
| Opciones evaluadas | Notificaciones simples; app companion sin binding fuerte; MFA con binding, payload firmado y biometría. |
| Decisión | MFA con binding fuerte, payload firmado, biometría y revocación remota. |
| Consecuencias | Requiere coordinación con companion app y backends de firma. |
| Riesgos aceptados | Dependencia de conectividad; variabilidad de APIs de reloj. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Aprobaciones indebidas | 0 | Crítico si >0 | Seguridad |
| Éxito de aprobación en reloj | > 98% | Warning si baja | UX |
| Tiempos de aprobación | Estables | Alerta si suben | Experiencia |
| Revocaciones ejecutadas | Rápidas y auditadas | Crítico si fallan | Control |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Wearable MFA | Uso de reloj como factor de autenticación. |
| Binding de dispositivo | Asociación de un dispositivo específico a la cuenta. |
| Payload firmado | Mensaje con firma para integridad/autenticidad. |
| Companion App | App móvil que comunica con el wearable. |
| Revocación | Quitar permisos/binding de un dispositivo. |

---

## Referencias

- [Apple Watch Connectivity](https://developer.apple.com/documentation/watchconnectivity)
- [WearOS Complications/Connectivity](https://developer.android.com/training/wearables/apps)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
