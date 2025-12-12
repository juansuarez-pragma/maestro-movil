# Caso 72: Firma y Certificados sin Dolor
## Gestionar Keystores y Provisioning Profiles con Seguridad

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | firma, certificados, keystore, provisioning profile, fastlane |
| **Patrón Técnico** | Secure Key Management, Automated Signing, Certificates Rotation |
| **Stack Seleccionado** | Fastlane match/cert/pilot + Secret Manager/Vault + CI |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo, necesito firmar builds Android/iOS sin compartir llaves por chat ni romper provisioning."*

Manejo manual de llaves y perfiles provoca builds fallidos y riesgo de filtración.

### Evidencia de Industria

- **Fastlane match/cert:** Estándar para compartir credenciales de forma segura.
- **Incidentes:** Llaves expuestas en repos generan compromisos de apps.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Seguridad** | Llaves filtradas permiten firmar apps maliciosas |
| **Productivo** | Builds fallan por perfiles vencidos |
| **Reputacional** | Revocación de certificados y pérdida de confianza |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Compartir llaves por email/chat | **INADECUADO:** Riesgo máximo de filtración. |
| **ACEPTABLE** | Guardar llaves en repo privado cifrado | **MEJORA:** Mejor, pero aún riesgoso y propenso a roturas. |
| **ENTERPRISE** | **Automatizar con vault + fastlane:** llaves en Secret Manager, rotación programada, provisioning automático, acceso por rol | **ÓPTIMO:** Seguro, repetible y auditable. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Almacenar keystores/profiles en vault o match repo cifrado. Rotar certificados y provisionings de forma controlada. Integrar con CI para firma automática. Auditoría de acceso. |
| **Restricciones Duras (NO permite)** | **Llaves hardcodeadas:** Prohibido. **Rotación manual sin plan:** Riesgo de caducidad. **Dependencia de personas:** Se elimina con automatización. |
| **Criterio de Selección** | Fastlane para automatizar firmas; Secret Manager/Vault para llaves; acceso por rol; rotación documentada; monitoreo de expiración. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Keystore | Archivo que contiene claves para firmar apps Android. |
| Provisioning Profile | Perfil que autoriza apps iOS a correr en dispositivos. |
| Secret Manager/Vault | Servicio seguro para guardar secretos. |
| Rotación de certificados | Renovar certificados antes de expiración. |
| Fastlane match/cert | Herramientas para gestionar credenciales de firma iOS/Android. |

---

## Referencias

- [Fastlane Match](https://docs.fastlane.tools/actions/match/)
- [Fastlane Cert/Pilot](https://docs.fastlane.tools/actions/)
