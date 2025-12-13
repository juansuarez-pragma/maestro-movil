# Caso 91: Migrar Monolito Flutter a Modular
## Desacoplar sin Detener Releases

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | migración, monolito, modularización, refactor |
| **Patrón Técnico** | [Strangler Fig](#term-strangler-fig "Patrón para reemplazar sistemas gradualmente."), Modularization, Feature Extraction |
| **Stack Seleccionado** | Flutter + packages internos + Melos + feature flags para cutover |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Reescritura big bang congela releases y genera regresiones masivas.
- Dependencias cíclicas y sin control hacen la modularización ingobernable.
- Sin CI por paquete, el refactor rompe áreas que no se probaron.

### Escenario de Negocio

> *"Como equipo, necesitamos modularizar un monolito Flutter grande sin parar releases."*

### Incidentes reportados
- **Migraciones fallidas:** Big bang causó meses sin releases y regresiones.
- **Ciclos de dependencia:** Dificultaron extraer features y frenaron velocity.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Migraciones móviles | Global | Enfoque incremental con packages y flags reduce riesgo. |
| Monorepos | Global | CI por paquetes afectados acelera feedback. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; gobernanza de dependencias/config es débil. |

**Resumen global**
- Strangler incremental con paquetes, flags y CI por afectación permite seguir liberando mientras se modulariza; big bang es de alto riesgo.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Productivo** | Congelamiento si se hace big bang |
| **Técnico** | Regresiones por mover código masivo |
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
| **Criterio de Selección** | Strangler; paquetes por dominio; Melos/monorepo; flags para cutover; monitoreo de regresiones por módulo. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Tests por paquete | Integridad tras extracción | QA/Móvil |
| Afectados | CI ejecuta solo lo impactado | DevOps |
| Flags | [Cutover](#term-cutover "Paso de sistema viejo a nuevo.") seguro y rollback | Móvil/QA |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Rollout | Activar nuevas rutas por flag | Riesgo acotado |
| Observabilidad | Métricas por módulo extraído | Control |
| Degradación | Fallback a código viejo si falla nuevo | Estabilidad |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Gobernanza | Grafo de dependencias y lint de ciclos | Salud |
| Versionado | Semver/lock de paquetes internos | Consistencia |
| Plan | Roadmap de extracción y hitos por dominio | Transparencia |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Modularizar sin frenar releases ni introducir regresiones masivas. |
| Opciones evaluadas | Big bang; extracción sin plan; strangler incremental con paquetes/flags/CI afectado. |
| Decisión | Strangler incremental con paquetes por dominio, flags de cutover y CI por afectación. |
| Consecuencias | Overhead de coordinación y gobernanza de dependencias; esfuerzo en configurar CI. |
| Riesgos aceptados | Complejidad inicial; deuda temporal durante transición. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Tiempo de build/tests | ↓ con CI por afectación | Warning si no baja | Velocidad |
| Regresiones por módulo | Tendencia a la baja | Crítico si sube | Estabilidad |
| Bloqueos de release | 0 por modularización | Crítico si hay | Continuidad |
| Dependencias cíclicas | 0 detectadas | Crítico si >0 | Salud del repo |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-strangler-fig"></a>Strangler Fig | Patrón para reemplazar sistemas gradualmente. |
| <a id="term-cutover"></a>Cutover | Paso de sistema viejo a nuevo. |
| <a id="term-package-interno"></a>Package interno | Módulo independiente en el monorepo. |
| <a id="term-dependencia-ciclica"></a>Dependencia cíclica | Referencia circular entre módulos; debe evitarse. |
| <a id="term-afectacion-por-paquete"></a>Afectación por paquete | Detectar qué paquetes cambian para limitar pruebas/build. |

---

## Referencias

- [Strangler Fig](https://martinfowler.com/bliki/StranglerFigApplication.html)
- [Melos Monorepo](https://melos.invertase.dev/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
