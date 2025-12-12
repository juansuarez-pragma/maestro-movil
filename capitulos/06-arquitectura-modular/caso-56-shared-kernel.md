# Caso 56: Shared Kernel
## Extraer Código Común sin Crear un Monolito

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | shared kernel, código común, módulos, dependencia cruzada |
| **Patrón Técnico** | Shared Kernel, Domain Library, Semantic Versioning |
| **Stack Seleccionado** | Flutter + packages internos (monorepo) + Melos + semver |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo, necesitamos compartir modelos/contratos sin acoplar todos los módulos."*

Sin un kernel común bien gobernado, se duplican modelos y se generan dependencias cíclicas o un monolito disfrazado.

### Evidencia de Industria

- **DDD:** Shared Kernel debe ser pequeño y estable; cambios controlados.
- **Monorepos grandes:** Requieren versionado y gobernanza para evitar ripple effects.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Técnico** | Acoplamiento, actualizaciones masivas, regresiones |
| **Productivo** | Velocidad de desarrollo se frena por dependencias frágiles |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Todo compartido en un módulo gigante | **INADECUADO:** Monolito, sin límites claros. |
| **ACEPTABLE** | Librería común sin gobernanza | **MEJORA:** Comparte código, pero riesgo de roturas frecuentes. |
| **ENTERPRISE** | **Shared kernel acotado:** contratos y modelos estables, semver, revisiones de cambio, tests contractuales | **ÓPTIMO:** Reuso sin acoplamiento excesivo, cambios previsibles. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Definir modelos/contratos comunes estables. Publicar como package interno versionado. Ejecutar tests contractuales entre módulos. Minimizar dependencias hacia el kernel. |
| **Restricciones Duras (NO permite)** | **Kernel gigante:** Debe ser pequeño. **Cambios frecuentes:** Aumentan rompimientos; requiere governance. **Dependencias cíclicas:** Prohibidas; revisar gráfos de dependencias. |
| **Criterio de Selección** | Kernel con semver, revisión de cambios, tests contractuales; uso de melos/monorepo para manejo de versiones y releases. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Shared Kernel | Subconjunto de modelo/contratos compartidos y estables. |
| Semver | Versionado semántico para comunicar compatibilidad. |
| Contract Testing | Validar que consumidores y proveedor coinciden en contratos. |
| Monorepo | Repositorio único que aloja múltiples paquetes. |
| Melos | Herramienta para gestionar monorepos de Dart/Flutter. |

---

## Referencias

- [Domain-Driven Design - Shared Kernel](https://www.domainlanguage.com/ddd/)
- [Semantic Versioning](https://semver.org/)
- [Melos](https://melos.invertase.dev/)
