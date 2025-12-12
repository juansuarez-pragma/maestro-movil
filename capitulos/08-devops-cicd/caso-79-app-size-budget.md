# Caso 79: App Size Budget
## Alertar Cuando el Bundle Supera 100MB

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | app size, budget, bundle, optimización |
| **Patrón Técnico** | Size Budgets, CI Gates, Asset Optimization |
| **Stack Seleccionado** | Flutter + CI con size check + reportes de tamaños por ABI/artefacto |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo, quiero ser alertado si la app supera 100MB antes de publicar."*

Bundles grandes afectan instalaciones, arranques y retención, especialmente en mercados con dispositivos de gama media/baja.

### Evidencia de Industria

- **Play/App Store:** Límites de tamaño afectan descargas sin WiFi.
- **Perf móvil:** Apps más pequeñas cargan más rápido y consumen menos datos.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX/Negocio** | Menos instalaciones por tamaño elevado |
| **Técnico** | Arranques más lentos y mayor consumo de recursos |
| **Operacional** | Falta de visibilidad hasta fases tardías del release |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Medir tamaño manual al final | **INADECUADO:** Se detecta tarde, sin histórico. |
| **ACEPTABLE** | Reporte de tamaño en CI sin gates | **MEJORA:** Visibilidad, pero sin enforcement. |
| **ENTERPRISE** | **Budgets + gates:** medir por ABI, alertas en CI si supera budget, reporte histórico, acciones de optimización automáticas | **ÓPTIMO:** Previene inflar la app y guía optimizaciones. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Medir tamaño de AAB/APK/IPA por ABI. Configurar budgets (p. ej., 100MB) y fallar pipeline si se excede. Reportar diffs de tamaño por commit. Sugerir acciones (split per ABI, shrink resources, deferred components). |
| **Restricciones Duras (NO permite)** | **Optimización automática limitada:** Algunas acciones requieren intervención manual. **Dependencias pesadas:** Pueden ser necesarias; requiere negociación. **iOS:** IPA incluye símbolos si no se filtra. |
| **Criterio de Selección** | Budget definido por negocio/mercados; CI con step de medición y gate; seguimiento histórico para detectar regresiones. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Size Budget | Límite de tamaño permitido para el artefacto. |
| Split per ABI | Generar binarios separados por arquitectura. |
| Shrink Resources | Remover recursos no usados en build. |
| Deferred Components | Cargar partes del app on-demand para reducir bundle inicial. |
| Gate CI | Paso que falla el pipeline si no se cumple el presupuesto. |

---

## Referencias

- [Flutter App Size Analysis](https://docs.flutter.dev/perf/app-size)
- [Google Play App Size Limits](https://support.google.com/googleplay/android-developer/answer/113469#size)
