# Prompt 06: Mi progreso (estadísticas)

> Usa el design system de Shine ya definido en este proyecto (ver 00-sistema-diseno.md).
> Esta es la pantalla más delicada del producto. Leer el principio rector antes de diseñar.

---

Diseña la pantalla **Mi progreso** de Shine, app de ejercicios para personas con dislexia. Convierte los registros de cada ejercicio (tiempo por intento, aciertos, repeticiones) en gráficas y mensajes claros para el usuario y su familia.

## Principio rector: la evolución de la dislexia no es lineal

Un usuario puede tardar más hoy que hace una semana sin estar empeorando (cansancio, dificultad del contenido, día malo). La pantalla NUNCA debe producir la lectura "estoy empeorando". Reglas de diseño derivadas:

1. Las métricas protagonistas son de esfuerzo, que solo pueden crecer: actividades realizadas, minutos de práctica, días de racha, áreas trabajadas.
2. Las métricas de rendimiento (tiempo, aciertos) se muestran siempre suavizadas: media móvil de los últimos 5 intentos, nunca el dato crudo de un solo día como titular.
3. Las comparaciones se hacen contra la mejor marca personal y contra el punto de partida del propio usuario ("cuando empezaste"), nunca contra otros usuarios.
4. La comparativa con el tiempo de referencia del ejercicio se presenta como una zona objetivo sombreada en la gráfica, no como una línea de aprobado/suspenso.
5. Si la tendencia reciente es plana o negativa, el texto acompañante es neutro y orientado a la acción: "Esta área se resiste un poco. Es normal: sigue practicando". Nunca flechas rojas hacia abajo ni porcentajes negativos.
6. Los textos de mejora solo aparecen cuando son reales y significativos: "Lees un 20% más rápido que cuando empezaste".

## Layout (scroll vertical, título "Mi progreso", barra inferior con Inicio y Mi progreso)

### Sección 1: Resumen de esfuerzo (fila de stat tiles con scroll horizontal)
- "Actividades" (total, número grande).
- "Esta semana" (número más sparkline de barras de los últimos 7 días).
- "Tiempo de práctica" (duración formateada, por ejemplo "2 h 15 min").
- "Racha" (días seguidos, icono de llama; si es 0 se muestra "Empieza hoy" en positivo).

### Sección 2: Mis áreas de trabajo (mapa de fortalezas)
- Título: "¿En qué estoy trabajando?"
- Una tarjeta por área (conciencia fonológica, discriminación visual, memoria, ortografía, fluidez lectora), cada una con:
  - Icono y nombre del área.
  - Barra de dedicación (cuánto ha practicado esa área, relativa al total).
  - Etiqueta de estado en positivo: "Va genial" / "Progresando" / "Para seguir practicando" (esta última con tono invitador, en ámbar, nunca en rojo).
- Esta sección responde a "dónde trabajar más" sin puntuar ni suspender.

### Sección 3: ¿Estoy mejorando? (gráfica de evolución)
- Selector de ejercicio (dropdown o chips scrollables).
- Gráfica de líneas: eje X intentos (o semanas), eje Y segundos.
  - Línea principal: media móvil de 5 intentos, gruesa, en teal.
  - Puntos individuales de cada intento en gris claro pequeño detrás (contexto, sin protagonismo).
  - Zona objetivo sombreada en teal muy claro (basada en el tiempo de referencia del ejercicio), etiquetada "zona objetivo".
  - Marcador de estrella ámbar en la mejor marca personal.
- Bajo la gráfica, una frase generada según los datos: "Tu media ha bajado de 14 s a 9 s desde que empezaste" o, si no hay mejora, "Sigues practicando con constancia: 12 intentos este mes".
- Tabla de datos accesible y colapsable bajo la gráfica (WCAG), con los mismos datos en texto.

### Sección 4: Aciertos (solo ejercicios con respuesta correcta/incorrecta)
- Gráfica de barras de porcentaje de aciertos por intento, suavizada igual que la sección 3.
- Presentado como "aciertos", nunca como "errores" ni "fallos".

### Sección 5: Historial de sesiones
- Título: "Mis sesiones".
- Tabla o lista de tarjetas: nombre de sesión, fecha, actividades completadas, tiempo de práctica.
- Ordenada de más reciente a más antigua.

## Accesibilidad de las gráficas (obligatorio)

- Contraste alto, sin grises claros sobre blanco.
- Etiquetas de datos directamente sobre la gráfica, no solo leyenda.
- Ninguna información solo por color: usar formas (estrella, punto, zona rayada) además del color.
- Tabla de datos alternativa colapsable bajo cada gráfica.
- Tipografía mínima 14sp también dentro de las gráficas.

## Estados

- Con datos abundantes.
- Con pocos datos (menos de 5 intentos): ocultar tendencias y mostrar "Completa unas cuantas actividades más para ver tu evolución" con ilustración.
- Vacío total: ilustración más "Completa tu primera actividad para ver tu progreso aquí".
- Carga: skeletons.

## Vista adulto/terapeuta (variante futura, diseñar solo como nota)

Los datos crudos por intento (tiempos exactos, desviaciones, comparativa con referencia) se reservan para una futura vista de tutor protegida, no para la vista del usuario. En v1 basta con dejar el patrón visual preparado.
