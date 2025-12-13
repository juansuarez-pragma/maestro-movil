# Caso 93: Dividir un Mono-Repo Legacy
## Separar en Múltiples Repos sin Perder Trazabilidad

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | monorepo split, migración, dependencias, trazabilidad |
| **Patrón Técnico** | Repository Split, Dependency Mapping, History Preservation |
| **Stack Seleccionado** | Git filter-repo/subtree + mapping de dependencias + CI por repo |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Copiar carpetas sin historia rompe trazabilidad y dependencias ocultas.
- Splits sin inventario provocan builds rotos y referencias cruzadas perdidas.
- Sin CI por repo, la migración bloquea releases.

### Escenario de Negocio

> *"Como equipo, necesitamos dividir un monorepo legacy en repos separados manteniendo historial y builds."*

### Incidentes reportados
- **Splits fallidos:** Perdieron historia y rompieron dependencias cruzadas, generando retrasos.
- **Repos nuevos sin CI:** Bloqueos de release por pipelines ausentes.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Splits de monorepo | Global | filter-repo/subtree preservan historia si se mapean paths. |
| Postmortems | Varios | Falta de inventario/CI causó builds rotos. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; gobernanza/config es brecha común. |

**Resumen global**
- Split planificado con inventario, historia preservada y CI por repo evita roturas; copias sin historia generan deuda y pérdida de trazabilidad.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Productivo** | Builds rotos tras el split |
| **Técnico** | Dependencias cruzadas no mapeadas |
| **Reputacional** | Retrasos en entregas por migración fallida |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Copiar carpetas a nuevos repos sin historial | **INADECUADO:** Se pierde trazabilidad y rompe dependencias. |
| **ACEPTABLE** | Split parcial con history shallow | **MEJORA:** Mantiene algo de historia, pero dependencias pueden quedar rotas. |
| **ENTERPRISE** | **Split planificado:** inventario de dependencias, filter-repo/subtree para preservar historia, CI configurado por repo, mapeo de referencias y documentación | **ÓPTIMO:** Trazabilidad y builds preservados. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Preservar historial de archivos movidos. Mapear dependencias antes del split. Configurar CI y versiones por repo. Documentar referencias cruzadas y reemplazos. |
| **Restricciones Duras (NO permite)** | **Dependencias ocultas:** Requiere análisis previo. **Historia parcial:** Puede perder commits si paths no se configuran bien. **Costo inicial:** Scripts y validación del split. |
| **Criterio de Selección** | filter-repo/subtree para historia; inventario de dependencias; CI por repo; guía de migración para developers. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Historia | Commits/pull requests preservados en cada repo | DevOps |
| Dependencias | Módulos compilando tras split | QA/Móvil |
| CI | Pipelines por repo ejecutan tests/lints | DevOps |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Documentación | Guía de rutas y reemplazos | Onboarding |
| Versionado | Estrategia de semver y publicación para repos nuevos | Consistencia |
| Comunicación | Calendario y alcance del split | Alineación |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Backups | Respaldar antes de reescrituras | Seguridad |
| Auditoría | Referencias cruzadas registradas | Trazabilidad |
| Gobernanza | Dueños por repo y políticas de cambios | Control |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Separar monorepo preservando historia y builds. |
| Opciones evaluadas | Copia sin historia; split parcial; split planificado con filter-repo/subtree + CI y documentación. |
| Decisión | Split planificado con preservación de historia, inventario de dependencias y CI por repo. |
| Consecuencias | Requiere scripting y coordinación; validaciones de builds en cada repo. |
| Riesgos aceptados | Posibles gaps de historia si paths no se configuran bien; esfuerzo inicial alto. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Builds rotos tras split | 0 | Crítico si >0 | Continuidad |
| Historia preservada | Confirmada por archivos clave | Warning si faltan | Trazabilidad |
| Tiempo de setup de CI | Predecible | Alerta si crece | Productividad |
| Dependencias cruzadas | Reducidas/eliminadas | Crítico si persisten | Salud |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| filter-repo | Herramienta para reescribir historial Git preservando paths. |
| Subtree split | Extraer un directorio con su historia a otro repo. |
| Dependencia cruzada | Uso de código entre módulos que se separarán. |
| Trazabilidad | Capacidad de seguir cambios a través de repos. |
| CI por repo | Pipelines separados adaptados a cada nuevo repositorio. |

---

## Referencias

- [git filter-repo](https://github.com/newren/git-filter-repo)
- [Git subtree](https://git-scm.com/book/en/v1/Git-Tools-Subtree-Merging)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
