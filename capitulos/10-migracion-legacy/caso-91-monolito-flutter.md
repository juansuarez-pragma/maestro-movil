# Caso 91: Migrar Monolito Flutter a Modular
## Desacoplar sin Detener Releases

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | migración, monolito, modularización, refactor |
| **Patrón Técnico** | Strangler Fig, Modularization, Feature Extraction |
| **Stack Seleccionado** | Flutter + packages internos + Melos + feature flags para cutover |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo, necesitamos modularizar un monolito Flutter grande sin parar releases."*

Refactors grandes pueden bloquear el delivery si no se hacen incrementalmente.

### Evidencia de Industria

- **Migraciones exitosas:** Usan enfoque incremental con paquetes internos y flags.
- **Monorepos:** Facilitan extracción de features sin duplicar código.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Productivo** | Congelamiento si se hace big bang |
| **Técnico** | Regressiones por mover código masivo |
| **Reputacional** | Releases atrasados afectan negocio |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Reescritura completa en paralelo | **INADECUADO:** Riesgo alto y tiempo prolongado. |
| **ACEPTABLE** | Extraer algunos módulos sin plan de cutover | **MEJORA:** Algo de progreso, pero caos en dependencias. |
| **ENTERPRISE** | **Strangler incremental:** crear paquetes por feature, limpiar dependencias, flags para cutover, CI con afectación por paquete | **ÓPTIMO:** Delivery continuo y refactor controlado. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Extraer features a packages internos gradualmente. Mantener releases continuos. Tests/CI por paquete afectado. Flags para alternar nueva/antigua implementación. |
| **Restricciones Duras (NO permite)** | **Big bang:** Evitar. **Dependencias cíclicas:** Requiere disciplina. **Versionado:** Paquetes internos deben versionarse/lockearse. |
| **Criterio de Selección** | Strangler; paquetes por dominio; melos/monorepo; flags para cutover; monitoreo de regresiones por módulo. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Strangler Fig | Patrón para reemplazar sistemas gradualmente. |
| Cutover | Paso de sistema viejo a nuevo. |
| Package interno | Módulo independiente en el monorepo. |
| Dependencia cíclica | Referencia circular entre módulos; debe evitarse. |
| Afectación por paquete | Detectar qué paquetes cambian para limitar pruebas/build. |

---

## Referencias

- [Strangler Fig](https://martinfowler.com/bliki/StranglerFigApplication.html)
- [Melos Monorepo](https://melos.invertase.dev/)
