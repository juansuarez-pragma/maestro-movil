# Caso 72: Firma y Certificados sin Dolor
## Gestionar Keystores y Provisioning Profiles con Seguridad

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | firma, certificados, keystore, provisioning profile, fastlane |
| **Patrón Técnico** | Secure Key Management, Automated Signing, Certificates Rotation |
| **Stack Seleccionado** | [Fastlane match/cert](#term-fastlane-match-cert "Herramientas para gestionar credenciales de firma iOS/Android.")/pilot + [Secret Manager/Vault](#term-secret-manager-vault "Servicio seguro para guardar secretos.") + CI |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Llaves compartidas por chat/email se filtran y permiten firmar apps maliciosas.
- Perfiles vencidos rompen builds en el momento del release.
- Sin rotación ni auditoría, el cumplimiento se debilita.

### Escenario de Negocio

> *"Como equipo, necesito firmar builds Android/iOS sin compartir llaves por chat ni romper provisioning."*

### Incidentes reportados
- **Fastlane match/cert:** Estándar para compartir credenciales; equipos sin match sufrieron revocaciones por filtración.
- **Llaves expuestas:** Repos públicos/privados mal configurados filtraron keystores.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Incidentes de claves expuestas | Global | Filtraciones provocaron revocaciones y reprovisionamientos urgentes. |
| Prácticas móviles | Global | Automatizar firmas reduce fallos de release y mejora cumplimiento. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; gestión de secretos es brecha común. |

**Resumen global**
- Firmas seguras requieren vault + automatización + rotación; el manejo manual expone seguridad y frena releases.

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
| **Restricciones Duras (NO permite)** | **Llaves hardcodeadas:** Prohibido. **Rotación manual sin plan:** Riesgo de caducidad. **Dependencia de personas:** Evitar mediante automatización y roles. |
| **Criterio de Selección** | Fastlane para automatizar firmas; Secret Manager/Vault para llaves; acceso por rol; rotación documentada; monitoreo de expiración. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Seguridad | Llaves no están en repo/logs; acceso auditado | Seguridad/DevOps |
| Release dry-run | Firma automática en CI con perfiles vigentes | Móvil/QA |
| Rotación | Renovación programada sin romper builds | DevOps |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Alertas | Notificar expiración próxima de perfiles/certs | Prevención |
| Acceso | Roles mínimos para firmar | Menor superficie |
| Documentación | Pasos de recuperación/rotación | Resiliencia |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Vault | Secret Manager/Vault cifrado y versionado | Seguridad |
| Match repo | Cifrado y accesos limitados | Control |
| Auditoría | Logs de acceso/uso de llaves | Cumplimiento |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Firmar apps con llaves seguras y provisioning vigente sin riesgo de filtración. |
| Opciones evaluadas | Compartir llaves; repo cifrado; vault + fastlane automatizado con rotación. |
| Decisión | Vault + fastlane (match/cert) con rotación y auditoría. |
| Consecuencias | Necesita setup inicial y gobernanza de accesos. |
| Riesgos aceptados | Dependencia de servicios de vault; coordinación en rotaciones. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Incidentes por claves expuestas | 0 | Crítico si >0 | Seguridad |
| Builds fallidos por perfiles | Tendencia a la baja | Warning si sube | Estabilidad |
| Tiempo de rotación | Predecible y sin downtime | Alerta si se extiende | Continuidad |
| Auditorías exitosas | 100% | Alerta si falla | Cumplimiento |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-keystore"></a>Keystore | Archivo que contiene claves para firmar apps Android. |
| <a id="term-provisioning-profile"></a>Provisioning Profile | Perfil que autoriza apps iOS a correr en dispositivos. |
| <a id="term-secret-manager-vault"></a>Secret Manager/Vault | Servicio seguro para guardar secretos. |
| <a id="term-rotacion-de-certificados"></a>Rotación de certificados | Renovar certificados antes de expiración. |
| <a id="term-fastlane-match-cert"></a>Fastlane match/cert | Herramientas para gestionar credenciales de firma iOS/Android. |

---

## Referencias

- [Fastlane Match](https://docs.fastlane.tools/actions/match/)
- [Fastlane Cert/Pilot](https://docs.fastlane.tools/actions/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
