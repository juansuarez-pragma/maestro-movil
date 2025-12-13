# CLAUDE.md - Contexto del Proyecto para Claude AI

Este archivo proporciona contexto a Claude Code cuando trabaja en este repositorio.

---

## Descripción del Proyecto

**Maestría Empresarial Móvil** es un libro/guía de arquitectura Flutter para aplicaciones de Banca y E-commerce de nivel empresarial. El contenido consiste en casos prácticos que alimentan una base de conocimiento RAG para un agente de IA "Líder Técnico".

**Estado actual (repo):**
- Casos publicados: **100** (`capitulos/**/caso-*.md`).
- Capítulo 1 renombrado a `capitulos/01-seguridad/` (antes `01-seguridad-bancaria/`).

---

## Estructura del Repositorio

```
maestro-movil/
├── README.md                    # Documentación principal
├── CLAUDE.md                    # Este archivo (contexto para Claude)
├── TABLA_DE_CONTENIDOS.md       # Índice de los casos publicados
└── capitulos/
    ├── 01-seguridad/            # Casos 1-10 (COMPLETADO)
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

## Formato de Documentación (para RAG)

Cada archivo `caso-XX-*.md` DEBE seguir esta estructura (agnóstica y sin código):

- **Sección 0: Metadata para Indexación (AI-Tags)**  
  Palabras clave de negocio, patrón técnico, stack seleccionado, criticidad.
- **Sección 1: Planteamiento (El "Trigger")**  
  Problema técnico + escenario de negocio (quote), incidentes reportados y analítica/prevalencia (tabla) + riesgos (tabla).
- **Sección 2: Matriz de Soluciones y Selección de Herramientas**  
  BAJA / ACEPTABLE / ENTERPRISE con trade-offs claros.
- **Sección 3: Profundización**  
  Capacidades (SÍ), restricciones (NO), criterio de selección y tablas de V&V/UX/operación/seguridad + mini-ADR (decisión) cuando aplique.
- **Sección 4: Impacto esperado**  
  KPIs/umbrales y resultado de negocio.
- **Glosario de Términos Clave**  
  Tabla con ancla `#glosario-de-terminos-clave`; cada término debe tener su propio `id` (`#term-*`) y los términos del texto deben enlazar a esos anchors para habilitar tooltips.
- **Referencias**  
  3–5 fuentes externas relevantes (incidentes, normas, guías/estudios).

**Regla NO-CODE (se mantiene):** No incluir bloques de código de implementación (Dart/Kotlin/Swift). Usa tablas, descripciones paso a paso, contratos de datos y diagramas ASCII si aplica.

---

## Convenciones de Nomenclatura

### Archivos
- `caso-XX-nombre-en-kebab-case.md`
- Ejemplo: `caso-01-token-nunca-expira.md`

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

## Instrucciones para Claude

Cuando trabajes en este proyecto:

1. **Al crear nuevos casos:**
   - Seguir el formato 0–4 + glosario + referencias (sin código de implementación).
   - Usar BAJA/ACEPTABLE/ENTERPRISE (no Junior/Senior/Architect).
   - Incluir incidentes reportados y riesgos en la sección 1.
   - Usar tablas para capacidades/limitaciones y V&V/operación; mantener KPIs en la sección 4.
   - Anclar cada término del glosario con `id` único (`#term-*`) y enlazarlo desde el cuerpo del caso.

2. **Al modificar casos existentes:**
   - Mantener la estructura descrita y los anchors del glosario.
   - No introducir código de app; priorizar tablas, diagramas ASCII y métricas claras.
   - Validar coherencia de KPIs/umbrales y referencias externas.

3. **Al hacer commits:**
   - Usar conventional commits
   - Incluir scope del capítulo afectado
   - Push al finalizar cambios

4. **Rotación de stack:**
   - Variar gestores de estado entre casos (BLoC, Cubit, Riverpod, Provider).
   - Usar diferentes tecnologías de persistencia según el caso.
   - Evitar repetir el mismo stack en casos consecutivos.

## Nota para Agentes (Claude/Codex)

- Prioriza consistencia de headings (0–4 + glosario + referencias) para maximizar recuperación en RAG.
- No generes código de implementación; el repositorio es documentación agnóstica (tablas, políticas, flujos ASCII, KPIs).
- Antes de afirmar “100 casos”, valida conteo real en `capitulos/**/caso-*.md` y revisa `TABLA_DE_CONTENIDOS.md`.

---

## Contacto

**Repositorio:** `git@github.com:juansuarez-pragma/maestro-movil.git`
**Organización:** Pragma
**Última actualización:** Enero 2025
