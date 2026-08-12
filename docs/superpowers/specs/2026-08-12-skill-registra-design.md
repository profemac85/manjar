# Skill "manjar": registrar comidas desde Claude

Diseño aprobado el 2026-08-12. Cierra el primer pendiente del CLAUDE.md: una skill de Claude
que reconozca "registra: ..." y devuelva el bloque JSON listo para la pestaña Pegar, sin
copiar el prompt largo desde la app.

## Objetivo

Que Max le escriba a Claude en la app del iPhone algo como
"registra: una porción de granola Wild Foods dorada y un yogurt Colun light" y reciba en un
solo mensaje el bloque JSON que `leerPegado()` entiende, con valores nutricionales reales
buscados en internet cuando el producto es de marca.

## Decisiones tomadas

1. **Dónde corre:** skill de claude.ai, subida en Ajustes > Capacidades > Skills, para usarla
   desde la app del iPhone. No es una skill de Claude Code.
2. **Despensa híbrida:** la skill funciona sin despensa (estima todo). Si en la conversación
   hay una tabla de despensa pegada (la del botón "Copiar prompt con mi despensa", formato
   `nombre | porción g | kcal/100g | prote/100g | carbo/100g | grasa/100g`), usa esos valores
   tal cual, sin reestimarlos. La fuente de verdad sigue siendo el teléfono.
3. **Bloque al tiro:** ante "registra: ...", Claude entrega inmediatamente el desglose con
   supuestos marcados y el bloque JSON. Las correcciones posteriores ("eran 200 g") re-emiten
   el bloque completo corregido. No conversa antes de entregar.
4. **Búsqueda web como comportamiento central:** para productos envasados o de marca que no
   estén en la despensa pegada, busca en internet la tabla nutricional real.
5. **Estructura:** un solo `SKILL.md`, sin archivos de referencia. Vive en `skill/manjar/`
   en este repo (fuente de verdad versionada); se empaqueta como zip para subir a claude.ai.

## Comportamiento

### Desglose

Las mismas reglas del prompt actual de `armarPrompt()` en manjar.html:

- Un ítem por alimento, nunca sumados en uno solo.
- Juntar repeticiones del mismo alimento en un ítem con la porción total.
- Heurísticas de porción: un puñado 60 a 80 g, un puñado grande 120 g, una porción de carne
  100 g, una de cereal 40 g. Los pesos explícitos se respetan tal cual.
- Descontar ingredientes excluidos ("sin mayonesa").
- Horas en formato 24 h por ítem cuando se mencionan. "Desayuno" es 08:00, "almuerzo" 13:30,
  "once" 18:30, "cena" 21:00 (las mismas horas de los chips de la app).
- Valores del alimento ya preparado como se come, considerando la cocción.
- Valores típicos de alimentos y marcas de Chile.

### Búsqueda web

- **Busca** la tabla nutricional real de productos envasados o de marca que no estén en la
  despensa pegada. Prioriza fuentes chilenas: sitio del fabricante primero, después
  secundarias (FatSecret, Open Food Facts, supermercados).
- **No busca** alimentos genéricos (pollo a la plancha, arroz, palta): tablas estándar.
- Normaliza todo a valores por 100 g; si la etiqueta viene por porción, convierte.
- Confianza alta si salió de tabla oficial, media si es fuente secundaria, baja si quedó
  estimado. La fuente se menciona en la `nota`.
- Si la búsqueda no encuentra la tabla, estima con confianza media o baja y lo dice.
- Todo producto buscado va también en el array `despensa` del bloque, para que la app lo
  aprenda y la próxima vez no haya que buscar.

### Respuesta

Un solo mensaje con dos partes:

1. Resumen legible de una o dos líneas: total de kcal y proteína, supuestos principales
   (toda porción asumida se dice explícitamente para que Max la valide).
2. El bloque JSON al final del mensaje, dentro de un fence ```json (el fence da botón de
   copiar en la app y `extraerJSON()` tolera los backticks).

Correcciones: cualquier ajuste posterior re-emite el bloque completo corregido, de nuevo como
lo último del mensaje. Max copia siempre el último bloque.

## Formato de salida (contrato con la app, no negociable)

```json
{"comidas":[{"nombre":"...","hora":"13:30","porcion_g":0,"kcal_100g":0,"prot_100g":0,"carb_100g":0,"gras_100g":0,"confianza":"alta|media|baja"}],
 "despensa":[{"nombre":"...","porcion_g":0,"kcal_100g":0,"prot_100g":0,"carb_100g":0,"gras_100g":0}],
 "nota":"supuestos principales en una o dos frases"}
