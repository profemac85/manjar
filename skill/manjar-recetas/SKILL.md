---
name: manjar-recetas
description: Usar cuando el usuario pida guardar o crear una receta para su app Manjar - mensajes que comienzan con "receta:", "guarda esta receta", "hazme una receta", "crea una receta", correcciones a una receta recién entregada (ingredientes, gramos, rendimiento, porciones, pasos), o cuando describa una preparación con ingredientes que quiera dejar registrada con sus macros.
---

# Manjar: guardar recetas

Convierte una receta (descrita por el usuario o creada contigo) en el bloque JSON que la
pestaña Pegar de su app Manjar sabe leer. La app guarda la receta en la despensa con sus
ingredientes, pasos y macros, y calcula los valores por 100 g a partir de los ingredientes.

Dos modos:

- **Guardar una receta suya:** entrega el resumen y el bloque AL TIRO, con todo supuesto
  declarado. Las correcciones vienen después y re-emiten el bloque completo.
- **Crear una receta a pedido** ("hazme un queque proteico de 150 kcal por porción"):
  acá sí conversa e itera hasta que el usuario esté conforme, y cada propuesta termina
  con el bloque listo por si ya le gustó.

## Reglas de ingredientes

1. Cada ingrediente lleva sus gramos CRUDOS y sus valores por 100 g (kcal, prote, carbo,
   grasa y SIEMPRE fibra; 0 si no aporta).
2. Si hay una tabla de despensa pegada en la conversación (formato
   `nombre | porción g | kcal/100g | prote/100g | carbo/100g | grasa/100g | fibra/100g`),
   USA ESOS VALORES TAL CUAL para los ingredientes que aparezcan ahí.
3. Productos de marca que no estén en la despensa: busca la tabla real en la web
   (fabricante chileno primero, después FatSecret u Open Food Facts). Genéricos (huevo,
   avena, palta): tablas estándar, sin buscar.
4. Pesos explícitos se respetan; medidas caseras se convierten y se declaran
   (1 taza de avena 90 g, 1 huevo 50 g, 1 cda de aceite 14 g, 1 cdta 5 g).

## Rendimiento y porciones

- `rinde_g` es el peso final YA COCIDO. Si el usuario no lo da, estímalo desde la merma
  típica y decláralo: horneados pierden 10 a 20% de agua, guisos cerca de 10%, sin cocción
  0%. Si no aplica merma, omite `rinde_g` (la app usa la suma cruda).
- `porcion_g`: si el usuario dice "salen 8 porciones", porcion_g = rinde entre 8. Si no
  dice nada, usa una porción razonable y decláralo.

## Los pasos

Preparación en pasos claros y breves, uno por elemento del array, en el orden real. Incluye
temperaturas y tiempos cuando existan. Sin numerarlos en el texto (la app los numera).

## Formato de salida

Al final del mensaje, dentro de un fence ```json, UN solo objeto y nada después:

```json
{"receta":{"nombre":"...","porcion_g":0,"rinde_g":0,
 "ingredientes":[{"nombre":"...","gramos":0,"kcal_100g":0,"prot_100g":0,"carb_100g":0,"gras_100g":0,"fibra_100g":0}],
 "pasos":["...","..."],
 "nota":"supuestos: merma, conversiones, fuentes"}}
```

- El parser de la app busca el objeto envoltorio de afuera hacia adentro: si emites varios
  objetos sueltos o texto después del bloque, la receta se pierde. El bloque es SIEMPRE lo
  último del mensaje.
- La app recalcula los macros desde los ingredientes y el rendimiento; los números que tú
  reportas en el texto son para que el usuario valide, así que calcúlalos con la misma
  fórmula: por 100 g = suma de aportes de ingredientes dividida por rinde_g por 100.

## La respuesta

1. Resumen corto: macros de la RECETA COMPLETA (kcal, P, C, G, F y rendimiento) y de CADA
   PORCIÓN (con sus gramos), más los supuestos (merma asumida, conversiones de medidas,
   fuentes de valores buscados).
2. El bloque JSON al final.

Correcciones posteriores ("ponle 300 g de queso", "que salgan 6 porciones") re-emiten el
bloque COMPLETO corregido, de nuevo como lo último del mensaje.

## Casos borde

- Receta sin ingredientes reconocibles o sin cantidades: pregunta lo mínimo, no inventes
  gramos imposibles de adivinar.
- Si el usuario quiere REGISTRAR que comió algo (no guardar una receta), eso es de la skill
  "manjar" de registro, no de esta.

## Ejemplo

Usuario: "receta: cheesecake proteico. 250 g de queso crema light, 200 g de yogurt griego,
100 g de avena. Se hornea 35 min a 180 y sale un molde de unos 480 g. Lo corto en 4."

Respuesta: "Receta completa: 754 kcal, P65 C84 G18 F11, rinde 480 g cocidos (asumí que los
480 g que me diste son el peso final). Cada porción de 120 g: 189 kcal, P16 C21 G5 F3.
Valores de ingredientes con tablas estándar."

```json
{"receta":{"nombre":"Cheesecake proteico","porcion_g":120,"rinde_g":480,
 "ingredientes":[
  {"nombre":"Queso crema light","gramos":250,"kcal_100g":98,"prot_100g":11,"carb_100g":4,"gras_100g":4,"fibra_100g":0},
  {"nombre":"Yogurt griego","gramos":200,"kcal_100g":60,"prot_100g":10,"carb_100g":4,"gras_100g":0.5,"fibra_100g":0},
  {"nombre":"Avena","gramos":100,"kcal_100g":389,"prot_100g":17,"carb_100g":66,"gras_100g":7,"fibra_100g":11}],
 "pasos":["Moler la avena hasta harina","Batir el queso crema con el yogurt y sumar la avena","Hornear 35 min a 180° y enfriar antes de cortar"],
 "nota":"480 g asumidos como peso final cocido. Valores de tablas estándar."}}
```
