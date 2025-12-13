# Caso 87: Fingerprint [Fallback](#term-fallback "Alternativa cuando el método principal falla.")
## Qué Hacer Cuando el Sensor Falla

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | fingerprint, fallback, biometría, accesibilidad |
| **Patrón Técnico** | Biometric Fallback, Risk-Based [Step-up](#term-step-up "Aumentar seguridad según riesgo."), Graceful Degradation |
| **Stack Seleccionado** | Flutter + local_auth + policies de fallback ([PIN](#term-pin "Código personal; debe tener políticas de seguridad.")/OTP) + Riverpod estado |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Problema detectado (técnico)
- Sensores defectuosos o dedos no reconocidos bloquean al usuario.
- Fallback débil (PIN simple/OTP sin límites) abre vectores de ataque.
- Sin logging de fallos, no se detectan patrones de fraude o UX mala.

### Escenario de Negocio

> *"Como usuario, si la huella falla, necesito otra opción segura para no quedar bloqueado."*

### Incidentes reportados
- **Apps bancarias:** Requieren fallback a PIN/OTP con controles; usuarios bloqueados aumentan churn.
- **Accesibilidad:** Algunos usuarios no pueden usar biometría; sin fallback se pierden.

### Analítica y prevalencia (industria)

| Fuente | Muestra / Región | Hallazgos relevantes |
|:-------|:-----------------|:---------------------|
| OWASP MASVS | Global | Fallback seguro es requisito para autenticación robusta. |
| Casos de soporte | Varios | Bloqueos por biometría fallida elevan tickets. |
| NowSecure 2024 | 1,000+ apps móviles | 85% fallan ≥1 control MASVS; autenticación/fallback mal gestionado es común. |

**Resumen global**
- Fallback gobernado con límites y step-up equilibra seguridad/UX; solo biometría o fallback débil genera bloqueos y riesgo.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **UX** | Usuarios bloqueados, churn |
| **Seguridad** | Fallback débil abre vectores de ataque |
| **Operacional** | Aumento de tickets de soporte |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Solo biometría, sin alternativa | **INADECUADO:** Bloqueos y mala UX. |
| **ACEPTABLE** | Fallback a PIN/OTP sin políticas | **MEJORA:** Evita bloqueo, pero puede ser más débil. |
| **ENTERPRISE** | **Fallback gobernado:** límites de intentos, step-up según riesgo, biometría opcional pero recomendada, auditoría de fallos | **ÓPTIMO:** Balance seguridad/UX con control. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Detectar fallos de biometría y ofrecer PIN/OTP. Limitar intentos y aplicar step-up. Registrar fallos para fraude/UX. Permitir desactivar biometría en dispositivos problemáticos. |
| **Restricciones Duras (NO permite)** | **Biometría obligatoria:** Si regulación exige, fallback puede ser limitado. **PIN débil:** Requiere políticas de longitud/complexidad. **[A11y](#term-a11y "Accesibilidad; cumplimiento para lectores de pantalla."):** Prompts deben ser accesibles. |
| **Criterio de Selección** | Fallback seguro (PIN/OTP) con límites, logging, y step-up; biometría opcional pero recomendada; mensajes claros. |

### 3.1 Plan de verificación (V&V)
| Tipo de verificación | Qué valida | Responsable/Entorno |
|:---------------------|:-----------|:--------------------|
| Seguridad | Políticas de PIN/OTP, límites de intentos | Seguridad/QA |
| UX | Flujo accesible y claro tras fallar biometría | QA/A11y |
| Observabilidad | Eventos `auth.fallback.*` con razón | Móvil/SRE |

### 3.2 UX y operación
| Tema | Política | Nota |
|:-----|:---------|:-----|
| Mensajes | Explicar por qué falló biometría y opciones | Transparencia |
| Step-up | OTP adicional en contextos de riesgo | Seguridad |
| Desactivación | Permitir desactivar bio en dispositivos problemáticos | Flexibilidad |

### 3.3 Operación y riesgo
| Tema | Política | Nota |
|:-----|:--------|:-----|
| Intentos | Bloqueo tras N fallos; cooldown | Previene abuso |
| Auditoría | Log de fallos y cambio de método | Trazabilidad |
| Cumplimiento | Respetar políticas regulatorias/locales | Conformidad |

### 3.4 Mini-ADR (Decisión de Arquitectura)
| Aspecto | Detalle |
|:--------|:--------|
| Problema | Bloqueos por biometría y riesgo de fallback débil. |
| Opciones evaluadas | Solo biometría; fallback simple; fallback gobernado con límites/step-up/logging. |
| Decisión | Fallback gobernado con límites, step-up según riesgo y auditoría. |
| Consecuencias | Requiere políticas claras y UX accesible; manejo de soporte. |
| Riesgos aceptados | Posible fricción por límites; regulación puede restringir fallback. |

---

## 4. Impacto esperado (vista rápida)

| KPI | Objetivo | Umbral/Alerta | Impacto esperado |
|:----|:---------|:--------------|:-----------------|
| Usuarios bloqueados por bio | ↓ vs baseline | Crítico si sube | UX |
| Intentos fallidos | Controlados con límites | Warning si crecen | Seguridad |
| Tickets de soporte | Tendencia a la baja | Alerta si suben | Operación |
| Incidentes por fallback débil | 0 | Crítico si >0 | Riesgo |

---

## Glosario de Términos Clave

<a id="glosario-de-terminos-clave"></a>

| Término | Definición breve |
|:--------|:-----------------|
| <a id="term-fallback"></a>Fallback | Alternativa cuando el método principal falla. |
| <a id="term-step-up"></a>Step-up | Aumentar seguridad según riesgo. |
| <a id="term-intentos-limitados"></a>Intentos limitados | Número máximo antes de bloqueo. |
| <a id="term-a11y"></a>A11y | Accesibilidad; cumplimiento para lectores de pantalla. |
| <a id="term-pin"></a>PIN | Código personal; debe tener políticas de seguridad. |

---

## Referencias

- [local_auth](https://pub.dev/packages/local_auth)
- [OWASP Mobile Auth](https://owasp.org/www-project-mobile-security-testing-guide/)
- [NowSecure - State of Mobile App Security 2024](https://www.nowsecure.com/blog/2024/04/state-of-mobile-app-security-2024/)
