# Nebulosas

Un visor para jugar con nebulosas dibujadas enteramente con código. Parte de una reconstrucción de la nebulosa de la Laguna (M8) y permite generar otros tipos de nebulosa, cambiarles el color y la forma, moverlas y editar el campo de estrellas.

Es un único archivo HTML, sin imágenes, sin dependencias y sin paso de build. Todo se calcula en el navegador sobre un `<canvas>`.

## Cómo abrirlo

Abrí `index.html` con doble clic en cualquier navegador moderno. Tarda uno o dos segundos en calcular la luz la primera vez.

Si preferís servirlo:

```sh
python3 -m http.server 8000
# y abrí http://localhost:8000
```

## Qué se puede hacer

El panel de la derecha está ordenado de lo general a lo fino.

**Nebulosa.** La Laguna es la reconstrucción fija de una foto real. Los otros cinco tipos se generan:

| Tipo | Inspirada en | Rasgos |
| --- | --- | --- |
| Planetaria | Helix, Anillo | Cáscara roja grumosa, interior turquesa, nudos oscuros en el borde |
| Bipolar | Mariposa | Dos lóbulos desiguales con bordes brillantes y una cintura de polvo |
| Supernova | Velo | Filamentos enredados sobre una cáscara, cortados en arcos |
| Pilares | Pilares de la Creación | Columnas de polvo con el borde iluminado del lado de la luz |
| Reflexión | Pléyades | Bruma azul con estrías alrededor de un cúmulo de estrellas calientes |

"Otra variante" genera otra nebulosa del mismo tipo. Cada variante tiene un número y la misma variante siempre da la misma imagen. Los tipos generados se pueden arrastrar a otra posición con "Mover en la imagen".

**Color.** Siete paletas (Original, Hubble, Hielo, Fuego, Esmeralda, Violeta, Mono) y un control para girar el tono.

**Forma.** Deforma el gas sin tocar las estrellas: Remolino, Inflar, Turbulencia, Espejo y Caleidoscopio, con un control de intensidad.

**Estrellas.**
- Prender o apagar las estrellas originales de la Laguna.
- Sumar estrellas generadas por tipo: de fondo, tipo Sol, enanas rojas, gigantes azules y destellos con cruz de difracción.
- "Nuevo campo de estrellas" reemplaza el campo por uno generado.
- Editar con clics: el modo Agregar está activo desde el inicio, así que un clic en la imagen suma una estrella. Borrar quita la más cercana y Mirar desactiva los clics. Esc sale de cualquier modo.

**Abajo del panel**: prender o apagar las capas (gas, estrellas, grano), "Sorprendeme" para una combinación al azar, "Guardar PNG" y "Volver a la Laguna original".

## Cómo funciona

### La Laguna

No es una foto. La imagen se midió fuera del navegador y se ajustó con funciones matemáticas simples. El objeto `MODEL` al principio del script guarda el resultado:

- `gas`: 6.500 manchas gaussianas `[x, y, sigma, R, G, B]`. Muchas tienen canales negativos: restan luz para formar el polvo y se compensan entre sí.
- `stars`: 1.734 estrellas con núcleo y halo.
- `cutouts`: parches que limpian el gas debajo de las diez estrellas más brillantes.
- `brightStars`: esas diez estrellas, con su forma medida y sus rayos de difracción.

El script que hizo ese ajuste no forma parte del proyecto.

### El motor

`rasterEngine(model)` corre en un Web Worker creado a partir de su propio código. Si el navegador no permite workers, corre en la página. El motor arma capas separadas en `Float32Array`: gas, estrellas y grano. En cada dibujo solo las combina y les aplica la paleta, por eso cambiar un color es instantáneo.

Los pasos de un dibujo:

1. **Capa de gas**: la de la Laguna, o una generada por la receta del tipo elegido. Se guarda hasta que cambian el tipo, la variante o la posición.
2. **Forma**: si hay una deformación elegida, el gas se muestrea desde otra posición, píxel por píxel. Se deforma la imagen ya sumada y no las manchas, porque las manchas con signo dejarían de compensarse.
3. **Color**: una matriz de color o un degradé según el brillo, y una curva suave que evita que los brillos se quemen en blanco.
4. **Estrellas y grano** se suman encima.

### Tipos generados

Cada tipo es una receta en `recipe(type, seed)` que devuelve, para cualquier punto, cuánta luz emite en tres colores (rojo del hidrógeno, turquesa del oxígeno, azul de reflexión) y cuánto polvo hay. Las recetas usan ruido determinista, así que la misma semilla siempre da la misma nebulosa. El motor las evalúa en una grilla a media resolución, las interpola y les suma una textura fina de ruido.

### Estrellas

Todas las estrellas son una lista con el mismo formato: las de la Laguna, las generadas y las agregadas a mano.

```
[x, y, sigma, R, G, B, sigmaHalo, rHalo, gHalo, bHalo,
 largoRayo, grosorRayo, anguloRayo, rRayo, gRayo, bRayo]
```

La página arma la lista y el worker la dibuja solo cuando cambia. Las estrellas generadas salen de una secuencia por tipo, así que subir un control agrega estrellas sin mover las que ya estaban.

## Cómo extenderlo

- **Una paleta nueva**: agregala a `PRESETS` con puntos de degradé `[posición, R, G, B]`, `gamma`, `hue` o una matriz `stars`, y sumá su muestra en el panel.
- **Un tipo de nebulosa nuevo**: agregá una rama en `recipe()` que complete `d[0..3]` (rojo, turquesa, azul, polvo) y un botón con `name="type"` en el panel.
- **Un tipo de estrella nuevo**: sumalo a `STAR_KINDS` y a `makeStar()`, con su control en el panel.

## Limitaciones conocidas

- La resolución es fija: 1536 × 859.
- Si se apagan las estrellas de la Laguna, quedan manchitas tenues donde estaban. Son restos de esas estrellas dentro del gas ajustado.
- La Laguna no se puede mover, porque fuera del cuadro de la foto no hay datos.
- Algunas deformaciones fuertes estiran restos de estrellas que quedaron en el gas.

## Estructura

```
index.html                               visor completo (HTML, CSS y JS en un archivo)
referencia/nebulosa-canvas.original.html  reconstrucción original, sin modificar
```

`referencia/` guarda el archivo tal como llegó. Con todos los controles en su valor inicial, `index.html` produce exactamente la misma imagen, píxel por píxel.
