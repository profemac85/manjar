# Navegación por días en HOY + macros por período en DATOS

Diseño aprobado el 2026-08-12. Dos mejoras a manjar.html, sin cambios de formato en los
datos guardados (las comidas ya viven con timestamp completo).

## 1. Navegación por días en la vista HOY

**Estado.** `diaVisto` (yyyy-mm-dd), parte en hoy al abrir la app. Toda la vista HOY se
pinta para `diaVisto`: ecuación, barra, macros, anillo, riel, panel de movimiento y lista
de comidas. En días sin movimiento (incluido el futuro) las quemadas van en 0.

**Barra superior.** `‹ fecha ›`. Flechas mueven un día. Tocar la fecha estando en otro día
vuelve a hoy; estando en hoy abre el calendario nativo (input date oculto, `showPicker()`
con fallback a `focus()`).

**Señal de otro día.** Fecha en color manjar con texto relativo ("ayer", "mañana", o
"lun 4 ago" si está lejos). El rótulo cambia: pasado "Quedó disponible", futuro
"Planificado", hoy "Disponible hoy" (igual que ahora). Sello de movimiento y estado vacío
con textos propios del día visto.

**Edición.** La lista del día visto conserva los mismos controles de siempre: hora editable
inline, borrar, y Agregar para sumar comida a ese día.

**Captura.** Estando parado en otro día, los chips Desayuno/Almuerzo/Once/Cena y "Ahora"
apuntan a `diaVisto` ("Ahora" usa la hora actual con la fecha vista). Al entrar a Agregar,
los campos de fecha y hora se reinician al día visto y la cabecera avisa "registrando en
ayer" cuando no es hoy. El campo sigue siendo editable a mano. La regla de "aún no llega,
era de ayer" de los chips solo aplica estando en hoy.

**Sin cambios.** MOVER, DATOS, el atajo de Salud (escribe al día que él manda) y la edición
de hora inline.

## 2. DATOS por período con panel de macros

**Selector.** Segmento Semana (7 días) / Mes (30 días) arriba de la vista. Gobierna KPIs,
comido contra gasto, pasos y el panel nuevo. La leyenda de la topbar muestra el rango.

**Panel "Macros por día".** Barra apilada por día: altura total = kcal comidas, segmentos
por aporte calórico de cada macro (prote×4 sandía, carbo×4 manjar, grasa×9 agua, los colores
del anillo). Línea de meta calórica. Días sin registro quedan como muñón tenue.

**Resumen del período.** Sobre el gráfico, las tres filas de macros (mismo estilo de las
barras de HOY): promedio diario en gramos contra la meta, y en la esquina el reparto
porcentual de calorías del período. Promedios solo sobre días con registro.

**KPIs.** kcal, pasos y balance promedio calculados sobre el período elegido.

## Validación

1. Flujo del CLAUDE.md: extraer JS, chequear ids/handlers/divs, `node --check`.
2. Simulación en node de la lógica nueva: desplazamiento de días por cruces de mes y año,
   etiqueta relativa, chips sobre día visto, apilado de macros (los segmentos suman la
   altura del día) y promedios que excluyen días vacíos.
3. Prueba visual en el artifact publicado; después push a GitHub Pages.

## Pendiente aparte (anotado, no en este cambio)

Rediseño estético general del HTML: a Max no le gustó cómo quedó. Se aborda como paso
propio después de estas mejoras.
