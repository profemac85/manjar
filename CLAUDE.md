# Manjar

App personal de registro de calorías y macros para el iPhone de Max. Un solo archivo,
`manjar.html`, sin build, sin dependencias, sin framework. Se usa como PWA agregada a la
pantalla de inicio.

## Regla número uno

**Todo vive en `manjar.html`.** HTML, CSS y JS en el mismo archivo. No dividir en módulos,
no agregar `package.json`, no meter librerías por CDN. Si algo parece pedir una dependencia,
primero proponerlo y esperar respuesta, no instalarlo.

Única excepción aprobada por Max: ZXing (@zxing/browser 0.1.5, Apache-2.0) vendorizado en un
segundo bloque `<script>` al final del archivo, marcado como código de terceros que no se
edita. Existe porque el Safari de su iPhone no trae `BarcodeDetector`. Se actualiza
reemplazando el bloque completo, nunca editándolo.

## Cómo hablarle a Max

- Español de Chile, neutro.
- **Sin rayas largas (—) para encerrar ideas.** Usar comas o paréntesis.
- Explicaciones breves y al grano salvo que pida lo contrario.
- Max tiene el criterio sobre porciones y realismo nutricional. Cuando se asuma un peso de
  comida, decírselo explícitamente para que lo valide.

## Cómo validar antes de entregar

No hay tests. El flujo que se usó siempre y conviene mantener:

```bash
# extraer el JS del HTML y chequear sintaxis
python3 -c "
import re
s=open('manjar.html',encoding='utf-8').read()
js=re.search(r'<script>\n\"use strict\";(.*?)</script>', s, re.S).group(1)
open('/tmp/m.js','w').write(js)
ids=set(re.findall(r'id=\"([^\"]+)\"', s))
print('ids faltantes:', sorted(set(re.findall(r'\\\$\(\"([^\"]+)\"\)', js))-ids))
h=set(re.findall(r'on(?:click|input|change)=\"(\w+)\(', s))
d=set(re.findall(r'(?:async )?function (\w+)', js))
print('handlers sin definir:', sorted(h-d))
print('balance divs:', s.count('<div')-s.count('</div>'))
"
node --check /tmp/m.js
```

Además, cuando se toca lógica de cálculo (anillo, riel, totales, parseo), correr una
simulación en `node` con números reales antes de dar por bueno el cambio. Así se encontró
el bug del extractor de JSON.

## Arquitectura

**Almacenamiento dual.** Detecta el entorno: si existe `window.storage` (artifact de Claude)
usa esa API asíncrona con guardado debounced a 400 ms; si no, `localStorage`. Clave:
`manjar.v2`. `cargar()` es async y se espera antes de pintar.

**API de Claude.** Endpoint `/v1/messages`, modelo `claude-sonnet-4-6`, `max_tokens` 1000.
Dentro del artifact funciona sin clave; en el teléfono requiere la clave propia de Max
guardada en el dispositivo, con `x-api-key`, `anthropic-version: 2023-06-01` y
`anthropic-dangerous-direct-browser-access: true`. Todo pasa por `pedirAClaude()`.
La búsqueda web (`web_search_20250305`) está activada solo en el modo Describir y el prompt
la restringe a productos de marca chilenos, porque cada búsqueda cuesta.

**Apple Health.** No hay HealthKit desde la web. Se usa un puente con dos atajos de iOS que
deben llamarse exactamente "Manjar Sync" (lee pasos y energía activa de hoy) y
"Manjar Registrar" (escribe la energía consumida). **El traspaso va por el portapapeles**,
no por URL: iOS manda cualquier `https://` a Safari, y el Safari normal tiene un
almacenamiento distinto al de la PWA instalada, así que los datos nunca llegarían. El Atajo
copia `{"pasos":N,"activas":N}` y `pegarSalud()` lo levanta con `clipboard.readText()` desde
un botón (iOS exige gesto del usuario). `aplicarTextoSalud()` tolera el JSON pelado o la URL
completa. Se mantiene `leerHashSalud()` para cuando la app corre en Safari. La fecha es
opcional: sin ella se asume hoy. Ojo: Atajos bloquea el traspaso hasta activar "Permitir
compartir grandes cantidades de datos" en Ajustes.

