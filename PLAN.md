# KameAR — Plan

Aplicación de realidad aumentada en WebXR para móviles y Meta Quest 3 con el robot cuadrúpedo [Kame32](https://github.com/JavierIH/kame32): su modelo 3D, sus marchas reales, físicas, consciencia del entorno y programación por bloques al estilo de los robots educativos.

## 1. Objetivos

- **Quest 3:** al voltear la mano no dominante aparece un panel para programar los movimientos del robot. Con la mano dominante se puede coger y empujar al robot.
- **Móviles (Android e iOS):** el panel de programación flota sobre el propio robot.
- **Físicas reales:** el robot camina por la fricción de sus patas, puede tropezar, volcar y ser empujado.
- **Consciencia del entorno:** el robot se apoya en el suelo y las mesas reales, choca con los muebles y queda oculto tras ellos donde el dispositivo lo permita.

## 2. Lo que aporta el repositorio kame32

- **Modelo 3D:** piezas sueltas en STL (`3d/stl/`) y el ensamblaje en FreeCAD (`3d/freecad/kame32.FCStd`). No hay un modelo articulado listo para web; hay que construirlo.
- **Movimiento:** 8 servos (2 por pata). Cada marcha es una tabla de periodo, amplitud, offset y fase por servo aplicada a un oscilador sinusoidal (`code/lib/Octosnake`):
  `ángulo = A·sin(2πt/T + φ) + offset`
- **Marchas** (`code/lib/Kame/kame.cpp`): `walk`, `backward`, `turnL`, `turnR`, `run`, `omniWalk`, `moonwalkL`, `dance`, `upDown`, `pushUp`, `frontBack`, `hello`, `jump`, `home`, `zero`.
- **Licencia:** CC BY-SA 4.0. Obliga a citar al autor (JavierIH), cosa que la página hará de forma visible, y a publicar cualquier modelo derivado con la misma licencia.

## 3. Formato del proyecto

Toda la aplicación es **un único archivo `index.html` autocontenido**: el HTML, el CSS (`<style>`) y el JavaScript (`<script type="module">`) van dentro del mismo archivo. No hay paso de compilación, ni `npm`, ni TypeScript.

- **Bibliotecas:** se cargan de forma externa desde un CDN mediante un `importmap`, con la versión fijada.
- **Modelo 3D:** las piezas STL se cargan de forma externa desde el repositorio original (vía CDN, con el commit fijado) y se ensamblan en el navegador. No se incrusta geometría en el HTML.
- **Lenguaje:** JavaScript moderno (módulos ES) con comentarios JSDoc para los tipos.
- **Uso:** basta con servir o publicar ese archivo. WebXR exige HTTPS (o `localhost`), así que no funciona abriéndolo con `file://`.

## 4. Stack

| Área | Elección | Motivo |
|---|---|---|
| Render y XR | Three.js desde CDN + WebXR (`immersive-ar`) | Un solo código para Quest 3 y móvil |
| Carga del modelo | `STLLoader` de Three.js sobre los STL del repositorio kame32 | Evita incrustar o alojar el modelo |
| Físicas | Rapier (`rapier3d-compat` desde CDN, con el WASM embebido en el módulo) | Articulaciones con motor; se carga con un solo `import` |
| Interfaz 3D | Paneles propios dibujados en canvas y usados como textura | Sin dependencias extra; funciona en la mano y sobre el robot |
| Pruebas | Autotest dentro de la página (`index.html?test`) + Immersive Web Emulator | Sin herramientas de compilación ni de test externas |
| Despliegue | Hosting estático con HTTPS y dominio público (p. ej. GitHub Pages) | Requisito de WebXR y de los App Clips de iOS |

## 5. Arquitectura

Los módulos son **secciones delimitadas dentro del `<script type="module">`** de `index.html`, cada una con su cabecera de comentario y sin dependencias circulares.

| Sección | Responsabilidad |
|---|---|
| `gait` | Port directo de `Octosnake` y `Kame` a JavaScript. Mismas tablas; produce 8 ángulos por frame. |
| `robot` | Carga los STL y los ensambla según una tabla de articulaciones (posición y eje de cada servo) definida en el código. El paralelogramo de cada pata se resuelve solo en lo visual. |
| `physics` | Cuerpo articulado en Rapier con motores de posición que siguen los ángulos de `gait`. |
| `platform` | Cada dispositivo declara sus capacidades (planos, malla, profundidad, manos, anclajes). El resto del código pregunta por capacidades, no por dispositivo. |
| `environment` | Convierte el entorno real en colisionadores y oclusión. |
| `program` | Modelo de bloques, intérprete paso a paso y resaltado del bloque activo. |
| `ui` | Un mismo panel de programación con dos anclajes: palma (Quest) o sobre el robot (móvil). |
| `input` | Manos en Quest, toque en pantalla en móvil. |
| `selftest` | Comprueba las marchas contra los valores del firmware cuando la URL lleva `?test`. |

## 6. Plataformas

### Meta Quest 3

- `hand-tracking` para el panel en la palma, coger y empujar.
- `mesh-detection` y `plane-detection`: la malla de la habitación pasa a ser colisionador estático.
- `depth-sensing` para ocultar el robot tras muebles reales.
- `anchors` para fijar su posición.

### Android (Chrome + ARCore)

- `hit-test` para colocar el robot.
- Planos detectados como suelo y mesas; si no hay planos, suelo inferido en el punto de colocación.
- `depth-sensing` para oclusión donde esté disponible.
- **Por verificar en dispositivo:** soporte real de `plane-detection` en Chrome Android.

### iOS

Safari en iPhone no tiene WebXR nativo. Se usan dos niveles:

| Nivel | Qué es | Qué da |
|---|---|---|
| **A. Puente WebXR por App Clip** (principal) | Un App Clip abre la web y expone la API WebXR sobre ARKit, sin instalar una app. Candidatos: Variant Launch y Needle Go. | El mismo código que en Android: sesión AR, hit-test, planos |
| **B. Modo 3D sin AR** (reserva) | El robot sobre un suelo virtual, con cámara orbital | Programación y físicas completas, sin entorno real. Sirve también en escritorio. |

- **Entorno en iOS:** previsiblemente solo planos y hit-test. Sin profundidad no hay oclusión: el robot colisiona con suelo y mesas pero se dibuja por delante de los muebles.
- **Descartado:** AR propio con cámara y SLAM en JavaScript (8th Wall cerró su plataforma alojada y la versión abierta, según lo publicado, no incluye SLAM).
- **Por comprobar en la fase 0:**
  - Si Needle Go funciona con Three.js puro o solo con Needle Engine.
  - Si los anclajes funcionan en el puente elegido.
  - Precio y condiciones de licencia reales del proveedor.

## 7. Interacción

### Quest 3

- **Mano no dominante:** el panel aparece cuando la normal de la palma apunta hacia la cabeza (con histéresis para que no parpadee) y queda anclado a la palma.
- **Mano dominante:**
  - Pulsa las fichas con el índice.
  - El pellizco coge el robot mediante una unión con muelle.
  - Dedos y palma tienen colisionadores para empujarlo.
- Ajuste de mano dominante, y mandos como alternativa.

### Móvil

- Panel flotante sobre el robot, siempre orientado a la cámara, que se toca por raycast desde la pantalla.
- Arrastrar el robot lo coge; un toque rápido lo empuja.

## 8. Programación por bloques

El modelo es el de Bee-Bot o ScratchJr: el programa es una **tira de fichas con iconos** que se lee de izquierda a derecha. No hay texto que escribir ni piezas que encajar.

```
┌──────────────────────────────────────┐
│ ▶ Ejecutar   ⏸ Paso   ■ Parar   🗑   │  controles
├──────────────────────────────────────┤
│ [↑ 3] [↻] [↑ 2] [🔁 2 ( [👋] [↑ 1] )] │  programa
├──────────────────────────────────────┤
│ Mover:    ↑  ↓  ↺  ↻                 │  paleta
│ Acciones: 👋  ⤒  💃  💪               │
│ Control:  🔁  ⏱  🚧  🐢/🐇            │
└──────────────────────────────────────┘
```

### Uso

- **Añadir:** tocar una ficha de la paleta la pone al final del programa. No se arrastra nada, porque arrastrar con el dedo en el aire es impreciso.
- **Editar:** tocar una ficha del programa la selecciona y muestra `−` / `+` para su número (1–9), flechas para moverla y un botón de borrar.
- **Repetir:** la ficha 🔁 abre un paréntesis. Las fichas siguientes entran dentro hasta que se toca «cerrar». Se dibuja como un corchete de color que abraza a las fichas.
- **Sensor:** 🚧 «si hay obstáculo» funciona igual que repetir: abraza las fichas que solo se ejecutan si el robot tiene algo delante (raycast contra el entorno).
- **Ejecución:** la ficha activa se ilumina. «Paso» ejecuta una sola ficha, para depurar.

### Fichas

| Color | Fichas | Marcha del firmware |
|---|---|---|
| Azul (mover) | Avanzar N, retroceder N, girar izquierda, girar derecha | `walk`, `backward`, `turnL`, `turnR` |
| Naranja (acciones) | Saludar, saltar, bailar, flexiones | `hello`, `jump`, `dance`, `pushUp` |
| Amarillo (control) | Repetir N, esperar, si obstáculo, velocidad | — |

### Reglas de diseño

- **Giros de 90°:** con física real, un número fijo de pasos no da un giro exacto. La ficha de giro ejecuta la marcha hasta que el rumbo medido cambia 90°.
- **Sin errores de sintaxis:** cualquier tira es un programa válido; un paréntesis sin cerrar se cierra solo al final.
- **Tamaño:** fichas de al menos 2 cm en el panel de la palma, 4 por fila, con salto de fila automático.
- **Niveles:** el nivel 1 solo tiene mover y acciones, el 2 añade repetir y el 3 añade el sensor.
- **Persistencia:** el programa se guarda en `localStorage`.
- **Por qué no Blockly:** exige arrastrar con precisión y leer texto pequeño; ambas cosas funcionan mal con manos en XR y con un panel sobre un robot de 15 cm.

## 9. Físicas

Se intenta primero la **física articulada real**.

- **Modelo:** cuerpo más 8 articulaciones de revolución con motor de posición, una por servo.
- **Servos:** par máximo y velocidad limitados según un microservo real (SG90/MG90S).
- **Masas:** calculadas del volumen de los STL más el peso de servos, placa y batería.
- **Estabilidad:** mundo físico a escala ×10, paso fijo con subpasos y detección continua de colisiones en los pies.
- **Fricción:** el coeficiente de los pies es el parámetro principal de ajuste.

### Colisiones entre piezas (ya implementado)

Las piezas del robot se tratan como sólidos: no pueden atravesarse.

- **Topes mecánicos:** cada servo tiene un intervalo permitido, medido cruzando las mallas reales triángulo a triángulo (BVH). Para la pata delantera izquierda: cadera 64–180°, pata 16–152°; las demás son simétricas. Fuera de ese intervalo la pata choca consigo misma o con el cuerpo.
- **Choques entre patas:** se comprueban en cada movimiento con las mallas reales; las patas implicadas se detienen justo antes del contacto.
- **Uso en la fase 2:** esos mismos topes serán los límites de las articulaciones en Rapier.

### Criterios de aceptación

1. Avanza 10 ciclos de `walk` sin desviarse más de 15°.
2. Gira con `turnL`/`turnR` y completa `jump` y `hello` sin volcar.
3. Soporta que lo cojan y lo suelten desde 30 cm sin que la simulación se rompa.
4. Mantiene 72 fps en Quest 3.

### Resultado de la fase 2 (medido en escritorio)

Implementado con Rapier: 13 sólidos (cuerpo y, por pata, soporte, pata y pie), 8 articulaciones motorizadas y, por pata, un paralelogramo real (rótula pata–pie y giro del pie bloqueado respecto al soporte), paso de 1/240 s. Los colisionadores son envolventes convexas simplificadas (40 vértices), lo que da unas 17 veces el tiempo real en escritorio.

| Criterio | Resultado |
|---|---|
| 1. `walk` recto | Cumplido: 586 mm en 10 ciclos, desvío de 2° |
| 2. Giros, `hello` y `jump` | Cumplido: `turnL` gira 163° en 4 ciclos y `jump` avanza unos 37 mm y cae de pie |
| 3. Caída desde 30 cm | Cumplido: aterriza de pie |
| 4. 72 fps en Quest 3 | Sin medir (60 fps en escritorio) |

Sobre `jump`: la primera versión volcaba hacia atrás por un defecto del modelo (el pie iba pegado a la pata y se recolocaba a mano en cada paso, lo que falseaba la palanca). Con el paralelogramo real el resultado depende del centro de masas del cuerpo: centrado y alto queda al límite del vuelco; con el valor adoptado, 10 mm adelantado y 8 mm de alto, cae de pie con margen (comprobado con 8–32 iteraciones, paso de 1/480 s, fricción 0,4–1,2 y par de 1,3 kg·cm). Ese centro de masas es una estimación: conviene medirlo en el robot real.

Parámetros elegidos sin medir en un robot real: masas (cuerpo 150 g, soporte con servo 20 g, pata 8 g, pie 7 g), fricción de los pies (0,9) y rigidez del servo. El par (1,8 kg·cm) y la velocidad (0,1 s/60°) son los de un SG90.

### Plan B

Solo si no se cumplen los criterios: cuerpo rígido con velocidad calibrada por marcha y patas únicamente animadas.

## 10. Fases

| Fase | Contenido | Hecho cuando… |
|---|---|---|
| 0 | `index.html` base con las bibliotecas por CDN, despliegue HTTPS, robot ensamblado a partir de los STL (tabla de articulaciones medida en el FreeCAD) y prueba del puente iOS | El modelo se ve en el navegador con las 8 articulaciones movibles y una escena AR mínima arranca en un iPhone |
| 1 | Port de las marchas + tests contra los valores del firmware | Todas las marchas se reproducen en el modo 3D |
| 2 | Física articulada real | Se cumplen los 4 criterios de aceptación |
| 3 | AR y entorno con la capa `platform/` (Quest, Android, iOS) | Camina por el suelo real en los tres dispositivos y choca con los muebles donde hay malla o planos |
| 4 | Tira de bloques e intérprete, panel sobre el robot | Un programa con repetir se ejecuta completo en Android e iPhone |
| 5 | Manos en Quest: panel en la palma, coger y empujar | Se programa y manipula el robot solo con las manos |
| 6 | Rendimiento, ayuda inicial y publicación | Probado en Quest 3, Android e iPhone |

### Estado de la fase 3

Implementado y comprobado en escritorio (autotest y entorno simulado con `?env`); **falta probarlo en los dispositivos**.

- **Capacidades:** la sesión pide como opcionales planos, malla, anclajes, manos y panel sobre la imagen; en el visor, también profundidad. El panel muestra qué ha concedido el dispositivo.
- **Colisiones con el entorno:** cada plano o malla detectado se añade a la simulación como malla fija, y se actualiza unas dos veces por segundo.
- **Suelo:** solo con planos (móviles) se conserva el suelo plano del punto de colocación, porque los planos tienen huecos. Con malla (Quest) el robot se apoya en ella y puede caerse de una mesa; si cae al vacío vuelve al punto de colocación.
- **Oclusión:** las paredes y planos verticales tapan al robot; el suelo y las mesas no, para que un error de altura no le tape los pies. En el visor la oclusión general es por profundidad.
- **Visor sin panel:** al colocar el robot echa a andar solo, para poder probar el entorno hasta que llegue el panel de la palma (fase 5).
- **iOS:** el puente queda preparado (`IOS_BRIDGE.variantLaunchKey` en `index.html`); sin clave de Variant Launch el iPhone sigue en modo 3D.
- **Pendiente:** anclajes (el robot no se reancla si el seguimiento deriva) y medir el rendimiento en Quest con la malla de la habitación.

Añadido tras las pruebas en Quest: el pellizco de la mano izquierda sale de la AR; colocado en una pared, el robot cae hasta lo que haya debajo; la malla azul y el rótulo flotante son opciones del panel, desactivadas por defecto.

### Estado de la fase 4

Implementado y comprobado en escritorio (autotest y uso con ratón); **falta probarlo en los dispositivos**.

- **Panel de fichas:** un plano dibujado en un lienzo que flota sobre el robot, de cara a la cámara y con tamaño aparente constante. Se toca con el ratón o el dedo en modo 3D, con un toque en la pantalla en AR y, en el visor, apuntando con la mano o el mando derecho y pellizcando (se dibuja el rayo y el punto de contacto).
- **Fichas:** avanzar, retroceder, girar a izquierda y derecha, saludar, saltar, bailar, flexiones, repetir, cerrar, esperar, si hay obstáculo y velocidad. Hasta 18 fichas, en dos filas.
- **Edición:** tocar una ficha de la paleta la añade; tocar una del programa permite cambiar su número, moverla o quitarla.
- **Ejecución:** ejecutar, paso a paso, parar y borrar. La ficha activa se ilumina.
- **Giros:** la ficha de giro sigue la marcha hasta que el rumbo medido cambia 90° por unidad (medido: 90° ±12°).
- **Avance:** cada unidad de avanzar o retroceder son 2 ciclos de la marcha, unos 11 cm.
- **Sensor:** «si hay obstáculo» lanza un rayo de 25 cm hacia delante contra el entorno.
- **Persistencia:** el programa se guarda en `localStorage`.
- **Colocación en AR:** con el robot colocado, los toques solo actúan sobre el panel; el botón «Mover» permite recolocarlo.
- **Pendiente:** los niveles (1 a 3) y el corchete de color que abraza las fichas repetidas (ahora se marca con paréntesis).

## 11. Riesgos

| Riesgo | Mitigación |
|---|---|
| Física a escala pequeña inestable (el robot mide unos 15 cm) | Escala ×10, subpasos, criterios de aceptación y plan B |
| Modelo 3D: la posición y el eje de giro de cada servo hay que sacarlos del FreeCAD y pasarlos a una tabla en el código | Es la primera tarea de la fase 0 |
| Dependencia de CDN: sin conexión, o si cambia una biblioteca o el repositorio kame32, la aplicación no carga | Versiones y commit fijados en las URL; mensaje de error claro si falla una carga |
| Un único archivo grande y difícil de mantener | Secciones delimitadas con cabecera, orden fijo y sin dependencias circulares |
| Sin compilación no hay comprobación de tipos ni tests automáticos externos | JSDoc y autotest en la página (`?test`) |
| Puente WebXR de iOS sin validar (capacidades, licencia, compatibilidad con Three.js) | Prueba en iPhone real en la fase 0; modo 3D como reserva |
| `plane-detection` en Chrome Android no garantizado | Suelo inferido desde el hit-test |
| Rendimiento en Quest con malla de habitación y físicas | Simplificar la malla para colisión; medir desde la fase 3 |

## 12. Extras posteriores

- Enviar el mismo programa al Kame32 físico: el firmware ya expone un servidor HTTP con `/button` y `/joystick`.

## 13. Dispositivos necesarios para probar

- Meta Quest 3.
- Un móvil Android con ARCore.
- Un iPhone (imprescindible para validar el puente de la fase 0).
