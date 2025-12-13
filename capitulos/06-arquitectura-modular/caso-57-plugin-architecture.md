# Caso 57: [Plugin](#term-plugin "Módulo externo que amplía funcionalidad del host.") Architecture
## Permitir que Terceros Extiendan tu App Bancaria

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | plugin architecture, extensibilidad, sandbox, terceros |
| **Patrón Técnico** | Plugin System, Sandbox Contracts, Capability-based Security |
| **Stack Seleccionado** | Flutter + módulos pluginizados + contratos de APIs + revisión/firmas |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Plugins con acceso amplio pueden filtrar datos o degradar la app.
- Sin contratos/firmas, terceros pueden romper compatibilidad o introducir malware.
- Falta de monitoreo/revocación hace difícil contener incidentes.

### Escenario de Negocio

> *"Como plataforma, quiero que terceros agreguen funcionalidades sin comprometer seguridad ni estabilidad."*

### Incidentes reportados
- **Super apps:** Exponen SDKs/plugin systems con APIs limitadas y procesos de review; plugins maliciosos llevaron a restricciones más fuertes.
- **Apps con extensiones:** Sin políticas de permisos se degradó la UX y se filtraron datos.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Super apps (APAC) | APAC | Plugins limitados por API/permiso y revisión previa. |
| Postmortems de extensiones | Varios | Falta de sandbox generó fugas de datos y crashes. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; controles de plugins/extensiones son críticos. |

**Resumen global**
- Extender apps sin sandbox y permisos controlados es riesgoso; se requieren contratos mínimos, firmas, telemetría y kill-switch.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Seguridad** | Acceso indebido a datos/acciones |
| **UX** | Caídas/jank por plugins deficientes |
| **Operacional** | Dificultad de versionar y revocar plugins |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Plugins con acceso total al core | **INADECUADO:** Riesgo alto de seguridad/estabilidad. |
| **ACEPTABLE** | APIs limitadas pero sin sandbox/firmas | **MEJORA:** Reduce riesgo, pero aún acoplado. |
| **ENTERPRISE** | **Plugins sandbox:** contratos de API, permisos explícitos, revisión/firmas, monitoreo y revocación | **ÓPTIMO:** Extensible con control y seguridad. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Exponer APIs contractuales limitadas (pagos, perfil). Control de permisos por plugin. Firmar y verificar plugins. Telemetría y aislamiento lógico. Revocar/actualizar plugins centralmente. |
| **Restricciones Duras (NO permite)** | **Aislamiento total:** Flutter no provee sandbox duro; políticas deben ser estrictas. **Plugins nativos:** Requieren revisión adicional. **Compatibilidad:** Cambios de API requieren versionado y contrato claro. |
| **Criterio de Selección** | Diseñar contratos mínimos; permisos explícitos; proceso de revisión/firmado; monitoreo de performance/seguridad de plugins. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Seguridad | [Permisos](#term-permisos "Declaración explícita de acciones/datos permitidos.") y acceso a APIs respetados | Seguridad/QA |
| Integration (CI) | Compatibilidad de contratos plugin-host | Móvil/QA |
| Observabilidad | Telemetría `plugin.*` (latencia, errores, permisos usados) | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Onboarding de plugin | Checklist de revisión + firma | Calidad |
| Degradación | Desactivar plugin ante fallo; fallback a experiencia base | UX protegida |
| Mensajes | Avisos claros de permisos y origen de plugin | Transparencia |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Kill-switch | Revocar plugins desde backend | Contención |
| Versionado | Semver y compatibilidad de contratos | Estabilidad |
| Auditoría | Registro de instalaciones y permisos | Cumplimiento |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Extender la app con plugins sin comprometer seguridad/estabilidad. |
| Opciones evaluadas | Acceso total; APIs limitadas sin sandbox; plugins con contratos, permisos y revisión. |
| Decisión | Plugins con contratos mínimos, permisos explícitos, firmas y monitoreo con kill-switch. |
| Consecuencias | Requiere proceso de revisión/auditoría y tooling de firma. |
| Riesgos aceptados | Sandbox no es completo; overhead operativo. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Incidentes de seguridad por plugins | 0 | Crítico si >0 | Seguridad |
| Latencia/errores por plugin | Controlados y observables | Alerta si suben | UX estable |
| Tiempo de publicación de plugin | Predictible con checklist | Warning si crece | Flujo de partners |
| Revocaciones | Ejecutables en minutos | Crítico si no | Contención |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-plugin"></a>Plugin | Módulo externo que amplía funcionalidad del host. |
| <a id="term-sandbox-logico"></a>Sandbox lógico | Restricción de capacidades y APIs disponibles. |
| <a id="term-permisos"></a>Permisos | Declaración explícita de acciones/datos permitidos. |
| <a id="term-firma-verificacion"></a>Firma/verificación | Garantizar integridad y autoría del plugin. |
| <a id="term-revocacion"></a>Revocación | Deshabilitar un plugin que viola políticas o falla. |

---

## Referencias

- [Plugin Architecture Patterns](https://martinfowler.com/articles/micro-frontends.html)
- [Capability-based Security](https://en.wikipedia.org/wiki/Capability-based_security)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
