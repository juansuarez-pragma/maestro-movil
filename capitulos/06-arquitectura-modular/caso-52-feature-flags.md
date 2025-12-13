# Caso 52: Feature Flags en Banca
## Lanzar Funciones a 1% de Usuarios sin Deploy

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | feature flags, rollout gradual, experimentación, toggles |
| **Patrón Técnico** | Feature Toggles, Progressive Delivery, Remote Config |
| **Stack Seleccionado** | Flutter + Remote Config/Flags SDK (LaunchDarkly/ConfigCat/Custom) + Riverpod |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Sin flags, cada cambio requiere release; rollback tarda horas.
- Flags hardcodeados sin expiración generan deuda y ramas de código muertas.
- Sin métricas de exposición/conversión, no se puede probar ni frenar impacto.

### Escenario de Negocio

> *"Como equipo, quiero habilitar una función a 1% sin publicar nueva versión y con rollback inmediato."*

### Incidentes reportados
- **Progressive delivery:** Usado en empresas de alto tráfico para reducir riesgo de releases.
- **Banca/fintech:** Flags permiten activar tras aprobación; incidentes por flags viejos causaron fallas tardías.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Progressive delivery reports | Global | Rollout gradual reduce incidentes en despliegues críticos. |
| Postmortems de flags | Varios | Flags sin expiración provocan comportamientos impredecibles meses después. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; gobernanza de configuración es débil. |

**Resumen global**
- Feature flags gestionados con targeting y expiración reducen riesgo y deuda; la falta de gobernanza deriva en regresiones tardías y errores masivos.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico/Reputacional** | Bugs impactan a todos sin rollback rápido |
| **Técnico** | Deuda por flags huérfanos; comportamiento impredecible |
| **Operacional** | Sin métricas no se detecta impacto negativo a tiempo |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Flags hardcodeados y sin expiración | **INADECUADO:** No hay control remoto ni limpieza. |
| **ACEPTABLE** | Remote config básico sin segmentación | **MEJORA:** Permite toggle, pero sin rollout gradual ni targeting. |
| **ENTERPRISE** | **Feature flags gestionados:** targeting, porcentajes, expiración, métricas, kill switch | **ÓPTIMO:** Rollout seguro, rollback instantáneo, gobernanza. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Rollout por porcentaje, segmento, dispositivo. Kill switch inmediato. Expiración de flags. Métricas de exposición y conversión. Evaluación local con cache y fallback offline. |
| **Restricciones Duras (NO permite)** | **Sin cleanup:** Flags viejos deben retirarse. **Latencia de fetch:** Requiere cache inicial y defaults seguros. **Seguridad:** Flags no deben exponer lógica sensible. |
| **Criterio de Selección** | SDK con targeting y gobernanza; Riverpod para exponer flags tipados; política de expiración y limpieza en cada release. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit/Contract | Tipos y defaults seguros de flags | Móvil/CI |
| Integration (CI) | Evaluación por targeting/porcentaje y fallback offline | QA/Móvil |
| Observabilidad | Exposición y conversión por flag; uso de kill switch | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Mensajes | Flags no deben revelar funcionalidades no habilitadas | Evita fuga de roadmap |
| Rollout | Progresivo con cohorts; rollback inmediato si métricas caen | Control de riesgo |
| Expiración | Definir TTL y remover flags en cada release | Reduce deuda |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Gobernanza | Dueño por flag y registro central | Responsabilidad clara |
| Seguridad | Flags solo gating, no secretos ni llaves | Minimiza exposición |
| Auditoría | Logs de cambios y toggles en producción | Trazabilidad |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Desplegar y revertir features sin lanzar versión completa. |
| Opciones evaluadas | Flags hardcodeados; remote config simple; plataforma de flags con targeting/expiración/telemetría. |
| Decisión | Feature flags gestionados con gobernanza, métricas y kill switch. |
| Consecuencias | Requiere proceso de expiración y catálogo; dependencia del SDK/servicio. |
| Riesgos aceptados | Complejidad de configuración; posibles delays de fetch si hay reglas complejas. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Tiempo de rollback | Inmediato vía kill switch | Crítico si > minutos | Riesgo acotado |
| Deuda de flags | Flags expirados removidos por release | Alerta si crece | Código limpio |
| Incidentes en rollout | ↓ vs baseline | Alerta si no baja | Calidad en producción |
| Conversión/uso por flag | Medida en cada cohort | Alerta si cae | Aprendizaje/negocio |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Feature Flag | Condicional remoto que activa/desactiva funcionalidad. |
| Progressive Delivery | Desplegar gradualmente a segmentos/porcentajes. |
| Kill Switch | Mecanismo de apagado inmediato ante incidentes. |
| Targeting | Seleccionar audiencias específicas para un flag. |
| Expiración de flags | Retirar flags antiguos para evitar deuda. |

---

## Referencias

- [Feature Toggles (Fowler)](https://martinfowler.com/articles/feature-toggles.html)
- [LaunchDarkly Docs](https://docs.launchdarkly.com/)
- [ConfigCat Targeting](https://configcat.com/docs/advanced/targeting/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
