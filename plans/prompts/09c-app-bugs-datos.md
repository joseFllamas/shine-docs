# 09c. App: bugs de corrección de datos (filtro uid, iteración, completado persistente)

- **Modelo recomendado: Fable.** Lógica sutil de corrección: derivación de estado desde el servidor, iteraciones con varios dispositivos, decisiones sobre qué persiste y dónde.
- Fase: 9 (saneamiento técnico). Alcance: app (`shine-app/`).
- Dependencias: ninguna. Bugs: A1, A4, A5, B6.

## Prompt

Lee `plans/02-plan-ejecucion-v1.md` (FASE 9, bloque App) y `plans/01-auditoria.md` (A1, A4, A5, B6). Trabaja en `shine-app/`. Lee antes `docs/metrics-tracking.md` y `docs/api-design.md`.

Tareas:

1. **A1, filtro por usuario**: `getMySummaries` y `getMyImageSummaries` (en `src/lib/api/statistics.ts` o donde vivan) reciben el uuid del usuario y no lo aplican. Añadir `filter[uid.id]={uuid}` a las queries. Revisa el resto de funciones de la capa API por si el patrón se repite.

2. **A4/B6, iteración por actividad**: hoy `iteration` es un contador global del store que no se resetea. Cambiar el cálculo: antes de crear el summary, consultar el último summary del usuario PARA ESA ACTIVIDAD (`filter[uid.id]` + `filter[field_activityid.id]` + `sort=-created` + `page[limit]=1`) y usar `field_iteration + 1` (o 1 si no hay). Ten en cuenta la ventana de carrera con dos dispositivos: no es necesario resolverla del todo (severidad baja según auditoría), pero documenta la limitación en el código donde corresponda.

3. **A5, completado derivado del servidor**: el estado "completada" de una actividad dentro de la sesión es local y se pierde al navegar. Derivarlo de los summaries del usuario (existe summary para esa actividad con esa sesión => completada) al cargar el detalle de sesión, y mantener el estado local solo como cache optimista tras completar. Revisa `app/(auth)/session/[id].tsx` y el store `src/store/session.ts` y decide qué parte del store sobra (pero la limpieza profunda es del prompt 09d).

Verificación:
- `npx tsc --noEmit` limpio.
- Recorrido manual (lo prueba el usuario): completar una actividad, salir al dashboard, volver a entrar en la sesión y ver la actividad como completada.
- Con dos sesiones de navegador con el mismo usuario: las iteraciones de una misma actividad no se duplican ni se comparten entre actividades distintas.

Al terminar: marca los checkboxes en `plans/02-plan-ejecucion-v1.md` y actualiza `docs/progress.md`.
