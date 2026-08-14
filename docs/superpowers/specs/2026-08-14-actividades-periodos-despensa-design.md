# Lote: actividades, períodos de calendario, despensa, fibra y recetas

Aprobado el 2026-08-14, todo en una tanda.

## 1. Actividades del día (reemplaza el modelo de movimiento)

- `DB.actividades`: lista de {id, dia, tipo, min, kcal, fuente:"manual"|"salud", ts}.
- `DB.movimiento[dia]` queda solo con {pasos, ts}. Migración automática al cargar: si un
  día viejo traía `activas`, se convierte en una actividad tipo "Salud" y se borra el campo.
- `movDe(dia)` mantiene su forma {pasos, activas, ts}: activas = suma de las actividades
  del día. Todo lo que ya usaba movDe (ecuación, anillo, racha, DATOS) sigue igual.
- El Atajo (registrarMov) guarda pasos y actualiza LA actividad "Salud" del día (una sola,
  se reemplaza). Editable y borrable como cualquiera, para resolver el doble conteo a mano.
- MOVER trabaja sobre `diaVisto`: pasos del día, formulario de actividad (tipo, minutos,
  kcal con estimación por MET y peso si se deja vacío) y lista de actividades del día.
- Swipe a la izquierda revela Eliminar (toast con Deshacer por 5 s). Tap abre el formulario
  en modo edición. Todo recalcula saldo del día al momento.
- METs: caminata 3.5, trote 9, bicicleta 7, pesas 4.5, natación 7, fútbol 8, yoga 3, otro 5.

## 2. Períodos de calendario en DATOS

- Selector Semana / Mes / Año (`perTipo`) con ancla `perAncla`.
- Semana: lunes a domingo, corte fijo lunes. Mes: 1 al último día real. Año: 1 ene a 31 dic
  con serie mensual (cada barra = promedio diario de los días con registro de ese mes, para
  que las líneas de meta diaria sigan siendo comparables).
- Navegación ‹ › por período, etiqueta explícita del rango ("11 – 17 ago", "agosto 2026",
  "2026"), pill "hoy" vuelve al período actual, calendario salta a la fecha elegida.
- KPIs: totales del período sobre los días reales. Promedios (macros) solo sobre días con
  registro. Racha intacta (siempre actual).

## 3. Despensa como pestaña (sexta en la barra)

- Vista propia: buscador sobre todos los productos, lista completa ordenada por usos.
- Por producto: "+ hoy" (abre el desglose editable en CAPTURA precargado con la porción
  guardada, punto 4), tap para editar (nombre, porción y valores por 100 g), eliminar con
  Deshacer.
- El bloque de despensa de Perfil se reemplaza por un acceso a la pestaña.

## 4. Fibra como cuarto macro

- `fibra_100g` en alimentos, `f` en comidas. Lo viejo cuenta 0 sin migración.
- Meta diaria configurable en Perfil (defecto 30 g), cuarta barra en HOY (verde --ok) y
  cuarta fila en el resumen de DATOS. No entra al anillo ni al apilado (reparten calorías).
- F visible junto a P C G en detalle de ítems y totales. Campo Fibra en Manual.
- Prompts (armarPrompt, REGLAS) y skill de Claude actualizados con fibra_100g; la skill hay
  que re-subirla a claude.ai.

## 5. Recetas

- Un alimento puede llevar `receta:{rend, ings:[snapshot por ingrediente]}`.
- Constructor en Despensa: ingredientes desde la propia despensa (o a mano), gramos por
  ingrediente, rendimiento final cocido; la app calcula los valores por 100 g del producto
  terminado. Editable después (recalcula).
- Registrar consumo pide gramos del producto terminado, por el flujo normal de "+ hoy".

## Validación

Flujo CLAUDE.md + simulación node: semanas que cruzan año, meses de 28/31, agregación
anual, migración de movimiento, estimación MET, cálculo de recetas, fibra en totales.
Prueba visual en browser. Deploy a Pages y artifact.
