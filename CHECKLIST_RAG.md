# Checklist de Calidad (RAG / Documentación)

Esta checklist ayuda a mantener consistencia editorial y mejorar la recuperación (RAG) sin agregar código de implementación.

## 1) Checklist por Caso (`capitulos/**/caso-*.md`)

- Tiene secciones: `0 Metadata`, `1 Planteamiento`, `2 Matriz`, `3 Profundización`, `4 Impacto`, `Glosario`, `Referencias`.
- Incluye:
  - Tabla de analítica con encabezado `| Fuente |`.
  - Tabla de KPIs con encabezado `| KPI |`.
  - Tabla de riesgos en sección 1.
  - “Mini-ADR” en sección 3.
- Glosario:
  - Tiene ancla `#glosario-de-terminos-clave`.
  - Terminos clave definidos de forma breve y consistente.
- Referencias:
  - Entre 3 y 5 links externos por caso (mínimo 3).
  - Al menos 1 fuente normativa/guía + 1 doc oficial + 1 incidente/estudio.
- Cumple regla **NO-CODE**:
  - No contiene bloques de código Dart/Kotlin/Swift.
  - Se usan tablas, políticas, pasos o diagramas ASCII.

## 2) Checklist de Índices

- `TABLA_DE_CONTENIDOS.md` enlaza todos los casos publicados y no contiene rutas obsoletas.
- `README.md` no afirma un conteo diferente al real.
- `CLAUDE.md` mantiene “Estado actual” y el formato documental.

## 3) Comandos rápidos (editoriales)

- Contar casos: `rg --files capitulos --glob 'caso-*.md' | wc -l`
- Ver faltantes 1–100 (si aplica): revisar `TABLA_DE_CONTENIDOS.md`
- Validar secciones:  
  - `rg -n \"^## 0\\. Metadata\" capitulos --glob 'caso-*.md' | wc -l`
  - `rg -n \"^## 4\\. Impacto esperado\" capitulos --glob 'caso-*.md' | wc -l`
- Detectar casos con pocas referencias:  
  - `for f in $(rg --files capitulos --glob 'caso-*.md'); do n=$(rg -o \"https?://\" \"$f\" | wc -l); [ \"$n\" -lt 3 ] && echo \"$n $f\"; done`

