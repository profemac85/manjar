# Manjar

App personal de registro de calorías y macros para el iPhone de Max. Un solo archivo,
`manjar.html`, sin build, sin dependencias, sin framework. Se usa como PWA agregada a la
pantalla de inicio.

## Regla número uno

**Todo vive en `manjar.html`.** HTML, CSS y JS en el mismo archivo. No dividir en módulos,
no agregar `package.json`, no meter librerías por CDN. Si algo parece pedir una dependencia,
primero proponerlo y esperar respuesta, no instalarlo.

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
deben llamarse exactamente "Manjar Sync" (lee pasos y energía activa, devuelve por hash de
URL `#salud={"fecha":"aaaa-MM-dd","pasos":N,"activas":N}`) y "Manjar Registrar" (escribe la
energía consumida). La app lee el hash al cargar y en `hashchange`.

**Hosting.** GitHub Pages desde el repo público github.com/profemac85/manjar (cuenta gh
profemac85). La app vive en https://profemac85.github.io/manjar/manjar.html y el index.html
solo redirige ahí. Deploy = commit + push; el build tarda cerca de un minuto. También hay un
artifact privado de prueba en claude.ai (la URL está en la memoria de Claude); ahí Foto y
Describir no funcionan porque el CSP bloquea la API.

## Lenguaje visual

Paleta en `:root`. Fondo tinta `#0D1B1E`, acento manjar `#E0A32E`, proteína sandía `#E4576B`,
carbo manjar, grasa agua `#5FA8B5`, verde ok `#86B04A`. Tipografía de despliegue
Avenir Next Condensed.

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
  vuelve a hoy (y en hoy abre el calendario). En otro día la fecha se pinta manjar y el
  rótulo cambia (pasado "Quedó disponible", futuro "Planificado"). Los chips de captura
  apuntan al día visto.
- **CAPTURA.** Cuatro pestañas: Foto, Describir, Manual, Pegar.
- **MOVER.** Pasos y kcal activas. Si las activas vienen vacías se estiman desde los pasos
  (aprox `0.00045 * peso` por paso).
- **DATOS.** Selector Semana/Mes (7 o 30 días, `perDatos`) que gobierna los KPIs y las
  series. Panel "Macros por día": barras apiladas por aporte calórico de cada macro (la
  lógica del anillo llevada a la serie) con resumen de promedios contra metas y reparto
  porcentual, calculado solo sobre días con registro. Después comido contra gasto, pasos,
  y peso.
- **PERFIL.** Mifflin-St Jeor para BMR y TDEE, proteína 1.8 g/kg, grasa 0.8 g/kg, piso de
  1500/1200 kcal. Interruptor "sumar lo quemado a la meta" (si está activo, la actividad base
  debe quedar en Sedentaria para no contar doble). Clave de API, despensa, respaldo.

## El anillo del día

Radio 50, circunferencia completa = `meta + quemadas`. Se llena con lo comido, repartido en
tres arcos proporcionales a las **calorías** que aporta cada macro (proteína ×4, carbo ×4,
grasa ×9), no a los gramos. El arco verde tenue va desde `meta` hasta `meta+quemadas` y solo
aparece cuando el interruptor de compensar está activo: representa el espacio extra que dio
el movimiento. Si se pasa del tope, los tres arcos se vuelven sandía y aparece un aro fino
por fuera (radio 58) con el exceso. **El centro va vacío**, porque la cifra ya está a la
izquierda.

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
