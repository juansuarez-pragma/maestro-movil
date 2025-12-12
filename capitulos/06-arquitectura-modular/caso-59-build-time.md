# Caso 59: Build Time de 45 Minutos
## Estrategias de Compilación Incremental

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | build time, compilación incremental, productividad, CI |
| **Patrón Técnico** | Incremental Builds, Caching, Affected-Only |
| **Stack Seleccionado** | Flutter + Melos/Monorepo + build caching (Bazel/Gradle) + CI paralelo |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo, no podemos esperar 45 minutos por cada build; necesitamos feedback rápido."*

Builds lentos reducen productividad y retrasan releases.

### Evidencia de Industria

- **Monorepos grandes:** Usan caching y builds incrementales para acelerar ciclos.
- **CI/CD:** Pipelines rápidos reducen lead time.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Productivo** | Menos iteraciones, mayor costo de cambios |
| **Económico** | Infra CI costosa por builds largos |
| **Reputacional** | Releases lentos, menor calidad por falta de feedback |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Build completo siempre | **INADECUADO:** Tiempo y costo altos. |
| **ACEPTABLE** | Cache parcial en CI | **MEJORA:** Acelera algo, pero no discrimina cambios. |
| **ENTERPRISE** | **Incremental + affected-only:** cache binario, pruebas por cambios, CI paralelo, split por módulos, warm caches locales | **ÓPTIMO:** Feedback rápido, menor costo. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Detectar paquetes afectados y construir solo esos. Usar caches compartidos (Gradle/Bazel/Flutter build). Paralelizar jobs. Warm cache en CI. Dividir artefactos por ABI. |
| **Restricciones Duras (NO permite)** | **Cambios globales:** tocan todo. **Caches inconsistentes:** Requiere invalidación correcta. **Herramientas:** Bazel/rules_flutter necesitan inversión inicial. |
| **Criterio de Selección** | Affected-only con Melos; caches robustos; paralelismo en CI; medir y ajustar invalidación. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Build incremental | Compilar solo lo que cambió. |
| Affected-only | Ejecutar build/tests solo en módulos impactados. |
| Build cache | Reutilizar outputs para evitar recompilar. |
| Paralelismo CI | Ejecutar jobs en paralelo para reducir tiempo total. |
| Split per ABI | Artefactos separados por arquitectura para optimizar tamaño/tiempo. |

---

## Referencias

- [Melos - Affected Packages](https://melos.invertase.dev/commands/list)
- [Gradle Build Cache](https://docs.gradle.org/current/userguide/build_cache.html)
- [Bazel Remote Cache](https://bazel.build/remote/caching)