**Hosting.** GitHub Pages desde el repo público github.com/profemac85/manjar (cuenta gh
profemac85). La app vive en https://profemac85.github.io/manjar/manjar.html y el index.html
solo redirige ahí. Deploy = commit + push; el build tarda cerca de un minuto. También hay un
artifact privado de prueba en claude.ai (la URL está en la memoria de Claude); ahí Foto y
Describir no funcionan porque el CSP bloquea la API.

## Lenguaje visual

Dos temas basados en la paleta del Colegio CREE, con tokens en `:root` (CREE Día, el
defecto) y `:root[data-tema="noche"]` (CREE Noche, interruptor en Perfil, guardado en
`DB.tema`). Los nombres de las variables conservan su rol histórico: `--tinta` es el fondo
(día `#F3F5FA`, noche `#0F1D33`), `--hueso` el texto (día azul marino `#1F3864`), `--manjar`
el acento ámbar (día `#F59E1D`, noche `#FFA726`), `--sandia` proteína, `--agua` grasa
(azules CREE), `--ok` verde. **Regla: cero colores duros fuera de los dos bloques de
tokens**; los gráficos SVG usan `var()` en sus atributos, así el cambio de tema recolorea
todo sin redibujar. `aplicarTema()` también actualiza el `theme-color` de la barra de
estado. Tipografía de despliegue Avenir Next Condensed.

**Decisión de diseño que hay que respetar:** no clonar la estética de los trackers
existentes. En particular, nada de anillos tipo Apple por sí solos. El anillo que existe se
justifica porque hace algo que ningún tracker hace: se tiñe según el aporte calórico de cada
macro.

## Vistas

- **HOY.** La cifra grande no es lo consumido sino el saldo: `meta + quemadas − comidas =
  disponible`, mostrado como ecuación visible. A la derecha, el anillo (ver abajo). Barras de
  macros, panel de movimiento, "riel del día" (SVG con una barra por comida en su hora real,
  curva acumulada y línea de meta), y la lista de comidas con hora editable inline.
  La vista es navegable por días: `diaVisto` con flechas junto a la fecha, tocar la fecha
  abre el calendario nativo, y un pill "hoy" (visible solo en otro día) vuelve de un toque.
  En otro día la fecha se pinta manjar y el rótulo cambia (pasado "Quedó disponible",
  futuro "Planificado"). Los chips de captura apuntan al día visto.
  La lista de comidas va agrupada por Desayuno (<11:30), Almuerzo (<16:00), Once (<19:45) y
  Cena, con subtotal por grupo (kcal y P C G F), y es editable inline con DOS unidades
  enlazadas: gramos y porciones (cambiar una recalcula la otra, y los botones − + suman o
  restan una porción). La porción oficial de un producto son los gramos guardados en su
  despensa (`porcionDe()`; sin despensa, lo registrado cuenta como 1 porción). El mismo par
  de campos está en el desglose editable (`cambiarPorciones`/`cambiarPorcComida`, con
  `basePorcion()`), siempre actualizando el campo hermano sin redibujar para no botar el
  foco. Acepta coma decimal.
- **CAPTURA.** Cinco pestañas: Foto, Describir, Manual, Pegar, Código. Manual lleva arriba un
  buscador de despensa que llena el formulario con la porción habitual. En el desglose
  editable, `cambiarGramos` actualiza la línea SIN redibujar la lista (`refrescarLinea`):
  si se reconstruye el input mientras se escribe, se pierde el foco a cada dígito.
