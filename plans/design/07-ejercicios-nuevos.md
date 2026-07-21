# Prompt 07: Players de los nuevos ejercicios tipo

> Usa el design system de Shine ya definido en este proyecto (ver 00-sistema-diseno.md).
> Fuente: "ejercicios - tipo.docx" (10 ejercicios basados en evidencia para dislexia).
> Todos comparten las fases intro y completada de los prompts 04 y 05. Aquí se diseña solo la fase de ejecución de cada uno.

Reglas comunes a todos: pantalla de foco total sin navegación, elementos táctiles de 48px mínimo, feedback de fallo siempre amable (ámbar suave, "Casi. Sigamos."), feedback de acierto en verde suave con micro-animación, progreso discreto abajo ("3 / 10").

---

## 7.1 Segmentación silábica

El usuario divide una palabra en sílabas.
- Palabra grande centrada (32sp), por ejemplo "frigorífico".
- El usuario toca entre dos letras para insertar un corte; aparece un separador visual (barra teal redondeada) con animación suave.
- Alternativa de interacción: botón grande de "palmada" para contar sílabas tocándolo tantas veces como sílabas haya, con contador visible.
- Botón "Comprobar" cuando hay al menos un corte.
- Acierto: las sílabas se separan con animación y se colorean alternando dos tonos suaves.

## 7.2 Omisión silábica

El usuario escucha una palabra y escribe cómo queda al quitarle una sílaba.
- Botón de altavoz grande para escuchar la instrucción (se puede repetir sin límite y sin penalización).
- La palabra original visible en grande con la sílaba a eliminar resaltada en ámbar.
- Campo de texto grande (24sp) con teclado, o selección entre 3 opciones en modo fácil.
- Diseñar ambas variantes: escritura libre y selección múltiple.

## 7.3 El intruso rímico

Encontrar la palabra que no rima.
- Cuadrícula de 3 o 4 tarjetas grandes, cada una con imagen y palabra debajo.
- Tocar una tarjeta reproduce su audio; mantener pulsado (o segundo tap en un icono) la selecciona como intruso.
- Instrucción arriba con botón de altavoz: "¿Cuál no rima?".
- Acierto: las tarjetas que riman se agrupan visualmente y el intruso se aparta con animación simpática.

## 7.4 Dictado de pseudopalabras

Escribir una palabra inventada que se escucha.
- Botón de altavoz muy grande centrado (se puede repetir).
- Campo de escritura grande con letras de 28sp y espaciado amplio.
- Sin autocorrector ni sugerencias del teclado.
- Al comprobar, mostrar la comparación letra a letra en positivo: las letras correctas en verde suave, las diferentes en ámbar (nunca rojo), con mensaje "¡Casi! Era así: badufo".

## 7.5 Segmentación léxica en oraciones

Separar una frase escrita sin espacios.
- Texto continuo grande y con mucho interlineado: "hoyymañanairemosajugar".
- Tocar entre dos letras inserta un espacio con animación (el texto se reacomoda suavemente).
- Los espacios insertados se marcan con una barrita teal que luego desaparece al validar.
- Botón "Comprobar". Acierto parcial: se fijan los espacios correctos y se invita a revisar el resto.

## 7.6 Matriz de discriminación visual (cancelación)

Encontrar todas las letras objetivo en una cuadrícula.
- Instrucción arriba: "Encuentra todas las letras p", con la letra objetivo en un chip grande.
- Cuadrícula de 5x8 a 5x10 celdas con letras visualmente confundibles (p, q, b, d, g), cada celda de 48px mínimo.
- Tocar una celda la marca con un círculo teal; volver a tocar la desmarca.
- Temporizador: barra de tiempo sutil arriba que se vacía, en ámbar (nunca números en cuenta atrás grande, que estresan).
- Al acabar: se revelan las que faltaban con un aro suave, sin mensaje de fallo.

## 7.7 Ortografía con código de colores

Completar el hueco de una palabra eligiendo entre dos letras asociadas a colores.
- Palabra grande con hueco: "co_ín".
- Dos botones enormes debajo, cada uno de un color consistente en toda la app para esa letra (por ejemplo j siempre roja suave, g siempre verde suave).
- Al elegir bien, la letra vuela al hueco y la palabra completa se muestra con la letra en su color.
- Incluir imagen opcional de apoyo (un cojín) sobre la palabra.

## 7.8 Reordenamiento de sílabas

Ordenar sílabas desordenadas para formar una palabra.
- 3 o 4 fichas grandes arrastrables con sílabas ("lon", "pan", "ta"), estilo piezas redondeadas con sombra.
- Línea base con huecos punteados donde soltarlas.
- Alternativa sin drag and drop (accesibilidad): tocar las fichas en orden y se van colocando solas.
- Imagen de apoyo opcional (un pantalón difuminado) como pista activable.

## 7.9 Dictado inverso (memoria de trabajo)

Escuchar una serie y repetirla al revés.
- Fase escucha: pantalla casi vacía, un icono de oreja u onda que pulsa mientras suenan los estímulos ("5... 2... 9"). Sin texto visible de los números.
- Fase respuesta: teclado numérico grande de la propia app (botones de 56px) y los huecos a rellenar visibles ("_ _ _").
- Cada pulsación rellena un hueco con animación. Botón deshacer discreto.
- Variante con palabras: selección entre tarjetas en lugar de teclado.

## 7.10 Instrucciones encadenadas

Escuchar una orden de varios pasos y ejecutarla sobre una escena interactiva.
- Escena tipo ilustración plana con 4 a 6 objetos amables (perro, lápiz, caja, círculo azul...), bien separados.
- Botón de altavoz para escuchar la instrucción completa (repetible antes de empezar, no durante).
- Los objetos responden al toque y al arrastre con feedback físico suave.
- Indicador de pasos completados: puntos que se van rellenando (paso 1 de 3).

---

## Entregable

Una lámina por ejercicio (10 en total) con la fase de ejecución en estado inicial, estado en interacción y estado de acierto. Reutilizar sin excepción los componentes del design system.
