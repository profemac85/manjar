# Los dos atajos de Salud

Manjar no puede hablar con Apple Salud desde la web, así que usa dos atajos de iOS como
puente. Los nombres tienen que ser exactos, porque la app los invoca por nombre.

## Atajo 1: "Manjar Sync" (trae pasos y energía activa)

En la app Atajos, toca + para crear uno nuevo y agrega estas acciones en orden. Cada acción
se busca escribiendo su nombre en el campo "Buscar acciones" de abajo.

1. **Buscar muestras de Salud**
   - `Tipo`: dejar en **Steps** (viene así por defecto).
   - `Fecha de inicio`: toca donde dice "está entre los últimos 7 días" y cámbialo a
     **es hoy**.
2. **Calcular estadísticas**
   - Cambia **Promedio** por **Suma** (toca la palabra Promedio).
   - Debe decir "de Muestras de Salud", se conecta solo.
3. **Establecer variable**
   - Nombre de la variable: **pasos**, valor: la Suma anterior (se conecta sola).
4. **Buscar muestras de Salud** (la segunda vez)
   - `Tipo`: cámbialo a **Energía activa**.
   - `Fecha de inicio`: **es hoy**.
5. **Calcular estadísticas**
   - Otra vez **Suma**.
6. **Establecer variable**
   - Nombre: **activas**.
7. **Texto**
   - Pega exactamente esto:
     `https://profemac85.github.io/manjar/manjar.html#salud={"pasos":PASOS,"activas":ACTIVAS}`
   - Ahora reemplaza la palabra `PASOS` por la variable **pasos** y `ACTIVAS` por la
     variable **activas**. Para insertarlas: borra la palabra, y con el cursor ahí toca la
     barra de variables que aparece sobre el teclado (o el botón "Seleccionar variable").
8. **Abrir URLs**
   - Debe tomar el Texto anterior.

Finalmente toca el nombre arriba, elige **Renombrar** y escribe exactamente `Manjar Sync`.

La primera vez que lo corras, iOS va a pedir permiso para leer datos de Salud. Acepta.

### Si te complica el paso de la energía activa

Se puede omitir: si el atajo manda solo los pasos, Manjar estima las calorías activas a
partir de ellos y de tu peso. Quedaría con las acciones 1, 2, 7 y 8, y el texto sería
`https://profemac85.github.io/manjar/manjar.html#salud={"pasos":PASOS}`.
La fecha tampoco es obligatoria: si no va en el JSON, la app usa el día de hoy.

## Atajo 2: "Manjar Registrar" (manda lo comido a Salud)

Este recibe el texto que le manda la app y escribe la energía consumida.

1. **Obtener el valor del diccionario**
   - Llave: `kcal`, en la **Entrada del atajo**.
2. **Registrar muestra de salud**
   - Tipo: **Energía consumida**.
   - Valor: el resultado del paso anterior.
   - Fecha: hoy.

Renómbralo exactamente `Manjar Registrar`.

En Ajustes del atajo (el ⓘ de abajo) activa **Mostrar en la hoja de compartir** para que la
app pueda pasarle el texto.

## Cómo se usan

- En Manjar, pestaña MOVER, botón "Traer pasos desde Salud": abre "Manjar Sync", que vuelve
  a la app con los datos en el hash de la URL.
- En Perfil, "Enviar lo comido a Salud": abre "Manjar Registrar" con el resumen del día.
