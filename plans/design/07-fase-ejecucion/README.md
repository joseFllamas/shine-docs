# Fase de ejecución — 10 ejercicios (mirror del diseño)

Copia local fiel del diseño visual definitivo de la fase de ejecución de los 10
ejercicios tipo. Es **referencia**, no implementación: aquí no hay código React
Native todavía (ver "Cómo se implementa" al final).

## Procedencia

- **Fuente**: canvas de claude.ai/design.
  - Proyecto: `3681df27-d335-4d2d-bc44-f4f78b3f4f2c`
  - Archivo: `Ejercicios tipo - Fase de ejecución.dc.html`
  - URL: https://claude.ai/design/p/3681df27-d335-4d2d-bc44-f4f78b3f4f2c?file=Ejercicios+tipo+-+Fase+de+ejecuci%C3%B3n.dc.html
- **Spec de origen** (el prompt con el que se generó): [../07-ejercicios-nuevos.md](../07-ejercicios-nuevos.md).
- **Design system**: usa los tokens del proyecto "Shine Design System"
  (`eabf0011-...`), ya portados a `shine-app/src/theme/`.

## Archivos

| Archivo | Qué es |
|---|---|
| `Ejercicios tipo - Fase de ejecución.dc.html` | Original **verbatim** del canvas (fuente de verdad). Solo renderiza dentro de claude.ai/design. |
| `preview.html` | Mirror **autocontenido**: se abre en cualquier navegador. Portados a HTML estándar los tags del canvas (`x-import` Button → `<button>`, `dc-import` Keyboard/Keypad → markup inline del DS, `sc-for` de la matriz → celdas expandidas con los datos reales). Tokens embebidos + Nunito de Google Fonts + iconos Lucide por CDN. |
| `tokens.css` | Copia congelada de los tokens del DS (colores, tipografía, espaciado, radios, sombras) para que `preview.html` no dependa de claude.ai. |

> Para verlo: abrir `preview.html` en el navegador (con conexión para Nunito e
> iconos Lucide; sin conexión cae a system-ui y los iconos no aparecen, el resto
> del layout se ve igual).

## Reglas comunes a las 10 láminas

Cada lámina muestra la ejecución en **3 estados**: `Inicial` · `En interacción` · `Acierto`.

- Pantalla de **foco total** sin navegación; solo una `x` (44px) arriba a la izquierda.
- Marco de móvil 376×764, `border-radius:40px`, sombra suave.
- Elementos táctiles de **48px** mínimo.
- **Feedback amable**: acierto en verde suave (`--success` / `--color-success-tint`),
  nunca rojo. El error/diferencia se muestra en **ámbar** (`--accent`).
- **Progreso discreto** abajo: barra ámbar + "N / 10".
- Acierto: círculo verde con `check`, titular corto con 🌟 y, a veces, una línea de refuerzo.
- CTA principal (botón teal pill del DS): `Comprobar` durante la ejecución, `Siguiente`
  al acertar (`Terminar sesión` en el último).

## Ejercicios

Zona objetivo de datos = qué necesitará el modelo Drupal cuando se implemente.
"Audio" marca dependencia de locución/TTS (bloquea v1, va a v1.1).

### 7.1 Segmentación silábica — *conciencia fonológica*
Dividir una palabra en sílabas tocando entre las letras (inserta un corte teal), o
contar con palmadas (botón alternativo). CTA `Comprobar` deshabilitado hasta el primer corte.
- **Inicial**: palabra grande (`frigorífico`, 46px, letter-spacing 8px), tres marcas de corte tenues, botón "Contar con palmadas".
- **Interacción**: cortes teal insertados entre sílabas (`fri | go | rífico`), chip "2 cortes" con icono `scissors`.
- **Acierto**: sílabas en fichas alternando teal/ámbar (`fri go rí fi co`), "¡Muy bien! · 5 sílabas exactas".
- **Datos**: palabra + posiciones de corte correctas. Sin audio obligatorio.

