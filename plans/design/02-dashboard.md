# Prompt 02: Dashboard (Inicio)

> Usa el design system de Shine ya definido en este proyecto (ver 00-sistema-diseno.md).

---

Diseña el **dashboard** (pantalla de inicio tras el login) de Shine, app de ejercicios para personas con dislexia. Es la pantalla más visitada: debe responder en 3 segundos a "qué hago hoy" y hacer sentir al usuario que avanza.

## Principio clave

La evolución de la dislexia no es lineal. El dashboard solo muestra métricas que SIEMPRE crecen o se mantienen (actividades hechas, minutos de práctica, racha de días). Nunca tiempos medios ni comparativas que puedan haber empeorado.

## Layout (scroll vertical, barra inferior fija con Inicio y Mi progreso)

### 1. Tarjeta de saludo
- "¡Hola, [Nombre]!" en H1, con saludo según hora del día.
- Línea de resumen semanal en cuerpo: "Esta semana: 5 actividades, 24 minutos".
- Racha de días con icono de llama o estrella: "3 días seguidos". Si no hay racha, no mostrar nada negativo, simplemente omitir.
- Fondo con degradado teal muy suave.

### 2. Continuar donde lo dejaste
- Si hay una sesión empezada sin terminar: tarjeta destacada con el título de la sesión, barra de progreso ("7 de 10 actividades") y botón primario "Continuar". Es el CTA principal de la pantalla.
- Si no hay ninguna empezada, esta sección no aparece.

### 3. Tus sesiones
- H2 "Sesiones".
- Lista vertical de tarjetas de sesión. Cada tarjeta:
  - Título de la sesión (H2).
  - Descripción corta (máximo 2 líneas, truncada).
  - Chips de área trabajada (por ejemplo "Conciencia fonológica") y de nivel (Inicial, Intermedio, Avanzado).
  - Barra de progreso con etiqueta "3 de 8 actividades".
  - Badge "Completada" en verde suave si está al 100%, con icono de check.
- Tocar la tarjeta navega al detalle de sesión.

### 4. Estado vacío
- Si no hay sesiones asignadas: ilustración amable más "Todavía no tienes sesiones. Vuelve pronto." Nunca pantalla en blanco.

## Estados a entregar

- Con datos (sesión a medias más lista de sesiones).
- Primer uso (sin actividad previa: saludo sin stats más lista de sesiones sin progreso).
- Vacío (sin sesiones).
- Carga (skeletons de tarjeta, no spinners a pantalla completa).

## Referencia previa

docs/stitch/dashboard.png (evolucionar: añade racha, "continuar" y chips de área/nivel que no existían).
