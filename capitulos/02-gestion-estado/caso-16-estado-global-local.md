# Caso 16: Estado Global vs Local
## El Dilema del Saldo en Tiempo Real

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | estado global, estado local, saldo en tiempo real, rendimiento |
| **Patrón Técnico** | State Partitioning, [Selector](#term-selector "Función que deriva una porción del estado minimizando rebuilds.") Optimization, Background Refresh |
| **Stack Seleccionado** | Flutter + Riverpod Selectors + Stream caching + InheritedModel para widgets pesados |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- [Estado global](#term-estado-global "Datos compartidos entre pantallas (saldo, usuario).") monolítico dispara rebuilds masivos y jank; estado excesivamente local provoca saldos inconsistentes entre pantallas.
- Sin stream cache/[TTL](#term-ttl "Tiempo máximo que un dato permanece válido antes de refrescar."), cada pantalla refresca saldo → tormenta de requests y lag.
- Sin deduplicar refresh y sin selectors, se desperdicia CPU/red y se generan glitches visuales.

### Escenario de Negocio

> *"Como usuario, quiero ver mi saldo actualizado en toda la app sin que se congele la UI."*

### Incidentes reportados
- Apps bancarias (2022): saldos desincronizados generaron cientos de tickets y pérdida de confianza.
- Guías Flutter perf: recomiendan granularidad fina y selectors para minimizar rebuilds.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Reportes de soporte bancario (2022) | Apps retail | Saldos distintos en diferentes pantallas → tickets y churn. |
| Estudios de rendimiento Flutter | Global | Rebuilds masivos por estado global; selectors reducen trabajo de UI. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; estado inconsistente causa bugs UX. |

**Resumen global**
- Inconsistencias de saldo erosionan confianza y aumentan soporte.
- Rebuilds masivos y refresh redundante degradan UX; partición + selectors son prácticas recomendadas.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Datos de saldo incorrectos → llamadas al soporte, riesgos de decisiones erróneas |
| **UX** | Jank y pantallas que parpadean por rebuilds innecesarios |
| **Técnico** | Complejidad de depuración por mezcla de estado global/local inconsistente |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Un único singleton con todo el estado | **INADECUADO:** Rebuild global, difícil de testear, acoplamiento. |
| **ACEPTABLE** | Dividir estado por features sin consistencia global | **MEJORA:** Menos rebuild, pero riesgo de desalineación (saldo distinto en pantallas). |
| **ENTERPRISE** | **Partición + sincronización:** estado global para datos compartidos (saldo), estado local para UI; selectors para granularidad; refresh en background | **ÓPTIMO:** Consistencia donde importa, rendimiento controlado, testabilidad. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Mantener saldo en store global con stream cache y TTL. Usar selectors para derivar vistas mínimas sin rebuild masivo. [Estado local](#term-estado-local "Estado efímero de UI (inputs, toggles).") para inputs/UI. Refresh en background con política de TTL corta y backoff. |
| **Restricciones Duras (NO permite)** | **Sin cache compartida:** Cada pantalla refrescando saldo saturará red. **Race de refresh:** Necesita deduplicación para evitar múltiples fetch simultáneos. **Consistencia fuerte:** En presencia de múltiples dispositivos, puede haber lag hasta converger. |
| **Criterio de Selección** | Riverpod por selectores y scopes; stream caching para evitar N requests; separar Domain state (saldo) de View state (filtros, toggles). |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Unit (CI) | Selectors solo notifican cuando cambia la porción de estado | Equipo móvil, CI |
| Integration (CI) | Un solo refresh compartido alimenta múltiples pantallas (dedup) | Móvil/Backend, CI |
| Performance | Rebuilds se mantienen bajos; medir FPS/jank | QA/Perf, dispositivos reales |
| Observabilidad | Eventos `balance.refresh` con TTL, dedup y latencia | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Indicador de frescura | Mostrar timestamp/estado de sync del saldo | Transparencia al usuario |
| TTL y backoff | TTL corto (p.ej. 30-60s) + backoff ante errores | Evita tormenta de requests |
| Offline | Cachear último saldo y marcarlo como “posiblemente desactualizado” | Control de expectativas |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| [Deduplicación](#term-deduplicacion "Evitar múltiples fetch simultáneos para la misma fuente de datos.") | Evitar múltiples fetch simultáneos; compartir stream cache | Reduce carga |
| Consistencia multi-dispositivo | Validar saldo contra backend en acciones críticas | Minimiza discrepancias |
| Observabilidad | Métricas de jank, latencia de refresh y divergencia | Control continuo |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Saldos inconsistentes y jank por estado global/local mal particionado. |
| Opciones evaluadas | Singleton global; estado local por pantalla; partición global/local con selectors y cache. |
| Decisión | Partición: saldo global + selectors + cache/TTL; estado local solo para UI. |
| Consecuencias | Necesita disciplina en scopes/providers; monitoreo de rebuilds. |
| Riesgos aceptados | Consistencia eventual; dependencia de backend para verdad absoluta. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Divergencia de saldo entre pantallas | < 0.1% de sesiones | Alerta si ≥ 0.1% | Confianza del usuario |
| Rebuilds innecesarios | Reducción medible vs baseline | Alerta si sube | UX más fluida |
| Latencia de refresh saldo | p95 < 1 s | Warning si se acerca | Datos frescos percibidos |
| Tickets por saldo incorrecto | ↓ vs baseline | Alerta si no baja | Menos soporte |
| Uso de red por refresh | Controlado (dedup + TTL) | Alerta si sube | Costos y perf |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-estado-global"></a>Estado global | Datos compartidos entre pantallas (saldo, usuario). |
| <a id="term-estado-local"></a>Estado local | Estado efímero de UI (inputs, toggles). |
| <a id="term-selector"></a>Selector | Función que deriva una porción del estado minimizando rebuilds. |
| <a id="term-ttl"></a>TTL | Tiempo máximo que un dato permanece válido antes de refrescar. |
| <a id="term-stream-cache"></a>Stream cache | Almacenar última emisión de un stream para nuevos suscriptores. |
| <a id="term-deduplicacion"></a>Deduplicación | Evitar múltiples fetch simultáneos para la misma fuente de datos. |

---

## Referencias

- [Flutter Performance Best Practices](https://docs.flutter.dev/perf/best-practices)
- [Riverpod Selectors](https://riverpod.dev/docs/concepts/providers/#select)
- [Nielsen Norman - Data Freshness UX](https://www.nngroup.com/articles/data-freshness/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
