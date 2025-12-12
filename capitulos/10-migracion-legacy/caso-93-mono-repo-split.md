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

### Escenario de Negocio

> *"Como equipo, necesitamos dividir un monorepo legacy en repos separados manteniendo historial y builds."*

Un monorepo mal gestionado puede ser lento; dividirlo sin plan rompe dependencias y trazabilidad.

### Evidencia de Industria

- **Splits exitosos:** Usan filter-repo/subtree para preservar historia y scripts para mapear dependencias.
- **Splits fallidos:** Pierden historial y rompen builds por dependencias ocultas.

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
| **Restricciones Duras (NO permite)** | **Dependencias ocultas:** Requiere análisis previo. **Historia parcial:** Algunos commits pueden perderse si no se configuran paths correctamente. **Costo inicial:** Scripts y validación del split. |
| **Criterio de Selección** | filter-repo/subtree para historia; inventario de dependencias; CI por repo; guía de migración para developers. |

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
