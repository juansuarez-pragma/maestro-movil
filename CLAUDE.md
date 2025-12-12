# CLAUDE.md - Contexto del Proyecto para Claude AI

Este archivo proporciona contexto a Claude Code cuando trabaja en este repositorio.

---

## Descripción del Proyecto

**Maestría Empresarial Móvil** es un libro/guía de arquitectura Flutter para aplicaciones de Banca y E-commerce de nivel empresarial. El contenido consiste en **100 casos prácticos** que alimentan una base de conocimiento RAG para un agente de IA "Líder Técnico".

---

## Estructura del Repositorio

```
maestro-movil/
├── README.md                    # Documentación principal
├── CLAUDE.md                    # Este archivo (contexto para Claude)
├── TABLA_DE_CONTENIDOS.md       # Índice de los 100 casos
└── capitulos/
    ├── 01-seguridad-bancaria/   # Casos 1-10 (COMPLETADO)
    ├── 02-gestion-estado/       # Casos 11-20
    ├── 03-rendimiento/          # Casos 21-30
    ├── 04-offline-first/        # Casos 31-40
    ├── 05-networking/           # Casos 41-50
    ├── 06-arquitectura-modular/ # Casos 51-60
    ├── 07-integracion-nativa/   # Casos 61-70
    ├── 08-devops-cicd/          # Casos 71-80
    ├── 09-hardware-iot/         # Casos 81-90
    └── 10-migracion-legacy/     # Casos 91-100
```

---

## Formato Obligatorio de Cada Caso

Cada archivo `caso-XX-*.md` DEBE seguir esta estructura de **6 secciones**:

### Sección 0: Metadata para Indexación (AI-Tags)
```markdown
| Campo | Valor |
|:------|:------|
| **Palabras Clave de Negocio** | tags, separados, por, comas |
| **Patrón Técnico** | Nombre del patrón |
| **Stack Seleccionado** | tecnología1 + tecnología2 + GestorEstado |
| **Nivel de Criticidad** | Alto / Medio / Bajo |
```

### Sección 1: Planteamiento del Problema (El "Trigger")
- **Escenario de Negocio:** Historia de usuario en formato quote
- **Evidencia de Industria:** 2-3 casos reales con fechas y datos
- **Riesgos:** Tabla con tipos (Económico, Regulatorio, Reputacional, etc.)

### Sección 2: Matriz de Soluciones y Selección de Herramientas

| Nivel de Madurez | Solución y Herramienta | Análisis de Decisión (Trade-offs) |
|:-----------------|:-----------------------|:----------------------------------|
| **BAJA** | Solución inadecuada | Por qué falla |
| **ACEPTABLE** | Solución mínima | Cumple pero tiene limitaciones |
| **ENTERPRISE** | Solución óptima | Por qué es la mejor para banca/e-commerce |

**IMPORTANTE:** NO usar "Junior/Senior/Architect". Usar BAJA/ACEPTABLE/ENTERPRISE.

### Sección 3: Profundización: Capacidades, Límites y Restricciones
Tabla con:
- **Capacidades (SÍ permite)**
- **Restricciones Duras (NO permite)**
- **Criterio de Selección** (por qué cada herramienta)

### Sección 4: Manos a la Obra: Estrategia de Implementación

#### REGLA "NO-CODE" (OBLIGATORIO)

En la Sección 4, **ESTÁ PROHIBIDO** escribir bloques de código de implementación (ni Dart, ni Kotlin, ni Swift).

**En su lugar, DEBES usar:**

1. **Tablas de Definición de Componentes y Responsabilidades**
   ```markdown
   | Capa | Componente | Responsabilidad |
   |:-----|:-----------|:----------------|
   | Data | AuthLocalDataSource | Persistencia segura de tokens... |
   ```

2. **Descripciones algorítmicas paso a paso (Lógica de negocio)**
   ```markdown
   ##### Algoritmo: TokenRefreshInterceptor

   **Propósito:** Manejar automáticamente la expiración de tokens...

   **Lógica paso a paso:**
   1. **onRequest:** SI la ruta está en lista de exclusión → continuar...
   2. **onError:** SI código es 401 → intentar refresh...
   ```

3. **Tablas de Contratos de Datos (Atributos y Reglas)**
   ```markdown
   | Atributo | Tipo | Descripción | Reglas de Validación |
   |:---------|:-----|:------------|:---------------------|
   | access_token | String (JWT) | Token de acceso | TTL máximo: 15 min |
   ```

4. **Sugerencias de Diagramas visuales usando el formato:**
   ```markdown
   #### Diagrama Sugerido: [Nombre del Diagrama]

   [DIAGRAMA DE {SECUENCIA|FLUJO|COMPONENTES|ESTADOS}]

   Título: ...
   Participantes: ...
   Flujo Principal: ...
   Flujo Alternativo: ...
   ```

**Objetivo:** Crear una arquitectura de referencia **agnóstica al lenguaje** que pueda ser implementada en cualquier stack tecnológico.

---

**DEBE incluir:**

1. **Justificación del Plan** (OBLIGATORIO)
   - Explicar cómo se deriva el plan del análisis del problema
   - Listar 3-4 puntos conectando problema → solución

2. **Fase 1: Diseño**
   - Tabla de Componentes y Responsabilidades
   - Contrato de Datos
   - Diagrama sugerido de arquitectura

3. **Fase 2: Implementación por Plataforma** (SEPARAR POR PLATAFORMA)
   - **2.1 Flutter (Cross-Platform)** — Tablas de definición, estados, eventos, algoritmos
   - **2.2 Android** — Tablas de configuración, algoritmos específicos
   - **2.3 iOS** — Tablas de configuración, algoritmos específicos