### 7.2 Omisión silábica — *conciencia fonológica* · **Audio**
Escuchar una palabra y decir cómo queda al quitar una sílaba. Dos variantes: escritura
libre (teclado del DS) y selección múltiple (modo fácil).
- **Inicial (escritura libre)**: botón altavoz repetible, palabra con la sílaba a quitar resaltada en ámbar (`[ca]miseta`), campo de texto vacío, `Keyboard` del DS.
- **Interacción (selección múltiple)**: altavoz grande, 3 opciones (`miseta` ✓ / `camita` / `seta`), la elegida en teal.
- **Acierto**: `camiseta − ca` → `miseta` en verde, "¡Bien hecho!".
- **Datos**: audio de la palabra, sílaba omitida, respuesta correcta, distractores (modo fácil).

### 7.3 El intruso rímico — *conciencia fonológica* · **Audio**
Encontrar la palabra que no rima. Tocar una tarjeta reproduce su audio; segundo toque/mantener pulsado la marca como intruso.
- **Inicial**: instrucción con altavoz "¿Cuál no rima?", cuadrícula 2×2 de tarjetas imagen+palabra (`gato`, `pato`, `plato`, `sol`).
- **Interacción**: tarjeta activa con badge de altavoz (teal), intruso marcado con borde ámbar punteado + pill "intruso".
- **Acierto**: las que riman se agrupan en un panel verde; el intruso (`sol`) se aparta con borde ámbar "no rima".
- **Datos**: set de palabras con imagen y audio; cuál es el intruso.

### 7.4 Dictado de pseudopalabras — *conciencia fonológica* · **Audio**
Escribir una palabra inventada que se escucha. Sin autocorrector. Comparación letra a
letra en positivo al comprobar.
- **Inicial**: altavoz muy grande (88px) repetible "sin límite", campo vacío, "sin corrector ni sugerencias", `Keyboard`.
- **Interacción**: altavoz pequeño "escuchar otra vez", campo con texto en curso (`badu|`, 30px, letter-spacing 6px), `Keyboard`.
- **Acierto**: comparación por celdas (`b a d u f o`) todas en verde; leyenda "acertada / distinta (nunca rojo)".
- **Datos**: audio de la pseudopalabra + su grafía objetivo. Comparación carácter a carácter (verde = igual, ámbar = distinta).

### 7.5 Segmentación léxica en oraciones — *fluidez lectora*  ← **v1 (lote 1)**
Separar una frase escrita sin espacios tocando entre letras (inserta espacio teal que
desaparece al validar). Sin audio.
- **Inicial**: instrucción, frase continua (`hoyymañanairemosajugar`, 34px, `word-break:break-all`), CTA deshabilitado.
- **Interacción**: espacios teal insertados; parte ya separada en negro, resto en gris; chip "3 espacios puestos".
- **Acierto**: palabras en fichas teal (`hoy y mañana iremos a jugar`), "¡6 palabras! · Separaste la frase entera".
- **Datos**: frase objetivo con las posiciones de espacio correctas.

### 7.6 Matriz de discriminación visual (cancelación) — *discriminación visual*  ← **v1 (lote 1)**
Encontrar todas las letras objetivo en una cuadrícula 5×8 de letras confundibles
(p/q/b/d/g). Temporizador sutil en ámbar (barra, sin cuenta atrás grande).
- **Inicial**: "Encuentra todas las **p**" (chip teal), barra de tiempo llena, cuadrícula neutra.
- **Interacción**: barra de tiempo a media, celdas encontradas marcadas en teal, "3 encontradas".
- **Acierto**: "¡Encontraste 7 de 8!", las encontradas en teal con badge `check`; la que faltó se revela con **aro ámbar** (sin mensaje de fallo).
- **Datos**: grid de caracteres, letra objetivo, índices objetivo, tiempo. Precisión = encontradas/total. Sin audio.

