# Caso 6: Session Hijacking en WiFi Público
## El Caso del Café que Vació Cuentas

---

## 0. Metadata para Indexación (AI-Tags)

| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | session hijacking, WiFi público, red insegura, sidejacking, seguridad de red |
| **Patrón Técnico** | Network Security Detection, Session Binding, Additional Verification on Untrusted Networks |
| **Stack Seleccionado** | connectivity_plus + Platform Channels (Network Security) + BLoC (NetworkSecurityBloc) |
| **Nivel de Criticidad** | Alto |

---

## 1. Planteamiento del Problema (El "Trigger")

### Escenario de Negocio

> *"Como usuario, quiero poder revisar mi cuenta bancaria desde el WiFi de la cafetería mientras espero mi café."*

Redes WiFi públicas son terreno fértil para ataques. ARP spoofing, rogue access points, SSL stripping.

### Evidencia de Industria

**DarkHotel APT:** Grupo que comprometía WiFi de hoteles para atacar ejecutivos.

**Estudio Kaspersky 2023:** 25% de WiFi públicos sin cifrado, 25% con cifrado débil.

### Riesgos

| Tipo | Impacto |
|:-----|:--------|
| **Económico** | Session tokens robados permiten ATO |
| **Reputacional** | "Usuario robado en Starbucks" |
| **Legal** | Banco demandado por no advertir |

---

## 2. Matriz de Soluciones y Selección de Herramientas

| Rol | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:----|:-----------------------|:----------------------------------|
| **Junior** | Confiar en HTTPS | **FALLA:** SSL stripping, proxies maliciosos |
| **Senior** | Detectar WiFi público + warning | **MEJORA:** Usuario informado. PERO: muchos ignoran |
| **Architect** | **Defensa profunda:** Detectar red, re-verificación, session binding a red, VPN sugerida | **ENTERPRISE:** Protección activa, no solo advertencia |

---

## 3. Profundización: Capacidades, Límites y Restricciones

| Dimensión | Detalle Técnico |
|:----------|:----------------|
| **Capacidades (SÍ permite)** | Detectar tipo de red. Identificar WiFi segura vs insegura. Step-up auth en redes no confiables. Vincular session a IP/network. |
| **Restricciones Duras (NO permite)** | **Certeza absoluta:** No garantiza que red privada no esté comprometida. **VPN detection:** Complejo. **iOS:** APIs más restrictivas. |
| **Criterio de Selección** | Se usa **BLoC** porque estado de red cambia dinámicamente y múltiples features reaccionan. |

---

## 4. Manos a la Obra: Estrategia de Implementación

### Fase 1: Modelo de Confianza

**Niveles:**
- `trusted`: Celular, WiFi empresarial, VPN
- `moderate`: WiFi doméstico WPA2/3
- `untrusted`: WiFi público, red abierta
- `hostile`: Red en blacklist

**Matriz de Acciones:**

| Operación | trusted | moderate | untrusted | hostile |
|:----------|:--------|:---------|:----------|:--------|
| Ver saldo | ✓ | ✓ | ✓ | ⚠️ |
| Transferir < $100 | ✓ | ✓ | ⚠️ Re-auth | ✗ |
| Transferir > $1000 | ✓ | ⚠️ OTP | ✗ | ✗ |

### Fase 2: Implementación

**NetworkSecurityBloc:**
- Escuchar cambios con `connectivity_plus`
- Evaluar nivel de confianza en cada cambio
- Estados: `NetworkTrusted`, `NetworkModerate`, `NetworkUntrusted`, `NetworkHostile`

**NetworkTrustEvaluator:**
- Tipo conexión (WiFi, Cellular)
- Si WiFi: obtener SSID, BSSID, security type
- Verificar contra whitelist
- Verificar VPN activa

**SessionNetworkBinding:**
- Guardar fingerprint de red al crear session
- Verificar en cada request
- Cambio de país en segundos → invalidar

---

## 5. Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

```
[ ] TAC-6.1: DEBE detectar y clasificar nivel de confianza de red.
[ ] TAC-6.2: En untrusted, operaciones > $100 DEBEN requerir auth adicional.
[ ] TAC-6.3: En hostile, BLOQUEAR operaciones financieras.
[ ] TAC-6.4: Usuario DEBE ser informado en red no confiable.
[ ] TAC-6.5: DEBE existir session binding a contexto de red.
[ ] TAC-6.6: Cambio drástico de contexto DEBE invalidar sesión.
[ ] TAC-6.7: DEBE sugerir VPN en red no confiable.
[ ] TAC-6.8: Certificate pinning activo independiente de red.
[ ] TAC-6.9: Usuarios pueden configurar whitelist de SSIDs.
[ ] TAC-6.10: Telemetría de operaciones en redes no confiables.
```

---

## 6. Estrategia de Pruebas (Shift-Left)

| # | Escenario | Qué Validar | Tipo |
|:-:|:----------|:------------|:-----|
| 1 | **WiFi abierto** | Detecta untrusted, warning, step-up | Integration |
| 2 | **Cambio de red** | Session re-evalúa, notifica si baja confianza | Integration |
| 3 | **Cambio país abrupto** | Session invalidada o re-auth | Integration |

---

*Anterior: [Secure PIN Pad](caso-05-secure-pin-pad.md) | Siguiente: [Remember Me](caso-07-remember-me-seguro.md)*
