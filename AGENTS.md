# Repository Guidelines

## Estructura del Proyecto
- Casos en `capitulos/<nn>-<tema>/caso-XX-nombre.md` (XX con cero a la izquierda). Usa la carpeta de capítulo correcta.
- Archivos base: `README.md` (visión general), `TABLA_DE_CONTENIDOS.md` (índice real), `CLAUDE.md` (contexto/formatos), `AGENTS.md` (esta guía).
- Cada caso sigue el template 0–4 + glosario + referencias, sin código de implementación.

## Formato Obligatorio de Cada Caso
- **0. Metadata (AI-Tags):** Palabras clave, patrón técnico, stack y criticidad.
- **1. Planteamiento:** Problema técnico + escenario de negocio, incidentes reportados y riesgos (tabla).
- **2. Matriz de Soluciones:** BAJA / ACEPTABLE / ENTERPRISE con trade-offs claros.
- **3. Profundización:** Capacidades vs límites, criterio de selección y tablas de V&V/UX/seguridad/operación según aplique.
- **4. Impacto esperado:** KPIs/umbrales y resultados de negocio.  
- **Glosario:** Tabla con ancla `#glosario-de-terminos-clave`; cada término usa `id` propio (`#term-*`) y el texto debe enlazar a él para tooltips internos.  
- **Referencias:** 3–5 fuentes (incidentes, normas, guías).

## Comandos Útiles
- Validar placeholders: `rg "<INSERT>" capitulos` o `rg "TODO" capitulos`.
- Revisión rápida de links: `rg -n "\\(http" capitulos` y abrir los cambios manualmente.
- Lints opcionales si están instalados: `markdownlint **/*.md`.

## Estilo y Nomenclatura
- Archivos en kebab-case: `caso-03-mitm-certificate-pinning.md`.
- Encabezados con `#`/`##`; textos en español y tono instructivo.
- Tablas alineadas con `|`; viñetas cortas, líneas < ~120 caracteres.
- No incluir bloques de código de app; usar tablas, diagramas ASCII y descripciones.

## Guía de Testing Editorial
- Verifica que existan todas las secciones 0–4, glosario con anclas y referencias.
- Comprueba que los términos clave en el cuerpo enlacen a su `#term-*` en el glosario.
- Revisa que las cifras/KPIs estén sustentadas y que las tablas rendericen sin romper el Markdown.

## Commits y Pull Requests
- Mensajes imperativos y breves: `docs(cap01): refinar pinning` o `refactor(toc): actualizar índices`.
- Un tema por commit; separa cambios globales de casos específicos.
- En PRs, incluye propósito, rutas tocadas y notas sobre KPIs/impacto si cambian.
