# Manual de programación de KameAR

Este manual explica cómo programar el robot con las fichas de [KameAR](https://skymaker.github.io/KameAR/). No hace falta saber programar: un programa es una fila de fichas que el robot ejecuta una detrás de otra, de izquierda a derecha.

## El panel de fichas

El panel flota sobre el robot. En las Meta Quest aparece también sobre la palma de la mano no dominante cuando la vuelves hacia ti. Tiene cuatro zonas, de arriba abajo:

| Zona | Para qué sirve |
|---|---|
| Botones | Ejecutar, parar y borrar el programa |
| Programa | La fila de fichas que has puesto (hasta 18, en dos filas de 9) |
| Edición | Cambia la ficha del programa que hayas seleccionado |
| Paleta | Las fichas disponibles, en tres grupos: Mover, Acciones y Control |

Cómo se pulsa en cada dispositivo:

| Dispositivo | Cómo pulsar |
|---|---|
| Móvil | Toca la ficha en la pantalla |
| Meta Quest | Apunta con el rayo de la mano dominante y pellizca, o pulsa con el índice el panel de la palma |
| Escritorio | Haz clic con el ratón |

El panel se esconde al pulsar **Ejecutar**, para dejar ver al robot. Toca el cuerpo del robot para que vuelva a aparecer; si lo tocas otra vez, se esconde.

## Crear y cambiar un programa

1. **Añadir:** toca una ficha de la paleta. Se coloca al final del programa.
2. **Seleccionar:** toca una ficha del programa. Queda rodeada de azul y aparecen sus botones en la zona de edición. Tócala de nuevo para soltarla.
3. **Cambiar el número:** con la ficha seleccionada, usa **−** y **+**. Las fichas «repetir» y «esperar» tienen además **−10** y **+10**.
4. **Cambiar el orden:** con la ficha seleccionada, **◀** y **▶** la mueven un puesto.
5. **Quitar:** **✕ Quitar** borra la ficha seleccionada.

El programa se guarda solo en el navegador: al volver a abrir la página sigue ahí. Si cambias algo mientras el robot está ejecutando, el programa se detiene.

## Los botones

| Botón | Qué hace |
|---|---|
| ▶ Ejecutar | Ejecuta el programa desde el principio y esconde el panel. Tras un paso, se llama **▶ Seguir** y continúa desde donde estaba |
| ⏭ Paso | Ejecuta una sola ficha y se detiene. Sirve para ver qué hace cada ficha y para buscar errores |
| ■ Parar | Detiene el programa y deja al robot en reposo |
| ⌫ Quitar | Borra la ficha seleccionada o, si no hay ninguna, la última |
| 🗑 Todo | Borra el programa entero |
| 📍 Mover | Solo en realidad aumentada: recoge el robot para colocarlo en otro sitio |
| ⟲ Inicio | Solo en modo 3D: detiene el programa y devuelve el robot al punto de partida |

Mientras el robot ejecuta, la ficha en curso aparece rodeada de blanco.

## Fichas de movimiento (azules)

Mueven al robot por el suelo. El número de la esquina indica cuántas veces se hace el movimiento.

| Ficha | Nombre | Qué hace | Número |
|:---:|---|---|:---:|
| ↑ | Avanzar | Camina hacia delante un tramo corto por cada unidad | 1 a 9 |
| ↓ | Retroceder | Camina hacia atrás un tramo corto por cada unidad | 1 a 9 |
| ↺ | Girar a la izquierda | Gira sobre sí mismo un cuarto de vuelta (90°) por cada unidad | 1 a 4 |
| ↻ | Girar a la derecha | Gira sobre sí mismo un cuarto de vuelta (90°) por cada unidad | 1 a 4 |

Cosas que conviene saber:

- **Los giros son exactos:** el robot mide su rumbo y sigue girando hasta completar el ángulo. Con el número 2 da media vuelta y con el 4, una vuelta entera.
- **Los tramos no son exactos:** el robot camina con físicas reales, así que puede resbalar o desviarse un poco, sobre todo al retroceder. Lo mismo le pasa al robot de verdad.
- **Los obstáculos lo frenan:** si choca con una pared o un mueble, sigue moviendo las patas sin avanzar hasta terminar la ficha.

## Fichas de acción (naranjas)

Son movimientos que el robot hace sin desplazarse. No llevan número: para repetirlos, mételos en una ficha «repetir».

| Ficha | Nombre | Qué hace |
|:---:|---|---|
| 👋 | Saludar | Se sienta y agita una pata |
| ⤒ | Saltar | Se agacha y da un salto |
| 💃 | Bailar | Balancea el cuerpo sin moverse del sitio |
| 💪 | Flexiones | Sube y baja la parte delantera del cuerpo |

## Fichas de control (amarillas)

No mueven al robot: deciden cómo se ejecutan las demás.

| Ficha | Nombre | Qué hace | Número |
|:---:|---|---|:---:|
| 🔁( | Repetir | Repite las fichas que hay hasta su cierre | 1 a 99 veces |
| ) | Cerrar | Marca dónde termina un «repetir» o un «si hay obstáculo» | — |
| ⏱ | Esperar | El robot se queda quieto | 1 a 99 segundos |
| 🚧( | Si hay obstáculo | Ejecuta las fichas que hay hasta su cierre solo si hay algo delante | — |
| 🐢 ▶ 🐇 | Velocidad | Cambia la velocidad de las fichas siguientes | lenta, normal, rápida |

### Repetir y cerrar

«Repetir» abre un paréntesis y «cerrar» lo termina. Todo lo que queda entre las dos fichas se repite las veces que diga el número.

```
🔁(3   ↑   👋   )   ↻
```

El robot avanza y saluda tres veces, y al final gira a la derecha una sola vez, porque el giro está fuera del paréntesis.

- **Si olvidas cerrar,** el paréntesis se cierra solo al final del programa: se repite todo lo que venga detrás.
- **Un cierre que sobra** no hace nada.
- **Se pueden meter unos dentro de otros:** cada «cerrar» termina el paréntesis abierto más cercano.

### Si hay obstáculo

Al llegar a esta ficha, el robot mira si tiene algo a menos de 25 cm por delante. Si lo hay, ejecuta las fichas de dentro; si no, las salta y sigue por detrás del cierre.

```
🚧(   ↻   )   ↑
```

Si hay algo delante, gira a la derecha y luego avanza. Si no hay nada, solo avanza.

- **Solo mira una vez,** en el momento de llegar a la ficha. Para que vuelva a mirar, métela en un «repetir».
- **Solo ve el entorno real detectado** en realidad aumentada: paredes, muebles y otros objetos de la habitación. En el modo 3D no hay obstáculos y las fichas de dentro nunca se ejecutan.
- **No ve tus manos.**

### Velocidad

Al añadirla marca la tortuga (lenta). Selecciónala y usa **+** para pasar a normal (▶) o a rápida (🐇).

- Afecta a las fichas que vienen **después** de ella, hasta otra ficha de velocidad.
- Cambia el ritmo de avanzar, retroceder, girar, bailar y las flexiones. Saludar, saltar y esperar duran siempre lo mismo.
- Cada vez que se ejecuta el programa empieza en velocidad normal.

## Programas de ejemplo

### Un cuadrado

```
🔁(4   ↑2   ↻   )
```

Repite cuatro veces: avanza dos tramos y gira un cuarto de vuelta. El robot vuelve cerca del punto de partida.

### Ida y vuelta

```
↑3   ↻2   ↑3   ↻2   👋
```

Avanza, da media vuelta, regresa, vuelve a mirar hacia delante y saluda.

### Explorador

```
🔁(20   🚧(   ↓   ↻   )   ↑   )
```

Repite veinte veces: si hay algo delante, retrocede y gira; después avanza un tramo. El robot recorre la habitación esquivando paredes y muebles. Solo funciona en realidad aumentada con el entorno detectado.

### Espectáculo

```
🐇   👋   🔁(2   💃   💪   )   ⏱2   ⤒
```

Pone la velocidad rápida, saluda, baila y hace flexiones dos veces, espera dos segundos y termina con un salto.

### Paseo lento y rápido

```
🐢   ↑2   🐇   ↑2   ▶   ↻4
```

Avanza despacio, luego deprisa, y termina dando una vuelta entera a velocidad normal.

## Si algo no sale como esperabas

| Qué pasa | Por qué | Qué hacer |
|---|---|---|
| No puedo añadir más fichas | El programa admite 18 fichas | Usa «repetir» para acortarlo |
| Se repite más de lo que quería | Falta la ficha «cerrar» | Ponla donde debe terminar la repetición |
| «Si hay obstáculo» no hace nada | No hay nada a menos de 25 cm, o estás en modo 3D | Pruébalo en realidad aumentada, frente a una pared |
| El robot no llega donde yo quería | Los tramos dependen del suelo y de los choques | Ajusta el número de la ficha y prueba con **⏭ Paso** |
| No veo el panel | Se esconde al ejecutar | Toca el cuerpo del robot |
| El robot se ha caído o se ha quedado atascado | Las físicas son reales | Pulsa **📍 Mover** (o **⟲ Inicio** en modo 3D) y colócalo de nuevo |
