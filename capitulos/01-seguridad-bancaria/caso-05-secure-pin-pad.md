# Caso 5: Keyloggers en el Teclado Virtual
## Protegiendo PINs en Apps de Banca Móvil

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | PIN seguro, teclado virtual, keylogger, entrada segura, banca móvil, scramble pad |
| **Patrón Técnico** | Secure Input, Custom Keyboard, Screen Recording Protection |
| **Stack Seleccionado** | Custom Widget (SecurePinPad) + Platform Channels (FLAG_SECURE) + Provider (PinEntryProvider) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero ingresar mi PIN de 6 dígitos para autorizar transferencias, con la certeza de que ningún malware puede capturarlo."*

El problema: Teclados de terceros pueden enviar datos a servidores externos. Malware con permisos de accesibilidad puede leer inputs. Screen recording captura posición de toques.

### Evidencia de Industria

**Caso ai.type Keyboard (2017):** Teclado que filtró datos de 31 millones de usuarios.

**EventBot Malware (2020):** Troyano que abusaba accesibilidad para capturar credenciales de 200+ apps bancarias.

**Estadística Kaspersky:** 29% del malware bancario usa keylogging o screen capture.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | PIN comprometido = cuenta comprometida |
| **Compliance** | PCI-DSS requiere protección de datos de autenticación |
| **Reputacional** | Crisis de PR si malware captura PINs |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Rol | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:----|:-----------------------|:----------------------------------|
| **Junior** | `TextField` con `obscureText: true` | **FALLA:** Teclado de terceros captura. Screen recording ve posiciones. Accessibility services leen campo. |
| **Senior** | Custom `PinPad` con botones propios | **MEJORA:** Evita teclados terceros. PERO: posición fija permite inferir PIN por recording. |
| **Architect** | **SecurePinPad:** Scramble, FLAG_SECURE, detección overlay, timeout | **ENTERPRISE:** Resistente a keyloggers, capture, overlays. |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Evitar teclados terceros. Aleatorizar posición de dígitos. Prevenir screenshots/recording. Detectar overlays. Timeout de entrada. |
| **Restricciones Duras (NO permite)** | **Accesibilidad maliciosa:** Con permiso observe text puede leer PIN. **Root:** Atacante puede hookear cualquier función. **FLAG_SECURE:** No funciona en algunas ROMs. |
| **Criterio de Selección** | Se usa **Provider simple** porque lógica es mínima (acumular dígitos). Custom Widget es necesario porque ningún package combina todas las características. |

---

## 4. Manos a la Obra: Estrategia de Implementación

### Fase 1: Diseño

**Características SecurePinPad:**
1. Grid 4x3 con dígitos scrambled
2. FLAG_SECURE activo
3. Detección de overlays
4. Timeout 30 segundos
5. Máximo 3 intentos

### Fase 2: Implementación

**SecureFlagHandler (Platform Channel):**
- Android: `window.setFlags(FLAG_SECURE, FLAG_SECURE)`
- Activar en `initState`, desactivar en `dispose`

**DigitScrambler:**
- Permutación aleatoria de [0-9]
- Usar `Random.secure()`
- Regenerar en cada build

**OverlayDetector (Android):**
- Verificar `canDrawOverlays` de otras apps
- Si overlay sospechoso → no mostrar PIN pad

**SecurePinPad Widget:**
- `StatefulWidget` con estado local
- `GridView.count(crossAxisCount: 3)`
- Solo dots (●) para PIN ingresado
- Feedback háptico sin visual del dígito

---

## 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

```
[ ] TAC-5.1: PIN DEBE ingresarse con teclado custom, NUNCA teclado sistema.
[ ] TAC-5.2: Posiciones de dígitos DEBEN aleatorizarse en cada aparición.
[ ] TAC-5.3: FLAG_SECURE DEBE estar activo (Android).
[ ] TAC-5.4: DEBE detectar overlays y NO mostrar PIN pad si existen.
[ ] TAC-5.5: PIN NUNCA debe mostrarse, solo dots (●).
[ ] TAC-5.6: Timeout de 30 segundos de inactividad.
[ ] TAC-5.7: Bloqueo tras 3 intentos fallidos por 5 minutos.
[ ] TAC-5.8: PIN NUNCA en logs, ni en debug.
[ ] TAC-5.9: Feedback háptico sin feedback visual del valor.
[ ] TAC-5.10: PIN transmitido hasheado con salt, NUNCA texto plano.
```

---

## 6. Estrategia de Pruebas (Shift-Left)

| # | Escenario | Qué Validar | Tipo |
|:-:|:----------|:------------|:-----|
| 1 | **Screenshot durante PIN** | Debe mostrar pantalla negra o bloquearse | Security |
| 2 | **App con overlay activo** | PIN pad NO debe mostrarse | Integration |
| 3 | **Timeout** | Tras 30 seg sin input, PIN se limpia | Widget Test |

---

*Anterior: [Root Detection](caso-04-root-jailbreak-detection.md) | Siguiente: [Session Hijacking](caso-06-session-hijacking-wifi.md)*
