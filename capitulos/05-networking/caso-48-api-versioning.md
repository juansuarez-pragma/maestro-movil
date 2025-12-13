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

### Problema detectado (técnico)
- `if (version)` dispersos generan rama espagueti y regresiones.
- Sin adapters/flags, la migración entre v1/v2/v3 rompe clientes legacy.
- Testing explota sin matriz por versión; falta sunset plan acumula deuda.

### Escenario de Negocio

> *"Como app, necesito hablar con v1, v2 y v3 del backend sin romper a los usuarios."*

### Incidentes reportados
- **APIs públicas:** Mantienen compatibilidad por periodos largos; clientes deben adaptarse progresivamente.
- **Migraciones fallidas:** Rompen apps legacy si no se gestionan contratos.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| APIs públicas | Global | Compatibilidad prolongada; adapters/flags para transición. |
| Migraciones fallidas | Varios | Bugs y outages al no aislar contratos. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; versioning/contratos es foco de bugs. |

**Resumen global**
- Versionar sin capa de compatibilidad provoca regresiones y código duplicado; adapters + flags + pruebas contractuales son esenciales.

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

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Contract tests | Adapters cumplen contratos v1/v2/v3 | QA/Backend |
| Integration (CI) | Headers/version correctos por flag/segmento | Móvil/CI |
| Observabilidad | Eventos `api.version` con ruta/versión | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Rollout | Flags para habilitar v2/v3 por cohorte | Control de riesgo |
| Sunset | Plan de retiro de versiones viejas | Reduce deuda |
| Errores | Manejar fallback si versión no soportada | Resiliencia |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Gobernanza | Catálogo de endpoints por versión; limpieza periódica | Evita rama espagueti |
| Testing | Matriz por versión/flag | Previene regresiones |
| Deprecación | Comunicar y retirar versiones | Salud del sistema |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Soportar v1/v2/v3 sin romper clientes ni multiplicar ramas. |
| Opciones evaluadas | `if` disperso; adapters parciales; capa de compatibilidad + flags + tests contractuales. |
| Decisión | Adapters por módulo + interceptor de versión + flags + plan de sunset. |
| Consecuencias | Mayor disciplina en testing/gobernanza; overhead de mantener múltiples versiones. |
| Riesgos aceptados | Complejidad de pruebas; deuda si no se ejecuta sunset. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Regresiones por versión | ↓ vs baseline | Alerta si no baja | Estabilidad |
| Tiempo de migración | Controlado con flags | Alerta si se extiende | Riesgo acotado |
| Deuda de versiones | Sunset ejecutado según plan | Alerta si se acumula | Código sano |
| Tickets por versión | ↓ vs baseline | Alerta si no baja | Soporte menor |

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
