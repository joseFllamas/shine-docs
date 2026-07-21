# Prompt 04: Player de actividades (intro y ejecución)

> Usa el design system de Shine ya definido en este proyecto (ver 00-sistema-diseno.md).

---

Diseña el **player de actividades** de Shine, app de ejercicios para personas con dislexia. Es el corazón de la app: donde el usuario hace el ejercicio. Toda actividad pasa por tres fases: intro, ejecución y completada (la fase completada se diseña en el prompt 05).

Regla de oro: durante la ejecución la pantalla es un espacio de foco total. Sin cabecera, sin navegación, sin reloj visible que meta presión. Solo el ejercicio y un indicador sutil de progreso.

## Fase 1: Intro (común a todos los tipos)

Pantalla completa, contenido centrado verticalmente.

- Título de la actividad (H1, centrado).
- Chips: nivel (Inicial / Intermedio / Avanzado) y área (por ejemplo "Fluidez lectora").
- Tarjeta de explicación: borde teal suave, icono de bombilla, texto explicando qué hay que hacer, en lenguaje directo y frases cortas. Puede incluir una imagen de apoyo.
- Texto de apoyo adicional opcional (cuerpo).
- Botón primario grande abajo: "Comenzar". Al pulsarlo empieza el cronómetro (invisible para el usuario) y se pasa a la ejecución.
- Enlace discreto arriba a la izquierda para volver a la sesión.

## Fase 2a: Ejecución, texto vertical (tap para avanzar)

- Una palabra o frase enorme centrada en pantalla (28 a 36sp, Bold, carbón sobre blanco roto).
- El usuario toca en cualquier punto de la pantalla para pasar a la siguiente. Animación slide-up de 200ms.
- Contador de progreso discreto abajo: "3 / 12" o puntos.
- Enlace "Salir" muy discreto arriba a la derecha (escape accesible).

## Fase 2b: Ejecución, blink (auto-avance)

- Igual que 2a pero la palabra o sílaba cambia sola cada N segundos con transición fade.
- Antes de empezar: cuenta atrás amable de 3, 2, 1 para que el usuario se prepare.
- Indicador sutil del ritmo (un punto que pulsa), nunca una cuenta atrás numérica agresiva por palabra.

## Fase 2c: Ejecución, lectura en voz alta (audio)

- Texto completo a leer, en fuente grande (mínimo 22sp), interlineado 1.8, alineado a la izquierda, máximo 60 caracteres por línea.
- Botón circular grande de micrófono centrado abajo:
  - Reposo: círculo con borde teal, icono de micro, etiqueta "Toca para empezar a leer".
  - Grabando: círculo pulsando en ámbar suave (no rojo), etiqueta "Leyendo...", cronómetro discreto debajo en gris.
  - Al acabar: botón "He terminado" (primario) que aparece a los 2 segundos de empezar.
- Si el permiso de micrófono se deniega: tarjeta amable explicando que se medirá solo el tiempo, con botón "Continuar sin micro".

## Fase 2d: Ejecución, posición de imágenes (memoria visual)

Dos subfases:
- Observación: se muestran 2 a 4 imágenes en fila o cuadrícula, grandes y con separación generosa. Texto guía: "Observa bien las imágenes". Botón "Ya las he visto".
- Pregunta: las imágenes se ocultan y se pregunta "¿Qué imagen estaba en la posición 2?" con las opciones en cuadrícula pulsable.
- Feedback de acierto: check verde suave y micro-animación. Feedback de fallo: NUNCA rojo ni cruz. Usar ámbar suave con mensaje "Casi. Sigamos." y pasar a la siguiente.

## Estados transversales

- Guardando resultados al terminar: overlay suave "Guardando..." (no bloquear más de lo necesario; si falla, continuar y reintentar en segundo plano).
- Reduced motion: todas las transiciones se sustituyen por cortes simples.

## Referencia previa

docs/stitch/activity-player.png (solo cubre texto vertical; diseñar el resto de variantes con el mismo lenguaje).
