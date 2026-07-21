# 12c. Pantalla: Player de actividad (mejoras v1)

- **Modelo recomendado: Opus.** Mejoras acotadas sobre players que ya funcionan; el único punto delicado (cola de reintento) está especificado como versión simple.
- Fase: 12 (pantallas v1). Alcance: app (`shine-app/`).
- Dependencias: 11a (tokens; los players ya migrados visualmente).

## Prompt

Lee `plans/02-plan-ejecucion-v1.md` (sección 4.4 y FASE 12) y el diseño `plans/design/04-activity-player.md`. Trabaja sobre `src/components/activities/` (ActivityBase, VerticalTextPlayer, AudioReadingPlayer, ImageGroupPlayer, ActivityRouter). No reescribas la arquitectura: está validada.

Tareas:

1. **Cuenta atrás en blink**: antes de empezar el auto-avance, overlay 3-2-1 (grande, `text.exercise`, animación suave; respetar reduced motion).
2. **Permiso de micro denegado** (AudioReadingPlayer): fallback amable que permite hacer el ejercicio con cronómetro sin grabación, con mensaje claro y opción de ir a ajustes. Nunca pantalla bloqueada.
3. **Guardado resiliente**: si el POST de logs/summary falla al terminar, la celebración se muestra igualmente y el payload se encola en storage (`@/lib/storage`) para reintento. Cola simple: al arrancar la app y al completar la siguiente actividad, reintentar pendientes en orden; descartar tras N intentos con aviso silencioso en consola. Sin librerías nuevas.
4. **Indicador de progreso discreto** durante la ejecución (ítem X de Y, sutil, sin cronómetro protagonista salvo en voz alta donde ya existe discreto).
5. **Salida accesible**: botón salir siempre visible con confirmación ("¿Quieres salir? Tu progreso de este intento no se guardará").

Recuerda: los summaries llevan área/nivel/aciertos (10b) e iteración por actividad (09c); no dupliques esa lógica, reúsala.

Verificación:
- `npx tsc --noEmit` y `npm run lint` limpios.
- Probar el guardado resiliente parando DDEV (`ddev stop`) antes de terminar una actividad: celebración visible, y al reactivar el backend el summary llega en el siguiente arranque o actividad.
- Blink muestra 3-2-1; denegar micro (web) muestra el fallback.

Al terminar: marca el checkbox en `plans/02-plan-ejecucion-v1.md` y actualiza `docs/progress.md`.