- **MOVER.** Sin pestaña propia: se entra desde el botón "Abrir" del panel de movimiento
  de HOY (y tiene "Volver" en su topbar). Trabaja sobre `diaVisto`. Pasos del día aparte, y `DB.actividades` como lista
  (tipo, min, kcal, fuente manual o "salud"). `movDe()` conserva su forma {pasos, activas,
  ts} con activas = suma de las actividades del día. El Atajo escribe pasos y UNA actividad
  "Salud" por día (se reemplaza, no se suma); editarla o borrarla resuelve el doble conteo
  con deportes anotados a mano. Kcal manuales vacías se estiman por MET (tabla `METS`) con
  los minutos y el peso. Swipe izquierda elimina (con Deshacer en el toast, `aviso()` acepta
  acción), tap edita en el formulario. Migración automática en `migrar()`: las `activas`
  viejas de movimiento se convierten en actividad Salud.
- **DATOS.** Períodos de calendario real: Semana (lunes a domingo, corte lunes), Mes (1 al
  último día) y Año (serie mensual donde cada barra es el promedio diario de los días con
  registro del mes, para que la línea de meta diaria siga siendo comparable). Estado
  `perTipo` + `perAncla`; flechas mueven de a un período, la etiqueta muestra el rango
  explícito, pill "hoy" vuelve al período actual. Arriba, panel de racha: días seguidos con registro y sin pasarse del tope
  (hoy sin registro no corta, hoy pasado la deja en 0; puntos de los últimos 7 días). La
  racha es siempre la actual, no depende de la ventana. Los KPIs son TOTALES del período (comido, quemado, balance; los promedios se leen
  en las barras), y el balance solo suma días con registro. Panel "Macros por día": barras
  apiladas por aporte calórico de cada macro (la lógica del anillo llevada a la serie) con
  resumen de promedios contra metas y reparto porcentual, calculado solo sobre días con
  registro. Después comido contra gasto, pasos, y peso. El peso se registra con fecha
  (un peso por día, reemplaza; solo el más reciente toca el perfil y las metas). Los KPIs
  son totales sobre los días reales del período; promedios solo sobre días con registro.
- **DESPENSA.** Quinta pestaña (la barra quedó Hoy, Datos, +, Despensa, Perfil). Buscador sobre todos los productos, "+ hoy" (abre el desglose
  editable en CAPTURA con la porción guardada precargada), edición inline (nombre, porción,
  valores por 100 g incluida fibra) y eliminación con Deshacer. Recetas: un alimento con
  `receta:{rend, ings}` guarda el snapshot de ingredientes y el rendimiento cocido;
  `calcularReceta()` deriva los valores por 100 g del producto terminado. Se registra
  consumo en gramos del terminado por el flujo normal.
- **PERFIL.** Mifflin-St Jeor para BMR y TDEE, o Katch-McArdle sobre la masa magra si el
  % de grasa corporal (opcional) está entre 3 y 70. Proteína 1.8 g/kg, grasa 0.8 g/kg, piso
  de 1500/1200 kcal. Meta de fibra configurable (30 g por defecto). Interruptor "sumar lo quemado a la meta" (si está activo, la actividad base
  debe quedar en Sedentaria para no contar doble). Clave de API, despensa, respaldo.

## El anillo del día

Radio 50, circunferencia completa = `meta + quemadas`. Se llena con lo comido, repartido en
tres arcos proporcionales a las **calorías** que aporta cada macro (proteína ×4, carbo ×4,
grasa ×9), no a los gramos. El arco verde tenue va desde `meta` hasta `meta+quemadas` y solo
aparece cuando el interruptor de compensar está activo: representa el espacio extra que dio
el movimiento. Si se pasa del tope, los tres arcos se vuelven sandía y aparece un aro fino
por fuera (radio 58) con el exceso. **El centro va vacío**, porque la cifra ya está a la
izquierda.

## La fibra

Cuarto macro: `fibra_100g` en alimentos y prompts, `f` en comidas (las viejas cuentan 0 sin
migración). Cuarta barra en HOY y cuarta fila en DATOS, color `--ok`. NO entra al anillo ni
al apilado de macros, porque esos reparten el aporte calórico y la fibra casi no aporta.

## El escáner de código de barras

