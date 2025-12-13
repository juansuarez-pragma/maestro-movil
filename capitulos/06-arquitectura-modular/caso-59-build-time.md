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

### Problema detectado (técnico)
- Builds completos toman 45 minutos; feedback lento frena entregas.
- Sin affected-only ni cache, se recompila todo aunque cambie un paquete.
- Pipelines largos elevan costos de CI y reducen calidad (menos iteraciones).

### Escenario de Negocio

> *"Como equipo, no podemos esperar 45 minutos por cada build; necesitamos feedback rápido."*

### Incidentes reportados
- **Monorepos grandes:** Sin cache/affected-only, pipelines se vuelven cuellos de botella.
- **Postmortems CI:** Caches inconsistentes generaron builds irreproducibles.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Equipos Flutter monorepo | Global | Uso de Melos + affected-only reduce tiempo de pipeline. |
| DevOps reports | Global | Caching correcto reduce 30-50% tiempos de build. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; pipelines lentos afectan respuesta a parches. |

**Resumen global**
- Affected-only + cache + paralelismo reducen tiempo y costo; requiere buena invalidación para evitar builds corruptos.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Productivo** | Menos iteraciones, mayor costo de cambio |
| **Económico** | Infra CI costosa por builds largos |
| **Calidad** | Releases lentos, menos pruebas completas |

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
| **Restricciones Duras (NO permite)** | **Cambios globales:** Tocan todo. **Caches inconsistentes:** Requieren invalidación correcta. **Herramientas:** Bazel/rules_flutter necesitan inversión inicial. |
| **Criterio de Selección** | Affected-only con Melos; caches robustos; paralelismo en CI; medir y ajustar invalidación. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Affected-only | Solo paquetes cambiados se construyen | Móvil/CI |
| Cache | Hit-rate y reproducibilidad de builds | DevOps/CI |
| Performance | Duración de pipeline vs baseline | QA/Perf |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Feedback | Gates de calidad rápidos (lint/test afectados) | Mejora velocidad |
| Observabilidad | Métricas de build por etapa y paquete | Diagnóstico |
| Alertas | Umbrales de tiempo/costo de pipeline | Correcciones tempranas |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Invalidación | Estrategia clara de cache busting | Evita artefactos corruptos |
| Paralelismo | Configurar workers y límites de recursos | Estabilidad |
| Seguridad | Firmar artefactos cacheados | Integridad |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Builds de 45 min ralentizan feedback y releases. |
| Opciones evaluadas | Build completo; cache parcial; incremental con affected-only y paralelismo. |
| Decisión | Incremental + affected-only, cache remoto y pipelines paralelos con monitoreo. |
| Consecuencias | Requiere tooling de cache y definición de invalidación; curva de setup. |
| Riesgos aceptados | Posibles inconsistencias de cache; inversión en infra. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Duración de pipeline | ↓ vs baseline | Warning si sube | Velocidad |
| Hit-rate de cache | Alto y estable | Alerta si cae | Eficiencia |
| Costo de CI | ↓ con builds reducidos | Alerta si sube | Ahorro |
| Tiempo a rollback | Rápido por builds cortos | Crítico si lento | Respuesta |

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
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
