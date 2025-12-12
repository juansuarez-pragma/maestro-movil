# Caso 71: CI/CD Móvil Confiable
## Pipelines Reproducibles para Flutter en Banca

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | CI/CD, pipelines móviles, reproducibilidad, release |
| **Patrón Técnico** | Pipeline as Code, Caching, Artifact Management |
| **Stack Seleccionado** | Flutter + Fastlane/Codemagic/GitHub Actions + caches Gradle/pub + firmas automatizadas |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo, necesito pipelines reproducibles y rápidos que generen builds firmados sin intervención manual."*

Pipelines frágiles causan retrasos en releases y errores de firma.

### Evidencia de Industria

- **Equipos móviles:** Uso de pipelines declarativos y cache para reducir tiempos.
- **Banca:** Requiere trazabilidad y cumplimiento en releases.

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
| **Capacidades (SÍ permite)** | Cache de pub/Gradle para acelerar builds. Jobs paralelos (lint/test/build). Firmas automatizadas con llaves en vault/secret manager. Generar artefactos por entorno (dev/stage/prod). Release notes automáticas. |
| **Restricciones Duras (NO permite)** | **Llaves expuestas:** Deben estar en vault, no en repo. **Infra lenta:** Cache y runners potentes son necesarios. **Compliance:** Auditoría y retención de artefactos obligatorias. |
| **Criterio de Selección** | Pipeline declarativo; cache efectivo; firmas seguras; gates de calidad; artefactos versionados y trazables. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Pipeline as Code | Definir CI/CD en archivos versionados. |
| Cache de build | Reutilizar dependencias y outputs para acelerar. |
| Vault/Secret Manager | Almacén seguro para llaves y secretos. |
| Artifact | Resultado de build (APK/AAB/IPA) almacenado y versionado. |
| Gate | Paso obligatorio (tests/linters) antes de avanzar. |

---

## Referencias

- [Fastlane for Flutter](https://docs.fastlane.tools/getting-started/ios/flutter/)
- [GitHub Actions Caching](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)