Pestaña Código en CAPTURA. Dos motores en `alternarScanner()`: `BarcodeDetector` nativo si
el navegador lo trae (más rápido), y si no el ZXing vendorizado (`decodeFromCanvas` sobre un
canvas al que se copia cada cuadro del video, cada 350 ms). El Safari del iPhone de Max no
trae el nativo, así que ahí corre ZXing. Cadena de resolución de
`resolverCodigo()`: 1) la despensa local por `a.codigo` (instantáneo, sin red, valores
propios, confianza alta); 2) Open Food Facts (API v2, gratis, con CORS; es base colaborativa
así que el resultado cae al desglose editable como sugerencia con confianza media,
`alimentoDesdeOFF()` normaliza kcal desde kJ si falta energy-kcal y usa serving_quantity
como porción con 100 g de respaldo); 3) sin datos, aviso que manda al modo Foto. Al guardar,
`guardarEnDespensa()` asocia el código al alimento, así el próximo escaneo es local. La
cámara se apaga al cambiar de modo o de vista (`detenerScanner()`). El visor lleva overlay
de encuadre (esquinas, línea de barrido, tokens `--scrim`/`--video-tx` iguales en ambos
temas porque van sobre video) y feedback en tres tiempos: "Código leído · N" en verde con
vibración, la búsqueda, y "✓ nombre" con toast al encontrar el producto.

## El desglose multi-ítem

Cualquier análisis (foto, texto o pegado) devuelve `{"items":[...],"nota":"..."}` y se pinta
como una lista de líneas editables: nombre, gramos que recalculan en vivo, casilla de
incluir/excluir, marca "estimado" si la confianza es baja, y botón de quitar. Se guardan como
registros separados que comparten una misma hora, así el riel muestra una sola barra de
comida pero la despensa aprende cada producto por separado.

Heurísticas de porción que están en el prompt: un puñado 60 a 80 g, un puñado grande 120 g,
una porción de carne 100 g, una de cereal 40 g. Junta repeticiones del mismo alimento.
Descuenta ingredientes excluidos ("sin mayonesa"). Respeta los pesos explícitos.

## La despensa

Base local de alimentos confirmados, normalizada sin tildes ni mayúsculas. Mientras Max
escribe en Describir aparecen coincidencias como botones de un toque, sin gastar API. Con el
campo vacío muestra los tres más usados. Reconfirmar sube el contador de usos y actualiza la
porción por defecto.

## El ciclo de pegado (pestaña Pegar)

Existe para evitar el costo de la API y la clave en el teléfono, y para poder conversar con
Claude antes de aceptar las porciones.

**La fuente de verdad es el teléfono.** La copia que recibe Claude es una foto del momento y
se va desfasando, y eso está aceptado: cada vuelta del ciclo las vuelve a acercar.

1. `copiarPrompt()` deja en el portapapeles las instrucciones más la despensa completa en
   formato tabla.
2. Max lo pega en Claude, agrega lo que comió, conversa y corrige.
3. Claude devuelve un bloque único con `comidas` (con `hora` por ítem) y `despensa` (solo los
   alimentos nuevos o corregidos).
4. Max lo pega en la app, revisa en el desglose editable y guarda. La despensa se actualiza
   sola con `aplicarDespensaPendiente()`.

`extraerJSON()` tolera prosa alrededor y backticks. **Ojo:** busca primero el objeto
envoltorio de afuera hacia adentro, y solo después cae a objetos sueltos. Si se invierte ese
orden vuelve el bug de quedarse con un objeto anidado del array `despensa` y perder todas las
comidas.

## Pendiente

- **La skill de Claude** que reconozca "registra: ..." y devuelva el bloque JSON listo, para
  bajar la fricción del lado de Claude. Con el botón de copiar prompt ya funciona sin ella,
  pero la skill lo haría de un paso.
- **Calibrar las heurísticas de porción** con el uso real. Max tiene que ir diciendo si los
  puñados quedan cortos o largos.
- Revisar si conviene que el modo Manual también abra el desglose editable en vez del
  formulario suelto.
