# 12b. Pantalla: Detalle de sesión

- **Modelo recomendado: Opus.** Pantalla con spec cerrada; la derivación de completadas viene hecha de 09c.
- Fase: 12 (pantallas v1). Alcance: app (`shine-app/`).
- Dependencias: 10b y 11a (y 09c para el estado completada).

## Prompt

Lee `plans/02-plan-ejecucion-v1.md` (sección 4.3 y FASE 12) y el diseño `plans/design/03-sesion-detalle.md`. Trabaja en `shine-app/app/(auth)/session/[id].tsx`.

Muestra (spec 4.3):
- Título, descripción, chips de área/nivel.
- Barra "X de Y actividades completadas" (ProgressBar con label).
- Lista de actividades: icono por tipo de ejercicio (Icon: book-open para lectura, zap para blink, mic para voz alta, image para imágenes), badge Completada/Pendiente y "Hecha N veces" si N > 0.
- CTA fijo inferior: "Continuar" (si hay pendientes y alguna hecha), "Empezar" (ninguna hecha), "Repetir" (todas hechas). Abre la primera actividad pendiente (o la primera si es repetir).
- Banner de celebración al 100% (MotivationCard o equivalente del design system), sin comparaciones.

Hace: carga la sesión con `include=field_activities`; deriva completadas y conteos de los summaries del usuario (lógica de 09c); abre el player.

Estados: carga, error con reintentar (de 09d), sesión sin actividades.

Verificación:
- `npx tsc --noEmit` y `npm run lint` limpios.
- Completar una actividad, volver: la barra y los badges se actualizan; el CTA cambia de Empezar a Continuar y, al acabar todas, a Repetir con la celebración visible.

Al terminar: marca el checkbox en `plans/02-plan-ejecucion-v1.md` y actualiza `docs/progress.md`.
