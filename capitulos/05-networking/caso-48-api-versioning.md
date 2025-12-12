# Caso 48: API Versioning Hell
## Soportar 3 Versiones de API en Producción

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | api versioning, compatibilidad, breaking changes, migración |
| **Patrón Técnico** | Backward Compatibility, Adapter Pattern, Feature Flags |
| **Stack Seleccionado** | Flutter + Dio interceptors/version header + adapters por versión + flags |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como app, necesito hablar con v1, v2 y v3 del backend sin romper a los usuarios."*

Versionar APIs sin estrategia lleva a ramas múltiples en cliente y bugs al migrar.

### Evidencia de Industria

- **APIs públicas:** Mantienen compatibilidad por periodos largos; clientes deben adaptarse progresivamente.
- **Migraciones fallidas:** Rompen apps legacy si no se gestionan contratos.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Técnico** | Código duplicado, regresiones al migrar |
| **UX** | Funcionalidades inconsistente según versión |
| **Operacional** | Dificultad de desplegar y probar múltiples ramas |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | `if (version)` dispersos en el código | **INADECUADO:** Rama espagueti, alto riesgo. |
| **ACEPTABLE** | Adapters por endpoint | **MEJORA:** Separa, pero puede multiplicarse sin gobernanza. |
| **ENTERPRISE** | **Capa de compatibilidad:** adapters por módulo, feature flags para activar nueva versión, pruebas contractuales, headers/version en interceptor | **ÓPTIMO:** Controla migración, reduce regresiones. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Encapsular diferencias por versión en adapters. Activar versiones por flag/segmento. Añadir headers/version en un solo interceptor. Pruebas contractuales por versión. |
| **Restricciones Duras (NO permite)** | **Multiplicación de variantes:** Requiere limpieza y sunset plan. **Data shape divergente:** Adapters pueden crecer mucho; planificar deprecación. **Testing complejo:** Necesita matriz de pruebas por versión/flag. |
| **Criterio de Selección** | Adapter pattern por módulo, flags para rollout, interceptor para version headers; plan de deprecación. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Compatibilidad hacia atrás | Mantener clientes antiguos funcionando en versiones nuevas. |
| Adapter | Capa que traduce entre contrato viejo y nuevo. |
| Sunset | Proceso de retirar una versión antigua. |
| Contrato | Estructura y reglas de un endpoint. |
| Feature Flag | Control remoto para habilitar versión/feature. |

---

## Referencias

- [API Versioning Best Practices](https://martinfowler.com/articles/enterpriseREST.html#versioning)
- [Google API Guidelines](https://cloud.google.com/apis/design/versioning)
