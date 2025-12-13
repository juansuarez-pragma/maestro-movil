# Caso 69: Widgets y Extensiones Seguras
## Compartir Datos entre App y [Widget](#term-widget "Superficie UI del OS fuera de la app principal.") sin Filtrar Información

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | widgets, extensiones, lockscreen, privacidad, performance |
| **Patrón Técnico** | App Extensions, Shared Container, Least Privilege |
| **Stack Seleccionado** | Flutter + widgets nativos (iOS/Android) + storage compartido controlado + feature flags |
| **Nivel de Criticidad** | Medio-Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Widgets muestran datos sensibles en pantalla bloqueada si no se controla privacidad.
- Compartición de datos con storage compartido sin cifrado permite filtraciones.
- Sin límites de refresh, widgets afectan batería y rendimiento.

### Escenario de Negocio

> *"Como usuario, quiero ver información útil (saldo resumido/estado) en un widget sin comprometer mi privacidad."*

### Incidentes reportados
- **Leaks por widgets:** datos visibles en lockscreen generaron reclamos y riesgos regulatorios.
- **Batería/perf:** widgets con refresh agresivo degradaron experiencia.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Apple WidgetKit | Global | Widgets deben ser ligeros y respetar privacidad. |
| Android App Widgets | Global | Updates deben ser controlados; datos sensibles con cuidado. |
| OWASP MAS | Global | [Minimización](#term-minimizacion "Guardar/mostrar solo lo estrictamente necesario.") y storage seguro para datos compartidos. |

**Resumen global**
- Widgets deben ser “read-only” y mínimos: datos no sensibles o derivados, storage compartido cifrado, refresh controlado.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Privacidad** | Exposición de información en lockscreen |
| **Seguridad** | Storage compartido sin cifrado |
| **UX/Performance** | Batería/latencia por refresh excesivo |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Widget con datos sensibles y storage compartido plano | **INADECUADO:** Filtración y riesgo regulatorio. |
| **ACEPTABLE** | Widget con datos minimizados, sin políticas claras | **MEJORA:** Menos riesgo, pero aún puede degradar perf o mostrar datos indebidos. |
| **ENTERPRISE** | **Widgets seguros:** minimización, cifrado en contenedor compartido, política de lockscreen, refresh controlado, flags para apagar | **ÓPTIMO:** Privacidad y estabilidad. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Mostrar datos resumidos (sin PII). Actualizar por señales (no polling). Controlar visibilidad en lockscreen. Cache seguro de “snapshot” mínimo. |
| **Restricciones Duras (NO permite)** | **Background limitado:** OS controla refresh. **Sensibilidad:** widgets no deben exponer datos protegidos. **Complejidad Flutter:** UI del widget no es Flutter, requiere nativo. |
| **Criterio de Selección** | Widget con datos no sensibles; cifrado; refresh controlado; kill switch por flags. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Privacidad | No exponer PII/secretos en widget | Seguridad/QA |
| Performance | Impacto batería/refresh en dispositivos | QA/Perf |
| Integración | Storage compartido cifrado y lectura consistente | Móvil/QA |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Contenido | Mostrar estados y resúmenes, no detalles | Privacidad |
| Bloqueado | “Ocultar contenido sensible” por defecto | Confianza |
| Control remoto | [Kill switch](#term-kill-switch "Apagado remoto ante incidentes.") del widget | Contención |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Cifrado | Datos compartidos cifrados en contenedor | Seguridad |
| Refresh | Evitar polling; usar triggers | Batería |
| Auditoría | Eventos `widget.*` (render, error, refresh) | Observabilidad |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Widgets pueden filtrar datos y degradar batería/perf. |
| Opciones evaluadas | Datos sensibles; minimización sin políticas; widgets seguros con cifrado/lockscreen/refresh controlado. |
| Decisión | Widgets con minimización + cifrado + políticas de lockscreen + refresh controlado + kill switch. |
| Consecuencias | Desarrollo nativo adicional; disciplina de contenido y seguridad. |
| Riesgos aceptados | Limitaciones de refresh del OS; cobertura parcial de datos. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Exposición de datos | 0 incidentes | Crítico si >0 | Privacidad |
| Crash/errores widget | Tendencia a la baja | Warning si sube | Estabilidad |
| Consumo batería | Dentro de budget | Alerta si sube | UX |
| Uso del widget | Medido por cohorte | Alerta si cae | Valor |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-widget"></a>Widget | Superficie UI del OS fuera de la app principal. |
| <a id="term-contenedor-compartido"></a>Contenedor compartido | Espacio compartido app↔widget (App Group / shared prefs). |
| <a id="term-minimizacion"></a>Minimización | Guardar/mostrar solo lo estrictamente necesario. |
| <a id="term-lockscreen"></a>Lockscreen | Pantalla bloqueada; requiere políticas de visibilidad. |
| <a id="term-kill-switch"></a>Kill switch | Apagado remoto ante incidentes. |

---

## Referencias

- [Apple WidgetKit](https://developer.apple.com/documentation/widgetkit)
- [Android App Widgets](https://developer.android.com/develop/ui/views/appwidgets)
- [OWASP MAS](https://mas.owasp.org/)
