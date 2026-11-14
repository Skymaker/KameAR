# KameAR
Robot cuadrípedo en Realidad Aumentada.

Basado en el trabajo de [Javier Isabel Hernandez](https://github.com/JavierIH/kame32)

**Pruébalo:** https://skymaker.github.io/KameAR/

KameAR pone un robot [Kame32](https://github.com/JavierIH/kame32) en tu habitación. Usa su modelo 3D y las marchas de su firmware, con físicas reales: camina por el suelo, choca con las paredes y los muebles, y se programa con una tira de fichas al estilo de los robots educativos.

## Dispositivos

| Dispositivo | Qué ofrece |
|---|---|
| Meta Quest 3 | Realidad aumentada con las manos: panel en la palma, coger y empujar el robot, oclusión con el entorno |
| Android (Chrome con ARCore) | Realidad aumentada: el robot sobre el suelo o una mesa, con el panel de fichas flotando sobre él |
| iPhone y iPad | Realidad aumentada a través de [Variant Launch](https://launch.variant3d.com/): Safari muestra una tarjeta que abre la página en un visor con AR. Sin oclusión con el entorno |
| Escritorio | Modo 3D, sin realidad aumentada, con la programación y las físicas completas |

## Uso

1. Abre la página y pulsa **Entrar en AR** (en escritorio se usa directamente en 3D). En iPhone, acepta antes la tarjeta de Variant Launch que aparece al abrir la página.
2. Apunta al suelo o a una mesa y toca (o pellizca, en el visor) para colocar el robot.
3. Toca las fichas del panel para crear el programa y pulsa **▶ Ejecutar**.

El panel se esconde al ejecutar; toca el robot para que vuelva. El botón **?** abre la ayuda con las instrucciones de tu dispositivo.

En las Quest:

- Vuelve hacia ti la palma de la mano no dominante para ver el panel sobre ella, y pulsa las fichas con el índice de la otra mano.
- Pellizca junto al robot con la mano dominante para cogerlo. Las dos manos lo empujan.
- Un pellizco con la mano no dominante sale de la realidad aumentada.

### Fichas

| Grupo | Fichas |
|---|---|
| Mover | Avanzar, retroceder, girar a la izquierda, girar a la derecha |
| Acciones | Saludar, saltar, bailar, flexiones |
| Control | Repetir, cerrar, esperar, si hay obstáculo, velocidad |

El [manual de programación](MANUAL.md) explica cada ficha con ejemplos.

## Desarrollo

Todo el proyecto es un único `index.html` con su CSS y su JavaScript. No hay compilación ni dependencias que instalar: las bibliotecas ([Three.js](https://threejs.org/), [three-mesh-bvh](https://github.com/gkjohnson/three-mesh-bvh) y [Rapier](https://rapier.rs/)) y las piezas STL del Kame32 se cargan por CDN.

Para probarlo en local basta un servidor de archivos estáticos:

```bash
python -m http.server 8765
```

Parámetros de la dirección:

| Parámetro | Efecto |
|---|---|
| `?test` | Ejecuta el autotest y muestra el resultado |
| `?env` | Añade un entorno simulado (un muro y un escalón) para probar sin dispositivo |

El diseño y el estado de cada fase están en [PLAN.md](PLAN.md).

# License
This robot is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).

![CC BY-SA 4.0](https://creativecommons.org/wp-content/themes/vocabulary-theme/vocabulary/svg/cc/license_badges/big/by_sa.svg)
