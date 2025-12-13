# Caso 67: Deep Links Seguros
## Universal Links / App Links sin Phishing ni Hijacking

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | deep links, onboarding, phishing, hijacking, recuperación de cuenta |
| **Patrón Técnico** | Universal Links / App Links, Link Hardening, Safe Navigation |
| **Stack Seleccionado** | Flutter + enlace universal (iOS) + app links (Android) + validación server-side + feature flags |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Deep links sin validación permiten **phishing** (links falsos) o **hijacking** (otra app captura el enlace).
- Parámetros sin firma/expiración habilitan replay en flujos sensibles (reset password, activación, KYC).
- Sin trazabilidad, es difícil auditar el origen del link y detectar abusos.

### Escenario de Negocio

> *"Como usuario, quiero abrir enlaces de activación y recuperación directamente en la app, sin riesgo de fraude."*

### Incidentes reportados
- **Account takeover:** enlaces de recuperación reutilizados o interceptados por apps maliciosas.
- **Hijacking de intent:** apps no autorizadas capturaron links cuando no había asociación correcta.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| Guías Android App Links | Global | La asociación de dominio evita resoluciones ambiguas y reduce hijacking. |
| Guías iOS Universal Links | Global | Asociaciones mal configuradas degradan a Safari y elevan phishing. |
| OWASP MAS | Global | Validación de entrada y “secure deep links” reducen abuso de intents/URLs. |

**Resumen global**
- Deep links deben tratarse como entrada no confiable: asociación de dominio + tokens firmados y con expiración + validación en backend.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Seguridad** | Phishing, account takeover, replay de enlaces |
| **UX** | Flujos rotos (abrir en Safari), soporte elevado |
| **Operacional** | Dificultad para investigar incidentes sin trazabilidad |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Deep links custom scheme sin validación | **INADECUADO:** Hijacking fácil, phishing y replay. |
| **ACEPTABLE** | Universal Links/App Links sin hardening | **MEJORA:** Mejor asociación, pero aún vulnerable si el payload no expira/firma. |
| **ENTERPRISE** | **Deep links hardened:** asociación de dominio, tokens firmados (one-time + expiración), validación server-side, allowlist de rutas, auditoría y métricas | **ÓPTIMO:** Reduce phishing/replay y mejora trazabilidad. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Abrir flujos sensibles con link verificado. Requerir token de un solo uso y expiración. Redirigir a rutas seguras internas. Medir éxito/errores por campaña/feature. |
| **Restricciones Duras (NO permite)** | **Entornos corporativos:** proxies/navegadores pueden alterar comportamiento. **Compatibilidad:** algunas apps/OS degradan universal links. **UX:** Demasiada fricción en validación puede aumentar abandono. |
| **Criterio de Selección** | Preferir Universal Links/App Links, firmar y expirar payload, validar en backend y registrar trazas. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Seguridad | Token one-time, expiración y firma | Seguridad/QA |
| Integration (Android/iOS) | Asociación de dominio y fallback | Móvil/QA |
| Observabilidad | Eventos `deeplink.*` con origen/ruta/resultado | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Fallback | Si app no está instalada, abrir web segura | Continuidad |
| Mensajes | Errores claros (link vencido) y CTA a solicitar uno nuevo | Reduce soporte |
| Anti-phishing | Mostrar dominio/branding y evitar redirecciones externas | Confianza |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Allowlist | Rutas permitidas explícitas | Evita abuso |
| Rotación | Rotar llaves de firma y versionar payload | Seguridad |
| Auditoría | Log de consumo de link (one-time) | Investigación |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Phishing/hijacking/replay mediante deep links. |
| Opciones evaluadas | Custom scheme; universal/app links sin hardening; deep links hardened con firma/expiración y validación. |
| Decisión | Universal Links/App Links + token firmado one-time con expiración + validación backend + allowlist. |
| Consecuencias | Requiere coordinación con backend y observabilidad; mantenimiento de asociación de dominio. |
| Riesgos aceptados | Degradación en algunos clientes; fricción cuando links expiran. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Links inválidos/replay | 0 consumos válidos | Crítico si >0 | Seguridad |
| Éxito de apertura | > 98% | Warning si baja | UX |
| Abandono en recuperación | ↓ vs baseline | Alerta si no baja | Conversión |
| Incidentes de phishing | ↓ | Crítico si sube | Riesgo |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| Universal Links | Enlaces iOS asociados a dominio que abren la app de forma segura. |
| App Links | Enlaces Android verificados (assetlinks) para abrir app sin chooser. |
| Hijacking | Captura del enlace por una app no autorizada. |
| One-time token | Token consumible una sola vez para evitar replay. |
| Allowlist | Lista explícita de rutas/acciones permitidas. |

---

## Referencias

- [Android App Links](https://developer.android.com/training/app-links)
- [Apple Universal Links](https://developer.apple.com/documentation/xcode/supporting-universal-links-in-your-app)
- [OWASP MAS](https://mas.owasp.org/)
