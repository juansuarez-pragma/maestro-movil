# Caso 57: Plugin Architecture
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

### Escenario de Negocio

> *"Como plataforma, quiero que terceros agreguen funcionalidades sin comprometer seguridad ni estabilidad."*

Permitir plugins requiere contratos claros, aislamiento y control de permisos.

### Evidencia de Industria

- **Super apps:** Exponen SDKs/plugin systems con APIs limitadas.
- **Riesgos:** Plugins maliciosos/fallidos pueden filtrar datos o romper UX.

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
| **BAJA** | Plugins con acceso total al app core | **INADECUADO:** Riesgo alto de seguridad/estabilidad. |
| **ACEPTABLE** | APIs limitadas pero sin sandbox | **MEJORA:** Reduce riesgo, pero aún acoplado. |
| **ENTERPRISE** | **Plugins sandbox:** contratos de API, permisos explícitos, revisión/firmas, monitoreo, revocación | **ÓPTIMO:** Extensible con control y seguridad. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Exponer APIs contractuales limitadas (pagos, perfiles). Control de permisos por plugin. Firmar y verificar plugins. Telemetría y aislamiento lógico. Revocar/actualizar plugins centralmente. |
| **Restricciones Duras (NO permite)** | **Aislamiento total:** Flutter no provee sandbox duro; políticas deben ser estrictas. **Plugins nativos:** Requieren revisión adicional. **Versioning:** Cambios de API implican gestionar compatibilidad. |
| **Criterio de Selección** | Diseñar contratos mínimos; permisos explícitos; proceso de revisión/firmado; monitoreo de performance/seguridad de plugins. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Plugin | Módulo externo que amplía funcionalidad del host. |
| Sandbox lógico | Restricción de capacidades y APIs disponibles. |
| Permisos | Declaración explícita de acciones/datos permitidos. |
| Firma/verificación | Garantizar integridad y autoría del plugin. |
| Revocación | Deshabilitar un plugin que viola políticas o falla. |

---

## Referencias

- [Plugin Architecture Patterns](https://martinfowler.com/articles/micro-frontends.html)
- [Capability-based Security](https://en.wikipedia.org/wiki/Capability-based_security)
