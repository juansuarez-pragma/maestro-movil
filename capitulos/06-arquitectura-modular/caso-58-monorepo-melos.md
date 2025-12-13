# Caso 58: Mono-Repo con [Melos](#term-melos "Tooling para gestionar workspaces Dart/Flutter.")
## Gestionar 15 Packages sin Perder la Cordura

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | monorepo, melos, packages, versionado |
| **Patrón Técnico** | [Monorepo](#term-monorepo "Repositorio único con múltiples paquetes.") Management, Workspace Tooling, Semantic Versioning |
| **Stack Seleccionado** | Flutter + Melos + Git hooks + CI para release/linters |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Repos separados o scripts ad-hoc rompen dependencias y sincronización de versiones.
- Sin tooling, builds y tests para 15 packages son lentos e inconsistentes.
- Changelogs/versionado manual generan releases con información incompleta.

### Escenario de Negocio

> *"Como equipo, necesitamos coordinar 15 packages Flutter sin caos en versiones y builds."*

### Incidentes reportados
- **Monorepos sin tooling:** Builds rotos por dependencias desalineadas.
- **Equipos grandes:** Sin filtros `--since`, los pipelines tardan y bloquean releases.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Proyectos Flutter monorepo | Global | Melos es estándar para bootstrap, lint/test y versionado. |
| Postmortems de release | Varios | Falta de semver/changelog provocó regresiones no documentadas. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; gobernanza de dependencias/config es débil. |

**Resumen global**
- Melos + semver + CI con filtros reduce tiempo de build y riesgos de versiones rotas; sin ello los monorepos se vuelven inmanejables.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Técnico** | Versiones inconsistentes, builds rotos |
| **Productivo** | Tiempo perdido en coordinación manual |
| **Reputacional** | Releases con bugs por falta de pruebas coordinadas |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Repos separados sin coordinación | **INADECUADO:** Rompe dependencias, releases dolorosos. |
| **ACEPTABLE** | Scripts ad-hoc para múltiples packages | **MEJORA:** Algo de ayuda, pero frágil y manual. |
| **ENTERPRISE** | **Melos + CI:** comandos unificados (bootstrap, format, test), versionado semver, changelogs automáticos, filtros por paquete | **ÓPTIMO:** Consistencia y productividad. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Bootstrapping en lote. Comandos de lint/test por paquete o afectados. Versionado/changelog automáticos. Hooks git/CI para evitar drift. |
| **Restricciones Duras (NO permite)** | **Conflictos de dependencias:** Requiere disciplina en versiones. **Builds largos:** Necesita caching/CI distribuido. **Gobernanza:** Melos no reemplaza políticas de revisión. |
| **Criterio de Selección** | Melos para workspace; semver disciplinado; CI con `melos list --since` y caché de dependencias. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| CI afectado | Tests/lints solo en paquetes cambiados | Móvil/CI |
| Release | Versionado semver y changelog generados | Móvil/QA |
| Observabilidad | Duración de pipelines y fallas por paquete | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Onboarding | Scripts `melos bootstrap` y docs claras | Reduce fricción |
| Consistencia | Formato/lint comunes ejecutados vía Melos | Calidad |
| Publicación | Pipelines predecibles con gates de calidad | Confianza |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| [Semver](#term-semver "Versionado semántico para comunicar compatibilidad.") | Cambios mayores requerirán plan de migración | Estabilidad |
| Cache | Habilitar caché de dependencias y build | Velocidad |
| Seguridad | Revisión de dependencias y firmas en releases | Cumplimiento |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Coordinar 15 packages sin versiones rotas ni builds lentos. |
| Opciones evaluadas | Repos separados; scripts ad-hoc; Melos + semver + CI con filtros. |
| Decisión | Melos como estándar, semver estricto, pipelines con filtros y caché. |
| Consecuencias | Requiere disciplina en versionado y mantenimiento de tooling. |
| Riesgos aceptados | Complejidad inicial de setup; dependencia en Melos. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Duración de pipeline | ↓ vs baseline | Warning si sube | Velocidad |
| Builds rotos por versión | 0 | Crítico si >0 | Estabilidad |
| Tiempo de release | Predecible con changelog auto | Alerta si crece | Cadencia |
| Errores de dependencia | Tendencia a la baja | Crítico si sube | Calidad |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-monorepo"></a>Monorepo | Repositorio único con múltiples paquetes. |
| <a id="term-melos"></a>Melos | Tooling para gestionar workspaces Dart/Flutter. |
| <a id="term-bootstrap"></a>Bootstrap | Instalar deps y preparar workspace. |
| <a id="term-semver"></a>Semver | Versionado semántico para comunicar compatibilidad. |
| <a id="term-affected-packages"></a>Affected packages | Paquetes impactados por un cambio. |

---

## Referencias

- [Melos Documentation](https://melos.invertase.dev/)
- [Semver](https://semver.org/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
