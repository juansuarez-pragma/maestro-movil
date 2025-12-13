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

### Problema detectado (técnico)
- Bundle crece sin visibilidad; se detecta tarde y bloquea releases.
- Tamaño elevado reduce instalaciones en mercados con datos limitados y ralentiza arranque.
- Sin budgets/gates, no hay accountability ni plan de optimización.

### Escenario de Negocio

> *"Como equipo, quiero ser alertado si la app supera 100MB antes de publicar."*

### Incidentes reportados
- **Play/App Store:** Límites de tamaño afectaron descargas sin WiFi.
- **Apps pesadas:** Installs y retención caen en dispositivos de gama media/baja.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Google Play size guidance | Global | Tamaño afecta conversión en descargas móviles. |
| Estudios de perf móvil | Global | Apps más ligeras reducen TTI y consumo de datos. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; optimización y configuración deficiente común. |

**Resumen global**
- Budgets con gates en CI y reportes por ABI previenen inflar el app y protegen conversión en mercados sensibles a tamaño.

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
| **Capacidades (SÍ permite)** | Medir tamaño de AAB/APK/IPA por ABI. Configurar budgets (ej. 100MB) y fallar pipeline si se excede. Reportar diffs por commit. Sugerir split per ABI, shrink resources, deferred components. |
| **Restricciones Duras (NO permite)** | **Optimización automática limitada:** Algunas acciones requieren intervención manual. **Dependencias pesadas:** Negociar para reducir. **iOS:** IPA incluye símbolos si no se filtra. |
| **Criterio de Selección** | Budget definido por negocio/mercados; step de medición y gate en CI; seguimiento histórico para detectar regresiones. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Size check CI | Tamaños por ABI vs budget | DevOps/CI |
| Reportes | Diffs de tamaño por commit/release | Móvil/Perf |
| Optimización | Acciones (shrink/split/deferred) aplicadas | Móvil/Perf |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Alertas | Notificar si se acerca al budget | Prevención |
| Targets | Budgets por mercado/tienda | Ajuste local |
| Transparencia | Dashboard de tamaño y tendencias | Visibilidad |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Gate | Pipeline falla si supera budget | Control |
| Revisión | Checklist de dependencias pesadas | Disciplina |
| Retención | Medir impacto en conversión/instalación | Negocio |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Falta de control sobre crecimiento del bundle. |
| Opciones evaluadas | Medición manual; reporte sin gates; budget con gates y acciones de optimización. |
| Decisión | Budgets con gates y reportes por ABI; plan de optimización. |
| Consecuencias | Requiere CI con steps de medición y disciplina de dependencias. |
| Riesgos aceptados | Posible fricción en releases si el budget es estricto. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Tamaño por ABI | ≤ budget (ej. 100MB) | Crítico si excede | Instalaciones |
| TTI/arranque | Mejora o estable | Warning si empeora | UX |
| Conversión de instalación | Estable/↑ en mercados sensibles | Alerta si cae | Negocio |
| Incidentes por tamaño en store | 0 | Crítico si >0 | Compliance tienda |

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
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
