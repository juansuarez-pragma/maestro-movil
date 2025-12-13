# Caso 78: [Dark Launch](#term-dark-launch "Desplegar funcionalidad oculta en producción.")
## Desplegar en Producción sin Activar la Funcionalidad

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | dark launch, progressive delivery, feature flags, riesgo |
| **Patrón Técnico** | Dark Launch, [Shadow Traffic](#term-shadow-traffic "Duplicar tráfico a nueva funcionalidad sin afectar respuesta al usuario."), Dual Execution |
| **Stack Seleccionado** | Flutter + flags/experiments SDK + duplicación de requests (shadow) + observabilidad |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Activar funcionalidad al 100% sin validar en prod eleva riesgo de incidentes.
- Sin shadow/flags, rollback es lento y afecta a toda la base.
- Sin telemetría previa, el go-live es ciego.

### Escenario de Negocio

> *"Como equipo, quiero desplegar una funcionalidad y probarla en producción sin exponerla a usuarios."*

### Incidentes reportados
- **Shadow traffic/dark launches:** Evitaron incidentes en despliegues de rutas críticas.
- **Equipos sin dark launch:** Bugs ocultos impactaron a todos en el go-live.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Progressive delivery | Global | Dark launch reduce riesgo antes de activar. |
| Postmortems de releases | Varios | Falta de flags/shadow provocó regresiones masivas. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; gobernanza/flags suele ser débil. |

**Resumen global**
- Dark launch con shadow traffic y flags permite validar en producción y rollback rápido; activaciones directas exponen a todos los usuarios.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Técnico** | Bugs ocultos salen al go-live sin pruebas en prod |
| **UX** | Impacto directo si la funcionalidad falla al activarse |
| **Operacional** | Difícil rollback si no hay flags/killswitch |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Desplegar y activar inmediatamente | **INADECUADO:** Riesgo alto. |
| **ACEPTABLE** | Toggle manual en prod sin shadow | **MEJORA:** Algo de control, pero sin validación previa. |
| **ENTERPRISE** | **Dark launch + shadow:** desplegar apagado, probar con shadow traffic/QA interna, observabilidad, luego activar gradualmente | **ÓPTIMO:** Riesgo reducido y rollback rápido. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Deploy con flag apagado. Enviar shadow traffic sin impactar respuesta al usuario. Monitorear métricas y errores. Activar gradualmente a segmentos. [Rollback](#term-rollback "Apagar o revertir rápidamente ante fallas.") inmediato via flag. |
| **Restricciones Duras (NO permite)** | **Shadow incompleto:** No replica 100% de casos. **Costo:** Duplicar requests aumenta carga. **Privacidad:** Debe cumplir tratamiento de datos aun en shadow. |
| **Criterio de Selección** | Flags con killswitch; shadow para rutas críticas; monitoreo antes de habilitar; activación gradual. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Shadow | Flujo corre en paralelo sin impactar usuario | Móvil/QA |
| Monitoreo | Errores/latencia en shadow antes de activar | Móvil/SRE |
| Rollout | Activación gradual con rollback vía flag | Móvil/QA |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Fallback | Kill switch inmediato | MTTR bajo |
| Comunicación | Estado de dark launch visible para soporte/negocio | Transparencia |
| Segmentos | Activar por cohortes tras shadow estable | Riesgo acotado |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Costos | Limitar shadow a rutas críticas | Eficiencia |
| Privacidad | Cumplir PII y masking también en shadow | Cumplimiento |
| Observabilidad | Eventos `dark_launch.*` y métricas comparativas | Control |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Despliegues directos sin validar en prod. |
| Opciones evaluadas | Activar al 100%; toggle sin shadow; dark launch con shadow y flags. |
| Decisión | Dark launch + shadow con observabilidad y activación gradual. |
| Consecuencias | Coste adicional de shadow y monitoreo; requiere flags sólidos. |
| Riesgos aceptados | Cobertura parcial de shadow; overhead operacional. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Incidentes en go-live | Tendencia a la baja | Crítico si sube | Estabilidad |
| Tiempo a rollback | Minutos vía flag | Crítico si tarda | MTTR |
| Errores en shadow | Detectados antes de activar | Warning si pasan a prod | Prevención |
| Coste de shadow | Controlado | Alerta si crece | Eficiencia |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-dark-launch"></a>Dark Launch | Desplegar funcionalidad oculta en producción. |
| <a id="term-shadow-traffic"></a>Shadow Traffic | Duplicar tráfico a nueva funcionalidad sin afectar respuesta al usuario. |
| <a id="term-flag-kill-switch"></a>Flag/Kill Switch | Control remoto para activar/desactivar features. |
| <a id="term-progressive-delivery"></a>Progressive Delivery | Activar features gradualmente con control. |
| <a id="term-rollback"></a>Rollback | Apagar o revertir rápidamente ante fallas. |

---

## Referencias

- [Shadow Launch Patterns](https://martinfowler.com/articles/feature-toggles.html)
- [Progressive Delivery](https://launchdarkly.com/blog/progressive-delivery/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
