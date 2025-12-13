# Caso 56: [Shared Kernel](#term-shared-kernel "Subconjunto de modelo/contratos compartidos y estables.")
## Extraer Código Común sin Crear un Monolito

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | shared kernel, código común, módulos, dependencia cruzada |
| **Patrón Técnico** | Shared Kernel, Domain Library, Semantic Versioning |
| **Stack Seleccionado** | Flutter + packages internos (monorepo) + [Melos](#term-melos "Herramienta para gestionar monorepos de Dart/Flutter.") + semver |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Sin kernel acotado, se duplican modelos/contratos y aparecen dependencias cíclicas.
- Un paquete gigante obliga a publicar todo ante cambios pequeños.
- Cambios sin semver ni contratos rompen múltiples módulos a la vez.

### Escenario de Negocio

> *"Como equipo, necesitamos compartir modelos/contratos sin acoplar todos los módulos."*

### Incidentes reportados
- **DDD:** Shared Kernel debe ser pequeño/estable; kernels grandes derivan en monolitos disfrazados.
- **Monorepos:** Cambios sin semver causaron builds rotos en múltiples apps.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Proyectos DDD | Global | Shared kernels grandes generan ripple effects y deuda. |
| Monorepos móviles | LATAM/EU | [Semver](#term-semver "Versionado semántico para comunicar compatibilidad.") y contratos reducen roturas de build. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; gobernanza de dependencias es crítica. |

**Resumen global**
- Kernel pequeño y versionado reduce acoplamiento y roturas masivas; gobernanza y contract testing son clave.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Técnico** | Acoplamiento, actualizaciones masivas, regresiones |
| **Productivo** | Velocidad se frena por dependencias frágiles |
| **Operacional** | Sin contratos/semver, los release trains se rompen |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Todo compartido en un módulo gigante | **INADECUADO:** Monolito, sin límites claros. |
| **ACEPTABLE** | Librería común sin gobernanza | **MEJORA:** Comparte código, pero riesgo de roturas frecuentes. |
| **ENTERPRISE** | **Shared kernel acotado:** contratos y modelos estables, semver, revisiones de cambio, tests contractuales | **ÓPTIMO:** Reuso sin acoplamiento excesivo, cambios previsibles. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Definir modelos/contratos comunes estables. Publicar como package interno versionado. Ejecutar tests contractuales entre módulos. Minimizar dependencias hacia el kernel. |
| **Restricciones Duras (NO permite)** | **Kernel gigante:** Debe ser pequeño. **Cambios frecuentes:** Aumentan rompimientos; requiere governance. **Dependencias cíclicas:** Prohibidas; revisar grafos de dependencias. |
| **Criterio de Selección** | Kernel con semver, revisión de cambios, tests contractuales; uso de Melos/monorepo para manejo de versiones y releases. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Contract tests | Consumidores y kernel coinciden en modelos/contratos | QA/Móvil |
| Integration (CI) | Versiones compatibles en release trains | Móvil/CI |
| Observabilidad | Grafo de dependencias y cambios por versión | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Compatibilidad | Mantener backward compatibility en cambios menores | Evita regresiones |
| Documentación | Changelog y migra guías por versión | Onboarding rápido |
| Distribución | Paquetes internos firmados/cacheados | Confiabilidad |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Gobernanza | Board de cambios y semver obligatorio | Estabilidad |
| Tamaño | Límite de superficie del kernel; revisión de nuevas dependencias | Control de alcance |
| Automatización | CI que bloquea dependencias cíclicas | Salud del repo |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Reusar modelos/contratos sin caer en monolito o ripple effects. |
| Opciones evaluadas | Sin kernel; kernel gigante; kernel acotado con semver y contratos. |
| Decisión | Kernel acotado, versionado semver, con contract tests y revisión de cambios. |
| Consecuencias | Requiere gobernanza y tooling de monorepo; disciplina de cambios. |
| Riesgos aceptados | Overhead de versiones; esfuerzo de mantener contratos. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Roturas por cambio de kernel | Tendencia a la baja | Crítico si sube | Estabilidad |
| Tiempo de adopción de versión | Ciclo predecible | Warning si se extiende | Release cadence |
| Tamaño del kernel | Mantener acotado (budget) | Alerta si crece | Modularidad |
| Dependencias cíclicas | 0 | Crítico si >0 | Salud del repo |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-shared-kernel"></a>Shared Kernel | Subconjunto de modelo/contratos compartidos y estables. |
| <a id="term-semver"></a>Semver | Versionado semántico para comunicar compatibilidad. |
| <a id="term-contract-testing"></a>Contract Testing | Validar que consumidores y proveedor coinciden en contratos. |
| <a id="term-monorepo"></a>Monorepo | Repositorio único que aloja múltiples paquetes. |
| <a id="term-melos"></a>Melos | Herramienta para gestionar monorepos de Dart/Flutter. |

---

## Referencias

- [Domain-Driven Design - Shared Kernel](https://www.domainlanguage.com/ddd/)
- [Semantic Versioning](https://semver.org/)
- [Melos](https://melos.invertase.dev/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
