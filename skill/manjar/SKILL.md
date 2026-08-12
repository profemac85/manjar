---
name: manjar
description: Usar cuando el usuario pida registrar comida para su app Manjar - mensajes que comienzan con "registra:" o "registra", descripciones de lo que comió para anotarlo, correcciones a un registro recién entregado (porciones, ingredientes, horas), o cuando pegue su despensa en formato "nombre | porción g | kcal/100g | prote/100g | carbo/100g | grasa/100g".
---

# Manjar: registrar comidas

Convierte lo que el usuario comió en el bloque JSON que la pestaña Pegar de su app Manjar
sabe leer. La respuesta es UN solo mensaje: resumen breve más el bloque al final. No
converses antes de entregar; las correcciones vienen después.

## Reglas de desglose

1. Un ítem por alimento, nunca sumados en uno solo. Once ingredientes son once ítems.
2. Junta repeticiones del mismo alimento en un ítem con la porción total.
3. Porciones habladas: un puñado 60 a 80 g, un puñado grande 120 g, una porción de carne
   100 g, una de cereal 40 g. Los pesos explícitos se respetan tal cual.
4. Si algo va sin un ingrediente ("sin mayonesa"), descuéntalo.
5. Valores del alimento ya preparado como se come: a la plancha suma poca grasa, frito bastante.
6. Horas en formato 24 h por ítem cuando se mencionen. Desayuno es 08:00, almuerzo 13:30,
   once 18:30, cena 21:00. Sin mención de hora, omite el campo.
7. Alimentos y marcas de Chile por defecto.

## Despensa del usuario

Si en la conversación hay una tabla pegada con formato
`nombre | porción g | kcal/100g | prote/100g | carbo/100g | grasa/100g`, esa es su despensa
personal: USA ESOS VALORES TAL CUAL, sin reestimarlos ni buscarlos. Ajusta solo la porción si
comió distinto. Líneas ilegibles se ignoran sin reclamar. Sin tabla pegada, estima o busca.

## Búsqueda web

- BUSCA la tabla nutricional real de productos envasados o de marca que no estén en la
  despensa pegada ("granola Wild Foods", "yogurt Colun light"). Prefiere el sitio del
  fabricante chileno; si la tabla oficial no es legible, fuentes secundarias (FatSecret,
  Open Food Facts).
- NO busques alimentos genéricos (pollo a la plancha, arroz, palta): tablas estándar.
- Normaliza a valores por 100 g; si la etiqueta viene por porción, convierte.
- Confianza: "alta" con tabla oficial, "media" con fuente secundaria, "baja" si quedó
  estimado. Menciona las fuentes en la nota.
- Si no encuentras la tabla, estima, marca la confianza y dilo en la nota.

## Formato de salida

Al final del mensaje, dentro de un fence ```json, UN solo objeto y nada después:

```json
{"comidas":[{"nombre":"...","hora":"13:30","porcion_g":0,"kcal_100g":0,"prot_100g":0,"carb_100g":0,"gras_100g":0,"confianza":"alta"}],
 "despensa":[{"nombre":"...","porcion_g":0,"kcal_100g":0,"prot_100g":0,"carb_100g":0,"gras_100g":0}],
 "nota":"supuestos principales en una o dos frases"}
```

- El parser de la app busca el objeto envoltorio de afuera hacia adentro: si emites varios
  objetos sueltos o texto después del bloque, el registro se pierde. El bloque es SIEMPRE
  lo último.
- `confianza` en minúsculas: alta, media o baja. `hora` opcional, HH:MM de 24 h.
- `despensa` lleva solo alimentos nuevos o corregidos (incluidos los buscados en la web,
  para que la app los aprenda y no haya que buscarlos de nuevo). Vacía si no hay ninguno.

## La respuesta

1. Resumen de una o dos líneas: total de kcal y proteína, y TODA porción asumida dicha
   explícitamente ("asumí la porción de granola en 37 g") para que el usuario la valide.
2. El bloque JSON al final.

Correcciones posteriores ("eran 200 g", "sin palta"): re-emite el bloque COMPLETO corregido,
de nuevo como lo último del mensaje. El usuario copia siempre el último bloque.

## Casos borde

- Mensaje sin comida reconocible: pregunta, no inventes.
- Comida mezclada con otra petición: responde lo otro primero y cierra con el bloque.

## Ejemplo

Usuario: "registra: un yogurt Soprole Protein natural endulzado y una palta chica al almuerzo"

Respuesta: "Almuerzo registrado a las 13:30: 265 kcal, 12 g de proteína. El yogurt salió de
la tabla oficial de Soprole; la palta chica la asumí en 100 g."

```json
{"comidas":[
  {"nombre":"Yogurt Soprole Protein+ natural endulzado","hora":"13:30","porcion_g":155,"kcal_100g":68,"prot_100g":6.6,"carb_100g":6.3,"gras_100g":1.8,"confianza":"alta"},
  {"nombre":"Palta","hora":"13:30","porcion_g":100,"kcal_100g":160,"prot_100g":2,"carb_100g":8.5,"gras_100g":14.7,"confianza":"media"}],
 "despensa":[{"nombre":"Yogurt Soprole Protein+ natural endulzado","porcion_g":155,"kcal_100g":68,"prot_100g":6.6,"carb_100g":6.3,"gras_100g":1.8}],
 "nota":"Yogurt desde tabla oficial de Soprole. Palta chica asumida en 100 g, valores estándar."}
```
