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

### Escenario de Negocio

> *"Como usuario, quiero aprobar operaciones desde mi reloj sin sacar el teléfono."*

Usar wearables como segundo factor reduce fricción, pero debe ser seguro y vinculado al dispositivo correcto.

### Evidencia de Industria

- **MFA en wearables:** Bancos y apps de pago ofrecen aprobaciones desde el reloj.
- **Riesgos:** Notificaciones inseguras o sin binding permiten spoofing.

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
