# Caso 71: CI/CD Móvil Confiable
## Pipelines Reproducibles para Flutter en Banca

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | CI/CD, pipelines móviles, reproducibilidad, release |
| **Patrón Técnico** | [Pipeline as Code](#term-pipeline-as-code "Definir CI/CD en archivos versionados."), Caching, [Artifact](#term-artifact "Resultado de build (APK/AAB/IPA) almacenado y versionado.") Management |
| **Stack Seleccionado** | Flutter + Fastlane/Codemagic/GitHub Actions + caches Gradle/pub + firmas automatizadas |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Builds manuales o pipelines frágiles generan retrasos y errores de firma.
- Sin cache ni paralelismo, los tiempos de pipeline exceden ventanas de release.
- Llaves mal gestionadas exponen seguridad y cumplimiento.

### Escenario de Negocio

> *"Como equipo, necesito pipelines reproducibles y rápidos que generen builds firmados sin intervención manual."*

### Incidentes reportados
- **Equipos móviles:** Pipelines sin cache/paralelo duplicaron tiempos y rompieron releases críticos.
- **Banca:** Llaves en repositorios provocaron revocaciones y auditorías.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Mobile DevOps surveys | Global | Cache + jobs paralelos reducen 30-50% duración de pipelines. |
| Incidentes de llaves expuestas | Global | Exposición de keystores provocó regeneración y bloqueos de publicación. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; gestión de secretos/config es débil. |

**Resumen global**
- Pipelines declarativos con cache/paralelismo y llaves en vault reducen tiempos y riesgos; pipelines manuales generan deuda y fallas de cumplimiento.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Productivo** | Builds fallidos, demoras en releases |
| **Seguridad** | Manejo inseguro de llaves de firma |
| **Calidad** | Falta de gates de tests/linters causa regresiones |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Builds manuales locales | **INADECUADO:** Inconsistente, sin trazabilidad. |
| **ACEPTABLE** | Pipeline básico sin cache ni firmas automatizadas | **MEJORA:** Repetible pero lento; riesgo en manejo de llaves. |
| **ENTERPRISE** | **CI/CD robusto:** pipeline como código, cache binario, tests/linters gates, firmas automatizadas, artefactos versionados | **ÓPTIMO:** Rápido, seguro, auditable. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Cache de pub/Gradle para acelerar builds. Jobs paralelos (lint/test/build). Firmas automatizadas con llaves en vault/secret manager. Artefactos por entorno (dev/stage/prod). Release notes automáticas. |
| **Restricciones Duras (NO permite)** | **Llaves expuestas:** Deben estar en vault, no en repo. **Infra lenta:** Cache y runners potentes son necesarios. **Compliance:** Auditoría y retención de artefactos obligatorias. |
| **Criterio de Selección** | Pipeline declarativo; cache efectivo; firmas seguras; gates de calidad; artefactos versionados y trazables. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Pipeline dry-run | Declaraciones y gates correctos | DevOps |
| Seguridad | Llaves en vault y secretos enmascarados | Seguridad |
| Performance | Duración de pipeline vs baseline; hit-rate de cache | DevOps/QA |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Notificaciones | Alertas en fallos/gates y éxito de release | Transparencia |
| Artefactos | Publicar AAB/APK/IPA versionados | Reproducibilidad |
| Rollback | Artefactos previos disponibles | Respuesta rápida |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Firmas | Keystores y certificados en vault, rotación | Cumplimiento |
| Observabilidad | Métricas de pipeline y fallos por etapa | Mejora continua |
| Branching | Estrategia clara (main/release) con gates | Control |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Pipelines lentos e inseguros afectan releases y cumplimiento. |
| Opciones evaluadas | Builds manuales; pipeline básico; pipeline declarativo con cache, firmas seguras y gates. |
| Decisión | CI/CD robusto con cache/paralelismo, llaves en vault y gates de calidad. |
| Consecuencias | Requiere inversión en DevOps y monitoreo; manejo disciplinado de llaves. |
| Riesgos aceptados | Dependencia de runner/infra; tuning de cache. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Duración de pipeline | ↓ 30-50% vs baseline | Warning si no baja | Velocidad |
| Hit-rate de cache | Alto y estable | Alerta si cae | Eficiencia |
| Incidentes por llaves | 0 | Crítico si >0 | Seguridad |
| Releases fallidos | Tendencia a la baja | Crítico si sube | Calidad |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-pipeline-as-code"></a>Pipeline as Code | Definir CI/CD en archivos versionados. |
| <a id="term-cache-de-build"></a>Cache de build | Reutilizar dependencias y outputs para acelerar. |
| <a id="term-vault-secret-manager"></a>Vault/Secret Manager | Almacén seguro para llaves y secretos. |
| <a id="term-artifact"></a>Artifact | Resultado de build (APK/AAB/IPA) almacenado y versionado. |
| <a id="term-gate"></a>Gate | Paso obligatorio (tests/linters) antes de avanzar. |

---

## Referencias

- [Fastlane for Flutter](https://docs.fastlane.tools/getting-started/ios/flutter/)
- [GitHub Actions Caching](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