```

- UN solo objeto envoltorio, lo último del mensaje. `extraerJSON()` busca de afuera hacia
  adentro; varios objetos sueltos o texto después del bloque rompen el parseo.
- `hora` es opcional por ítem, formato HH:MM de 24 h.
- `despensa` lleva solo alimentos nuevos o corregidos; vacía si no hay ninguno.
- `confianza` en minúsculas: alta, media o baja.

## Ejemplo de referencia (validado en rol el 2026-08-12)

Entrada: "Hoy comí 1 porción de granola protein Wild Foods dorada, un yogurt Colun light
natural, y un yogurt Soprole Protein natural endulzado. Esto fue al desayuno."

Respuesta esperada: resumen ("Desayuno registrado a las 08:00: 297 kcal, 26 g de proteína",
fuentes y porción de granola asumida en 37 g) y el bloque:

```json
{"comidas":[
  {"nombre":"Granola Wild Protein Golden Crunchy","hora":"08:00","porcion_g":37,"kcal_100g":346,"prot_100g":29.5,"carb_100g":35.4,"gras_100g":9.7,"confianza":"media"},
  {"nombre":"Yogurt Colun Light natural","hora":"08:00","porcion_g":120,"kcal_100g":53,"prot_100g":4,"carb_100g":8.5,"gras_100g":0.3,"confianza":"media"},
  {"nombre":"Yogurt Soprole Protein+ natural endulzado","hora":"08:00","porcion_g":155,"kcal_100g":68,"prot_100g":6.6,"carb_100g":6.3,"gras_100g":1.8,"confianza":"alta"}],
 "despensa":[
  {"nombre":"Granola Wild Protein Golden Crunchy","porcion_g":37,"kcal_100g":346,"prot_100g":29.5,"carb_100g":35.4,"gras_100g":9.7},
  {"nombre":"Yogurt Colun Light natural","porcion_g":120,"kcal_100g":53,"prot_100g":4,"carb_100g":8.5,"gras_100g":0.3},
  {"nombre":"Yogurt Soprole Protein+ natural endulzado","porcion_g":155,"kcal_100g":68,"prot_100g":6.6,"carb_100g":6.3,"gras_100g":1.8}],
 "nota":"Soprole desde tabla oficial (68 kcal/100g). Granola y Colun desde FatSecret porque las tablas oficiales están en imagen. Porción de granola asumida en 37 g."}
```

Notas del ejemplo: Soprole salió de soprole.cl (oficial, confianza alta); Wild Foods y Colun
publican la tabla solo como imagen, así que se usó FatSecret (confianza media). Los envases
de yogurt (120 y 155 g) son pesos reales de las fuentes; la porción de granola quedó asumida
y declarada.

## Activación

- Frontmatter de la skill con `name: manjar` y una `description` en español que gatille con:
  "registra:", "anota lo que comí", descripciones de comida dirigidas al registro, y
  menciones de Manjar.
- El disparador principal y documentado es "registra: ...". El resto es tolerancia, no
  promesa.

## Casos borde

- Mensaje sin comida reconocible: preguntar, no inventar.
- Comida mezclada con otra petición: registrar la comida y responder lo demás aparte, con el
  bloque siempre al final.
- Tabla de despensa pegada con formato imperfecto: usar lo que se entienda, ignorar líneas
  ilegibles sin reclamar.
- Producto de marca no chileno o inexistente: buscar igual; si no aparece, estimar con
  confianza baja y decirlo.

## Empaquetado e instalación

1. `skill/manjar/SKILL.md` en este repo.
2. `cd skill && zip -r manjar.zip manjar` para generar el paquete.
3. Subir el zip en claude.ai: Ajustes > Capacidades > Skills.
4. Actualizaciones: editar en el repo, re-empaquetar, re-subir.

## Validación (manual, no hay tests posibles del lado de claude.ai)

1. Registro simple sin despensa (el ejemplo de referencia).
2. Registro con la tabla de despensa pegada antes: los valores pegados se usan tal cual.
3. Corrección posterior ("la granola eran 60 g"): el bloque completo se re-emite corregido.
4. Cada bloque generado se pega en el artifact publicado (o en la app) y se verifica que
   `leerPegado()` lo parsea y el desglose editable muestra todos los ítems con sus horas.

## Fuera de alcance

- Sincronización automática de la despensa entre teléfono y skill (la copia se desfasa y eso
  está aceptado, igual que en el ciclo de pegado actual).
- Cambios en manjar.html: la app ya parsea este formato; no se toca nada.
- Registro por foto vía skill (la app ya lo hace con la API cuando tenga clave).
