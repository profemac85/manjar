# Recetas consultables y skill manjar-recetas

Aprobado el 2026-09-20.

## En la app

- Las recetas suman `pasos` (preparación, un paso por línea en el constructor).
- Despensa con secciones: segmento Todo | Recetas (`despFiltro`).
- Ficha de consulta (`fichaHTML`): tocar una receta abre ingredientes, pasos numerados y
  macros por porción y de la receta completa (con número de porciones), más botones
  "Registrar una porción hoy", "Editar receta" y cerrar. Los productos normales siguen
  abriendo su editor.
- Pegar acepta el bloque `{"receta":{...}}`: `guardarRecetaPegada()` valida nombre e
  ingredientes con gramos, calcula por 100 g con `calcularReceta()` (misma matemática del
  constructor), guarda o actualiza por nombre normalizado y abre la ficha en Despensa.
  El validador de `extraerJSON()` se extendió con `o.receta` respetando el orden de
  afuera hacia adentro.

## La skill manjar-recetas (claude.ai, separada de la de registro)

Gatilla con "receta:", "guarda esta receta", "hazme una receta". Dos modos: guardar
(bloque al tiro con supuestos declarados) y crear a pedido (conversa e itera, cada
propuesta cierra con el bloque). Reglas: ingredientes con gramos crudos y valores por
100 g (despensa pegada tal cual, marcas por web, genéricos con tablas), conversiones de
medidas caseras declaradas, `rinde_g` cocido con merma declarada (o omitido para usar la
suma cruda), `porcion_g` desde el número de porciones. Testeada con subagente y validada
con el `extraerJSON()` real: despensa respetada, conversiones y merma declaradas, porción
bien calculada, bloque parseable y último.
