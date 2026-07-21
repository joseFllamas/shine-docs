# Prompt 03: Detalle de sesión

> Usa el design system de Shine ya definido en este proyecto (ver 00-sistema-diseno.md).

---

Diseña la pantalla de **detalle de sesión** de Shine, app de ejercicios para personas con dislexia. Una sesión es un grupo ordenado de actividades (ejercicios) que el usuario realiza de una en una.

## Layout (scroll vertical, flecha atrás arriba a la izquierda, sin barra inferior)

### Cabecera
- Título de la sesión (H1).
- Descripción (cuerpo, 18sp).
- Chips de área trabajada y nivel.
- Barra de progreso de la sesión con etiqueta: "8 de 10 actividades completadas".

### Lista de actividades
Cada actividad es una fila (tarjeta ligera) con:
- Icono según tipo de ejercicio: texto (Aa), parpadeo (rayo o destello), audio/lectura en voz alta (micrófono), imágenes (foto).
- Título de la actividad.
- Badge de estado: "Completada" (verde suave con check) o "Pendiente" (gris neutro). Nunca "Fallida" ni estados negativos.
- Si está completada: etiqueta pequeña con las veces realizada ("Hecha 3 veces"). No mostrar tiempos aquí.
- La fila entera es pulsable y lleva al player de esa actividad.

### CTA inferior fijo (sticky)
- Botón primario ancho: "Continuar" (abre la primera actividad no completada) o "Empezar" si ninguna está hecha.
- Si todas están completadas: banner de celebración "¡Has completado esta sesión!" y el botón pasa a "Repetir sesión" (secundario). Repetir es positivo: la práctica repetida es parte del método.

## Estados a entregar

- Sesión sin empezar.
- Sesión a medias.
- Sesión completada al 100% (celebración).
- Carga (skeleton de filas).
- Error de carga: mensaje amable con botón "Reintentar".

## Referencia previa

docs/stitch/session-detail.png (evolucionar: añadir chips de área/nivel y el estado de sesión completa).
