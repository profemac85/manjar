# Lote: edición en HOY, porciones, % grasa, buscador, racha y fechas en DATOS

Aprobado el 2026-08-12. Ocho cambios:

1. **Bug de foco en el desglose (crítico).** `cambiarGramos()` redibujaba la lista completa
   en cada dígito y botaba el foco del input. Ahora el `oninput` actualiza solo la kcal de la
   línea, sus macros y los totales, sin reconstruir inputs (`refrescarLinea` + `pintarTotales`).
2. **Lista de HOY agrupada.** Secciones Desayuno (<11:30), Almuerzo (<16:00), Once (<19:45)
   y Cena (resto), cada una con subtotal de kcal. Grupos vacíos no se muestran.
3. **Edición inline en HOY.** Gramos editables por comida (recalcula kcal y macros
   proporcionalmente, requiere gramos > 0 de referencia) con `onchange`, no `oninput`.
4. **Porciones.** La porción oficial de un producto son los gramos guardados en la despensa
   (se actualizan en cada confirmación). Botones − + en las líneas del desglose y de HOY
   suman o restan UNA porción; si el alimento no está en la despensa, la porción es la
   cantidad con que se registró (`base_g` en el desglose, gramos actuales en HOY).
5. **% grasa corporal en Perfil (opcional).** Si está entre 3 y 70, el BMR usa Katch-McArdle
   (370 + 21,6 × masa magra); vacío mantiene Mifflin-St Jeor. Se muestra la masa magra en
   "Tus números".
6. **Buscador de despensa en Manual.** Campo de búsqueda que muestra coincidencias como
   botones y llena el formulario con la porción habitual.
7. **Racha en DATOS.** Panel con días seguidos cumpliendo: con registro y sin pasarse del
   tope del día (meta + quemadas si compensa). Hoy sin registro no corta la racha, solo no
   suma; hoy pasado la deja en 0. Puntos de los últimos 7 días (ok verde, pasado sandía,
   hueco sin registro).
8. **Fechas en DATOS.** `finDatos` con flechas que mueven la ventana de a `perDatos` días,
   tocar la fecha abre el calendario, pill "hoy" vuelve al presente. Todas las series y KPIs
   terminan en `finDatos`. La racha es siempre la actual, no depende de la ventana.

Validación: flujo CLAUDE.md, simulación node (grupos por hora, reescalado proporcional,
pasos de porción, Katch-McArdle, racha con casos borde) y prueba visual en el browser.
