# Caso 70: In-App Updates y Control de Versiones
## Forzar Actualizaciones Críticas sin Romper la Experiencia

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | in-app updates, versiones mínimas, rollout, seguridad |
| **Patrón Técnico** | Minimum Supported Version, Forced Update, Progressive Rollout |
| **Stack Seleccionado** | Flutter + verificación de versión (backend) + Play In-App Updates (Android) + feature flags |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Versiones antiguas con vulnerabilidades permanecen activas sin control.
- Forzar update sin política genera bloqueos y reseñas negativas.
- Sin rollout gradual, un bug en la nueva versión impacta a toda la base.

### Escenario de Negocio

> *"Como banco, necesito elevar la versión mínima cuando hay un cambio crítico de seguridad, sin causar caída masiva."*

### Incidentes reportados
- **Hotfix de seguridad:** apps con versiones antiguas siguieron operando con fallas críticas.
- **Forced updates mal diseñados:** bloquearon usuarios por incompatibilidad temporal.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Android In-App Updates | Global | Flujos inmediatos o flexibles ayudan a mantener versiones al día. |
| Prácticas de “minimum version” | Global | Backend-gated minimum version es patrón común para seguridad y compatibilidad. |
| NowSecure 2024 | 1,000+ apps móviles | Dependencias/versions desactualizadas son hallazgo recurrente de seguridad. |

**Resumen global**
- Política de versión mínima (server-side) + rollout + comunicación reduce riesgo de mantener versiones vulnerables.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Seguridad** | Exposición por clientes desactualizados |
| **UX** | Bloqueo abrupto y frustración si se fuerza mal |
| **Operacional** | Soporte elevado en ventanas de update |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | No controlar versiones; esperar que el usuario actualice | **INADECUADO:** Mantiene vulnerabilidades y deuda. |
| **ACEPTABLE** | Banner para actualizar sin bloqueo | **MEJORA:** Menos fricción, pero no garantiza mitigación. |
| **ENTERPRISE** | **Minimum version + rollout:** control server-side, actualización in-app (Android), flags/canary, mensajes claros y fallback | **ÓPTIMO:** Seguridad y control de riesgo. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Definir versión mínima por entorno. Forzar update solo para incidentes críticos. Rollout por cohorte. Telemetría de adopción por versión. |
| **Restricciones Duras (NO permite)** | **iOS:** no existe in-app update equivalente; depende de App Store. **Tiempos de review:** puede retrasar mitigaciones. **Conectividad:** usuarios sin red no actualizan. |
| **Criterio de Selección** | Gating de versión en backend, mensajes graduales, rollout y métricas; in-app updates en Android cuando aplique. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Compatibilidad | Versiones mínimas y endpoints soportados | Backend/QA |
| Rollout | Cohortes y flags para mitigación | Móvil/QA |
| Observabilidad | Adopción por versión y fallos de update | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Mensajes | Explicar por qué se requiere update (seguridad/compatibilidad) | Confianza |
| Gradualidad | Banner → soft-block → hard-block (solo crítico) | Menos fricción |
| Soporte | FAQ y fallback a canales alternos | Operación |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Rollout | Canary y métricas antes de subir versión mínima | Control |
| Incidentes | Playbook para elevar min version | Respuesta |
| Métricas | % base en versión mínima y tiempo de adopción | Gobernanza |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Mantener clientes vulnerables por falta de control de versiones. |
| Opciones evaluadas | No controlar; banners; min version server-side con rollout y update in-app Android. |
| Decisión | Minimum supported version desde backend + rollout y comunicación; in-app updates en Android cuando aplique. |
| Consecuencias | Requiere coordinación backend/tiendas y manejo de UX; iOS limitado. |
| Riesgos aceptados | Dependencia de tiempos de store; usuarios offline quedan atrás. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Base en versión mínima | > 95% | Warning si baja | Seguridad |
| Tiempo de adopción | Reducido (días) | Alerta si se extiende | Operación |
| Tickets por update | Tendencia a la baja | Warning si sube | Soporte |
| Incidentes por legacy | ↓ | Crítico si no baja | Riesgo |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Versión mínima | Versión mínima permitida por backend/política. |
| Soft-block | Bloqueo suave que permite posponer temporalmente. |
| Hard-block | Bloqueo obligatorio hasta actualizar (solo crítico). |
| Rollout | Despliegue gradual por cohortes. |
| Canary | Cohorte pequeña para validar antes de escalar. |

---

## Referencias

- [Android In-app updates](https://developer.android.com/guide/playcore/in-app-updates)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
