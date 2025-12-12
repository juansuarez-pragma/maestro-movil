# Repository Guidelines

## Estructura del Proyecto
- Contenido en `capitulos/<nn>-<tema>/caso-XX-nombre.md`, agrupado por decenas (01–10, 11–20, etc.). Ubica cada caso nuevo en la carpeta del capítulo correcto.
- Referencias base: `README.md` (visión general), `TABLA_DE_CONTENIDOS.md` (índice de los 100 casos), `CLAUDE.md` (contexto del agente y formato obligatorio).
- Cada caso debe mantener el template de seis secciones (0–5) descrito en `README.md`/`CLAUDE.md`, incluyendo metadata y TACs por plataforma.

## Comandos de Desarrollo, Build y Test
- Repositorio solo de contenido; no requiere compilación.
- Validaciones rápidas: `rg "<INSERT>" capitulos` para detectar placeholders; `rg "TODO" capitulos` para ubicar pendientes.
- Lints opcionales si existen localmente: `markdownlint **/*.md` y `spellcheck **/*.md`. Si no están, revisa manualmente títulos, tablas y fences.

## Estilo y Nomenclatura
- Nombres de archivo: `caso-XX-descripcion-kebab-case.md` con `XX` zero-padded. No renumeres casos existentes.
- Encabezados: jerarquía Markdown con `#`; usa terminología en español coherente con los títulos de capítulos.
- Tablas: mantén el formato con `|`; prefiere viñetas breves sobre párrafos largos.
- Indentación con espacios; procura líneas < ~120 caracteres para legibilidad y chunking RAG.

## Guía de Testing
- No hay suite automatizada; la verificación es editorial: valida las seis secciones, TACs por plataforma y 2–3 referencias de industria por caso.
- Al agregar escenarios, incluye criterios de aceptación claros por plataforma (Flutter, Android, iOS, Backend) y verifica que comandos/configuración sean correctos.
- Previsualiza tablas y bloques de código para evitar errores de render antes de subir cambios.

## Commits y Pull Requests
- Sigue el historial: prefijos comunes `refactor(<capitulo>)`, `docs:`, `feat:`. Mensajes imperativos y breves (ej.: `refactor(cap01): ajustar matriz de soluciones`).
- Un tema por commit; no mezcles cambios de capítulos con actualizaciones globales.
- Los PR deben incluir propósito, casos tocados (número y ruta), resumen de cambios en TACs/metadata y capturas del Markdown renderizado si se alteran tablas.
- Enlaza con la tarea o issue correspondiente y solicita revisión a los owners del capítulo impactado.

## Consejos para el Agente/RAG
- Optimiza metadata: completa pero concisa; evita repetir el mismo tag en muchos casos si no aporta recuperación.
- Al introducir nuevas herramientas o patrones, agrega un breve “Por qué” en la Sección 3 para ayudar al agente a responder con contexto.
