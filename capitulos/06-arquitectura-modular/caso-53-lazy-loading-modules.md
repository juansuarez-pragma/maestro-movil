# Caso 53: El Módulo que Pesaba 40MB
## Lazy Loading de Features por Demanda

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | lazy loading, módulos, tamaño de app, on-demand |
| **Patrón Técnico** | Dynamic Feature Loading, Code Splitting, Deferred Components |
| **Stack Seleccionado** | Flutter + deferred imports + Android Dynamic Features + Riverpod para gating |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como equipo, quiero reducir el tamaño inicial descargando features pesadas solo cuando se usan."*

Features raras pero pesadas inflan el bundle y afectan instalaciones y arranque.

### Evidencia de Industria

- **Apps con módulos KYC/Video:** Se cargan bajo demanda para no inflar el core.
- **Android App Bundle:** Soporta Dynamic Feature para reducir descargas iniciales.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Descarga tardía si no se anticipa; puede frustrar al usuario |
| **Técnico** | Complejidad en gating y fallback; errores de carga en red pobre |
| **Reputacional/Económico** | Instalaciones fallidas por tamaño inicial elevado |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Todo en el bundle principal | **INADECUADO:** Tamaño grande, arranque más lento. |
| **ACEPTABLE** | Eliminar assets/libros no usados | **MEJORA:** Reduce algo, pero no evita módulos pesados en core. |
| **ENTERPRISE** | **Carga diferida:** deferred imports + Dynamic Features (Android) / segmentar assets, gating y precarga condicional | **ÓPTIMO:** Tamaño inicial menor, arranque rápido, flexibilidad. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Separar features pesadas (KYC, video) en módulos on-demand. Descargar anticipadamente según señales (navegación, perfil). Mostrar progreso y fallback. Reducir tamaño por ABI (split). |
| **Restricciones Duras (NO permite)** | **iOS:** No hay equivalente directo a Dynamic Feature; se limita a assets en tiempo de compilación. **Red pobre:** Descarga on-demand puede fallar; necesita fallback. **Reflexión:** Deferred puede romper si se usa reflexión/dynamic. |
| **Criterio de Selección** | Deferred para código Dart; Dynamic Feature para nativo/recursos; gating via Riverpod/feature flags; precarga cuando hay buena red/carga. |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Deferred Import | Carga de librería Dart bajo demanda. |
| Dynamic Feature | Módulo descargable de Android App Bundle. |
| Gate/Flag | Condición que habilita la carga de un módulo. |
| Preload | Descargar anticipadamente basado en señal de uso probable. |
| Split per ABI | Generar artefactos por arquitectura para reducir tamaño. |

---

## Referencias

- [Deferred Components](https://docs.flutter.dev/development/ui/advanced/deferred-components)
- [Android Dynamic Features](https://developer.android.com/guide/app-bundle/dynamic-delivery)
