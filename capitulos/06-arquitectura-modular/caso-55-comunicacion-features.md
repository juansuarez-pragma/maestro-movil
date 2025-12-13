# Caso 55: Comunicación entre Features
## [Event Bus](#term-event-bus "Canal para publicar/suscribirse a eventos.") vs BLoC-to-BLoC

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | comunicación interna, módulos, eventos, acoplamiento |
| **Patrón Técnico** | Event Bus, [Mediator](#term-mediator "Componente que coordina comunicaciones entre módulos."), Shared State Contracts |
| **Stack Seleccionado** | Flutter + Riverpod scoped providers + Event channels tipados |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Llamadas directas entre features generan dependencias circulares y regresiones.
- Eventos sin contrato se rompen al modificar payloads; trazabilidad pobre.
- Sin scopes, un evento local puede impactar a toda la app.

### Escenario de Negocio

> *"Como equipo modular, necesito que features se comuniquen sin acoplarse y sin duplicar lógica."*

### Incidentes reportados
- **Micro-frontends:** Usan event bus/mediator tipado para evitar acoplamiento; incidentes por eventos sin versión causaron bugs de sincronía.
- **Apps grandes:** Cambios en payloads rompieron features no coordinados.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Micro-frontends | Global | Mediators/buses tipados reducen regresiones entre módulos. |
| Postmortems de eventos | Varios | Eventos sin contrato/versionado provocan fallas cruzadas. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; governance/config es punto débil. |

**Resumen global**
- Contratos tipados y scopes por módulo reducen acoplamiento y facilitan debugging; buses globales sin gobernanza acumulan deuda.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Técnico** | Cascadas de cambios y efectos colaterales |
| **UX** | Estados desalineados entre features |
| **Operacional** | Dificultad para evolucionar módulos independientemente |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Llamadas directas entre features | **INADECUADO:** Acoplamiento y dependencias cíclicas. |
| **ACEPTABLE** | Event bus global sin contratos | **MEJORA:** Menos acoplamiento, pero caos si no hay contratos. |
| **ENTERPRISE** | **Bus tipado/mediator:** eventos tipados, contratos claros, scopes por módulo, handlers aislados | **ÓPTIMO:** Desacople, trazabilidad y evolución independiente. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Publicar/escuchar eventos tipados entre módulos. Scopes por feature con providers. Logging y tracing de eventos para debugging. Sustituir handlers en testing. |
| **Restricciones Duras (NO permite)** | **Eventos sin contrato:** Riesgo de drift si no se tipan/versionan. **Abuso de globales:** Bus global puede volverse dumping ground; requiere gobernanza. **Orden garantizado:** Si se necesita orden total, usar colas o coordinación. |
| **Criterio de Selección** | Bus/mediator tipado para desacople; Riverpod scoped providers para inyección de handlers; contratos y versionado de eventos. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Contract/Unit | Estructura y versión de eventos | Móvil/CI |
| Integration (CI) | Entrega de eventos entre módulos con scopes | QA/Móvil |
| Observabilidad | Trazas `event.*` con origen/destino y latencia | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Filtros | Solo suscribirse a eventos del scope relevante | Evita ruido |
| Idempotencia | Handlers deben tolerar re-entrega | Robustez |
| Degradación | Silenciar eventos no críticos si fallan | UX estable |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Gobernanza | Catálogo y versionado de eventos | Control de cambios |
| Seguridad | Evitar que eventos contengan datos sensibles sin cifrado | Protección |
| Testing | Simular eventos cross-feature en CI | Previene regresiones |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Comunicación entre módulos genera acoplamiento y bugs cruzados. |
| Opciones evaluadas | Llamadas directas; bus global sin contratos; mediator/bus tipado con scopes. |
| Decisión | Bus/mediator tipado con contratos, scopes y trazabilidad. |
| Consecuencias | Requiere catálogo/versionado de eventos y disciplina de scopes. |
| Riesgos aceptados | Posible complejidad inicial; riesgo de abuso si no hay gobernanza. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Bugs cross-feature | ↓ vs baseline | Crítico si no baja | Estabilidad |
| Tiempo de refactor | ↓ con contratos claros | Alerta si igual | Velocidad |
| Latencia de eventos | p95 estable | Warning si sube | UX consistente |
| Incidentes por payload roto | 0 | Crítico si >0 | Calidad |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-event-bus"></a>Event Bus | Canal para publicar/suscribirse a eventos. |
| <a id="term-mediator"></a>Mediator | Componente que coordina comunicaciones entre módulos. |
| <a id="term-evento-tipado"></a>Evento tipado | Evento con estructura y tipos definidos. |
| <a id="term-scope-de-modulo"></a>Scope de módulo | Ámbito de dependencias/eventos dentro de un feature. |
| <a id="term-tracing"></a>Tracing | Seguimiento de eventos para depuración. |

---

## Referencias

- [Mediator Pattern](https://refactoring.guru/design-patterns/mediator)
- [Event-driven Architecture](https://martinfowler.com/articles/201701-event-driven.html)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
