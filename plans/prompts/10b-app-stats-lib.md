# 10b. App: módulo de cálculo de estadísticas + rellenar campos nuevos

- **Modelo recomendado: Fable.** Diseño de un módulo de cálculo que encapsula el criterio pedagógico del producto (suavizado, mejores marcas, rachas); es la base de la pantalla más importante.
- Fase: 10 (capa de datos de estadísticas). Alcance: app (`shine-app/`).
- Dependencias: 10a (campos nuevos en el backend) y 09c (filtro uid, iteración).

## Prompt

Lee `plans/02-plan-ejecucion-v1.md` (secciones 2.1 a 2.4 y FASE 10 App) y `docs/metrics-tracking.md`. Trabaja en `shine-app/`.

Tareas:

1. **Rellenar los campos nuevos al crear summaries** (los 3 tipos, en `src/lib/api/tracking.ts`): `field_main_area` y `field_activity_level` (la app ya tiene cargado el nodo actividad en ese momento; coste cero), y `field_correct_count`/`field_items_count` donde haya aciertos. Actualizar los tipos TS de `src/types/activitysummary.ts`. Recuerda el gotcha 13 de CLAUDE.md: `label` es requerido en attributes.

2. **Módulo `src/lib/stats/`** puro y testeable (sin imports de React ni de la capa API). Funciones, cada una con su test:
   - `movingAverage(values, window)`: media móvil, ventana 5 por defecto.
   - `personalBest(summaries)`: mejor marca (menor tiempo, o mayor precisión según el tipo).
   - `baselineVsRecent(summaries)`: media de los 3 primeros intentos vs media de los 5 últimos; devuelve el porcentaje de mejora SOLO si es positivo y mayor del 10% (si no, null). Regla de producto: nunca devolver mejora negativa.
   - `streak(dates)`: racha de días consecutivos con actividad, contando hoy o ayer como vigente.
   - `weeklyBuckets(summaries)`: actividades y minutos por día de la semana actual y total semanal.
   - `areaAggregation(summaries)`: dedicación (nº y minutos) por área usando `field_main_area` denormalizado.
   - `accuracy(summary)`: porcentaje de acierto desde `field_correct_count`/`field_items_count` (null si no aplica).
   Todas toleran listas vacías y datos incompletos (summaries antiguos sin área).

3. **Resolver nombres reales**: las queries de summaries deben traer los nombres de actividad y sesión (via `include=field_activityid,field_sessionid` o fetch complementario; evalúa cuál soporta JSON:API para esas referencias y elige el de menos requests). Eliminar los títulos placeholder tipo "Actividad 1".

Verificación:
- `npm test` verde con los tests de todas las funciones (incluye casos límite: 0, 1 y 4 intentos; racha rota; summaries sin área).
- `npx tsc --noEmit` limpio.
- Crear un summary desde la app (usuario) y comprobar en `/jsonapi/` que llega con área, nivel y contadores.
- Una única query de summaries del usuario alimenta todas las funciones (criterio de la FASE 10).

Al terminar: marca los checkboxes en `plans/02-plan-ejecucion-v1.md` y actualiza `docs/progress.md` y `docs/statistics-design.md` si el contrato cambia.
