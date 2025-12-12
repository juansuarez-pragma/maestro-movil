# Caso 41: BFF — El Guardián del Móvil
## Orquestar 7 Microservicios en Una Llamada

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | BFF, orquestación, microservicios, latencia, agregación |
| **Patrón Técnico** | Backend for Frontend (BFF), Aggregation, Response Shaping |
| **Stack Seleccionado** | Flutter + Dio + BFF REST/GraphQL + Riverpod para cache |
| **Nivel de Criticidad** | Crítico |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario móvil, quiero ver mi dashboard sin esperar a 7 llamadas secuenciales."*

El frontend móvil sufre latencia y complejidad al hablar con múltiples servicios; el BFF reduce roundtrips y adapta payloads.

### Evidencia de Industria

- **Netflix/Shopify:** BFFs móviles reducen latencia y payloads para dispositivos limitados.
- **Banca:** Dashboards con múltiples agregaciones requieren una capa dedicada.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Pantallas lentas, timeouts |
| **Técnico** | Lógica duplicada en cliente, mayor complejidad y bugs |
| **Reputacional/Económico** | Menor conversión si el dashboard tarda |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Cliente orquesta múltiples APIs directamente | **INADECUADO:** Latencia alta, acoplamiento, cambios frecuentes. |
| **ACEPTABLE** | Proxy simple sin agregación | **MEJORA:** Un endpoint, pero sin reducir payload ni complejidad. |
| **ENTERPRISE** | **BFF completo:** agregación, cache server-side, response shaping, contratos específicos por pantalla | **ÓPTIMO:** Menos roundtrips, contratos estables, menor lógica en cliente. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Reducir llamadas a una sola por pantalla. Adaptar payloads al modelo de UI. Aplicar cache server-side y e-tags. Versionar contratos sin romper cliente. |
| **Restricciones Duras (NO permite)** | **Sin BFF evolutivo:** Cambios de dominio requieren actualizar BFF. **Escalabilidad:** BFF puede ser cuello de botella sin autoscaling. **Consistencia:** Datos pueden estar stale si cache no se invalida bien. |
| **Criterio de Selección** | BFF para reducir latencia y acoplamiento; GraphQL opcional para response shaping; Riverpod para cache local y estados claros. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| BFF | Backend for Frontend; capa dedicada para un cliente (móvil/web). |
| Aggregation | Combinar respuestas de múltiples servicios en una sola. |
| Response Shaping | Ajustar payloads a las necesidades del cliente. |
| E-Tag | Hash de recurso para validar/inutilizar cache. |
| Latencia percibida | Tiempo que el usuario percibe antes de ver datos útiles. |

---

## Referencias

- [Pattern: Backend for Frontend](https://samnewman.io/patterns/architectural/bff/)
- [Netflix BFF Lessons](https://netflixtechblog.com/)
