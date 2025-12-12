# Caso 55: Comunicación entre Features
## Event Bus vs BLoC-to-BLoC

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | comunicación interna, módulos, eventos, acoplamiento |
| **Patrón Técnico** | Event Bus, Mediator, Shared State Contracts |
| **Stack Seleccionado** | Flutter + Riverpod scoped providers + Event channels internos |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo modular, necesito que features se comuniquen sin acoplarse y sin duplicar lógica."*

Comunicación ad-hoc genera dependencias circulares y estados inconsistentes.

### Evidencia de Industria

- **Micro-frontends:** Usan event bus/mediator para evitar acoplamiento directo.
- **Apps grandes:** Contratos claros de eventos reducen regresiones.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Técnico** | Acoplamiento, cascadas de cambios, bugs difíciles de trazar |
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
| **Restricciones Duras (NO permite)** | **Eventos sin contrato:** Riesgo de drift si no se tipan. **Abuso de globales:** Bus global puede volverse dumping ground; requiere gobernanza. **Orden garantizado:** Puede requerir colas si se necesita orden total. |
| **Criterio de Selección** | Bus/mediator tipado para desacople; Riverpod scoped providers para inyección de handlers; contratos y versionado de eventos. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Event Bus | Canal para publicar/suscribirse a eventos. |
| Mediator | Componente que coordina comunicaciones entre módulos. |
| Evento tipado | Evento con estructura y tipos definidos. |
| Scope de módulo | Ámbito de dependencias/eventos dentro de un feature. |
| Tracing | Seguimiento de eventos para depuración. |

---

## Referencias

- [Mediator Pattern](https://refactoring.guru/design-patterns/mediator)
- [Event-driven Architecture](https://martinfowler.com/articles/201701-event-driven.html)