4. **Fase 3: Observability**
   - Métricas
   - Alertas

### Sección 5: Salida para el Agente: Criterios de Aceptación Técnicos (TACs)

**SEPARAR POR PLATAFORMA:**

```markdown
### TACs Flutter (Cross-Platform)
[ ] TAC-X.Y-FLUTTER: Descripción...

### TACs Android
[ ] TAC-X.Y-ANDROID: Descripción...

### TACs iOS
[ ] TAC-X.Y-IOS: Descripción...

### TACs Backend (Referencia)
[ ] TAC-X.Y-BACKEND: Descripción...
```

### Sección 6: Estrategia de Pruebas (Shift-Left)
- Stack de Testing
- Tabla de escenarios con columna **Plataforma**

---

## Convenciones de Nomenclatura

### Archivos
- `caso-XX-nombre-en-kebab-case.md`
- Ejemplo: `caso-01-token-nunca-expira.md`

### TACs
```
TAC-{NumCaso}.{NumTAC}-{PLATAFORMA}: Descripción
```
Ejemplo: `TAC-1.3-FLUTTER`, `TAC-1.7-ANDROID`, `TAC-1.11-IOS`

### Plataformas válidas
- `FLUTTER` - Código Dart cross-platform
- `ANDROID` - Configuración/código nativo Android
- `IOS` - Configuración/código nativo iOS
- `BACKEND` - Referencia para equipo backend

---

## Stack Tecnológico Preferido

### Gestión de Estado (rotar entre casos)
| Tecnología | Cuándo usar |
|:-----------|:------------|
| **BLoC** | Flujos complejos, auditoría de eventos, múltiples estados |
| **Cubit** | Flujos simples sin eventos complejos |
| **Riverpod** | DI moderno, testing con overrides, estado compartido |
| **Provider** | Estados mínimos, casos simples |

### Persistencia
| Tecnología | Cuándo usar |
|:-----------|:------------|
| **flutter_secure_storage** | Tokens, credenciales, datos sensibles |
| **drift** | SQL local, queries complejos, relaciones |
| **Hive/Isar** | NoSQL rápido, caching, datos no relacionales |

### Networking
| Tecnología | Cuándo usar |
|:-----------|:------------|
| **Dio** | HTTP con interceptores, retry, logging |
| **Platform Channels** | SDKs nativos (pagos, biometría, etc.) |

---

## Casos de Industria Referenciales

Al escribir casos, incluir referencias a incidentes reales:

### Seguridad/Banca
- Revolut 2022 (token compromise)
- Capital One 2019 (WAF misconfiguration)
- DigiNotar 2011 (CA compromise)
- Target 2013 (PCI breach)
- Wells Fargo 2016 (insider fraud)

### Estadísticas
- Verizon DBIR (Data Breach Investigation Report)
- Kaspersky Security Reports
- ACFE Report to the Nations (fraud)
- NIST SP 800-63B (authentication guidelines)

### Normativas
- PCI-DSS (pagos)
- PSD2/SCA (Europa)
- GDPR (privacidad)
- SOX (auditoría)

---

## Comandos Git Comunes

```bash
# Ver estado
git status

# Agregar y commit
git add .
git commit -m "feat(capXX): descripción del cambio"

# Push
git push

# Crear rama para nuevo capítulo
git checkout -b feature/capitulo-XX-nombre
```

### Convención de Commits
```
tipo(scope): descripción

Tipos:
- feat: nuevo caso o capítulo
- refactor: cambios de formato/estructura
- fix: correcciones
- docs: documentación
```

---

## Tareas Pendientes del Proyecto

### Completados
- [x] Capítulo 1: Seguridad Bancaria (casos 1-10)

### Pendientes
- [ ] Capítulo 2: Gestión de Estado (casos 11-20)
- [ ] Capítulo 3: Rendimiento (casos 21-30)
- [ ] Capítulo 4: Offline-First (casos 31-40)
- [ ] Capítulo 5: Networking (casos 41-50)
- [ ] Capítulo 6: Arquitectura Modular (casos 51-60)
- [ ] Capítulo 7: Integración Nativa (casos 61-70)
- [ ] Capítulo 8: DevOps/CI-CD (casos 71-80)
- [ ] Capítulo 9: Hardware/IoT (casos 81-90)
- [ ] Capítulo 10: Migración Legacy (casos 91-100)

---

## Instrucciones para Claude

Cuando trabajes en este proyecto:

1. **Al crear nuevos casos:**
   - Seguir EXACTAMENTE el formato de 6 secciones
   - Usar BAJA/ACEPTABLE/ENTERPRISE (no Junior/Senior/Architect)
   - Incluir "Justificación del Plan" en sección 4
   - Separar implementación por plataforma (Flutter, Android, iOS)
   - Separar TACs por plataforma
   - Incluir 2-3 referencias de industria reales

2. **Al modificar casos existentes:**
   - Mantener la estructura de 6 secciones
   - No mezclar configuraciones de diferentes plataformas
   - Verificar que TACs tengan el formato correcto

3. **Al hacer commits:**
   - Usar conventional commits
   - Incluir scope del capítulo afectado
   - Push al finalizar cambios

4. **Rotación de stack:**
   - Variar gestores de estado entre casos (BLoC, Cubit, Riverpod, Provider)
   - Usar diferentes tecnologías de persistencia según el caso
   - No repetir el mismo stack en casos consecutivos

---

## Contacto

**Repositorio:** `git@github.com:juansuarez-pragma/maestro-movil.git`
**Organización:** Pragma
**Última actualización:** Diciembre 2024