### 7.7 Ortografía con código de colores — *ortografía*
Completar el hueco eligiendo entre dos letras, cada una con un **color consistente en
toda la app** (p. ej. `j` roja suave, `g` verde suave). Imagen de apoyo opcional.
- **Inicial**: imagen ("un cojín"), palabra con hueco punteado (`co _ ín`, 52px), dos botones enormes de color (`j` rojo / `g` verde).
- **Interacción**: la letra elegida "vuela" al hueco con su color; botón elegido con sombra de color, el otro atenuado.
- **Acierto**: palabra completa con la letra en su color (`co`**`j`**`ín`), "La j siempre es de este color".
- **Datos**: palabra, posición del hueco, opciones con su color fijo, imagen opcional. Nota: introduce un mapa **letra→color** global nuevo en el DS.

### 7.8 Reordenamiento de sílabas — *ortografía*  ← **v1 (lote 1)**
Ordenar sílabas desordenadas para formar la palabra. Fichas arrastrables sobre huecos
punteados; alternativa sin drag: tocar en orden. Imagen de apoyo opcional.
- **Inicial**: imagen con "pista", 3 huecos punteados, fichas desordenadas (`lon`, `pan`, `ta`) con sombra elevada.
- **Interacción**: `pan` colocado en el primer hueco (teal); ficha en arrastre inclinada con sombra; huecos restantes punteados.
- **Acierto**: sílabas colocadas alternando teal/ámbar (`pan ta lon`), "pantalón 🌟".
- **Datos**: sílabas correctas en orden + barajadas, imagen opcional. Sin audio obligatorio.

### 7.9 Dictado inverso — *memoria de trabajo* · **Audio**
Escuchar una serie y repetirla al revés. Fase de escucha casi vacía; fase de respuesta
con teclado numérico propio (`Keypad`, botones 56px) y huecos.
- **Inicial (escucha)**: "Escucha con atención…", círculo teal 150px que pulsa con icono `audio-lines`, "luego lo repetirás al revés". Sin números visibles.
- **Interacción (respuesta)**: "Escríbelo al revés", huecos (`9` puesto, siguiente activo, tercero punteado), botón "deshacer", `Keypad`.
- **Acierto**: "escuchaste 5 · 2 · 9" → `9 2 5` en verde, "¡Al revés, perfecto! · Buena memoria".
- **Datos**: serie de estímulos (audio), longitud, respuesta = inversa. Variante con palabras (tarjetas en vez de teclado).

### 7.10 Instrucciones encadenadas — *memoria de trabajo* · **Audio**
Escuchar una orden de varios pasos y ejecutarla sobre una escena interactiva. Objetos
que responden a toque/arrastre. Puntos de progreso por paso.
- **Inicial**: altavoz "Escucha la orden antes de empezar", "paso 0 de 3" (3 puntos vacíos), escena con objetos separados (perro, lápiz, caja, círculo azul, manzana).
- **Interacción**: banner con el paso actual ("Toca el perro y llévalo a la caja"), "paso 2 de 3" (2 puntos verdes), objeto en arrastre con traza punteada hacia el destino punteado.
- **Acierto**: escena en verde, "¡Los 3 pasos! · Seguiste toda la orden sin perderte", CTA `Terminar sesión`, barra 10/10.
- **Datos**: escena (objetos + posiciones), lista ordenada de pasos (audio), condición de acierto por paso.

## Cómo se implementa (después)

El diseño es la entrada de la **FASE 13** del plan
([../../02-plan-ejecucion-v1.md](../../02-plan-ejecucion-v1.md)):

- **v1 (lote 1, sin audio)**: 7.5 segmentación léxica, 7.6 matriz, 7.8 reordenamiento.
  Prompts: `plans/prompts/13a-ejercicio-matriz.md`, `13b`, `13c`.
- **v1.1 (dependen de audio/TTS)**: 7.1–7.4, 7.7, 7.9, 7.10.

Cada ejercicio necesita: node type + bundles log/summary en Drupal, tipos TS espejo,
un Player en `shine-app/src/components/activities/`, rama en `ActivityRouter` y
`tracking`. Reutilizar los componentes del DS ya portados en `src/components/ui/`.
