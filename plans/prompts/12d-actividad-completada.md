# 12d. Pantalla: Actividad completada + mensajes de motivación

- **Modelo recomendado: Opus.** Pantalla pequeña con reglas de producto explícitas; el dato "mejor marca" viene de la stats lib.
- Fase: 12 (pantallas v1). Alcance: app (`shine-app/`).
- Dependencias: 12c. Bugs: A6 (filtro de mensajes).

## Prompt

Lee `plans/02-plan-ejecucion-v1.md` (sección 4.5 y FASE 12) y el diseño `plans/design/05-actividad-completada.md`. Trabaja en el estado "completed" de `ActivityBase` (o extráelo a componente propio si gana claridad).

Muestra (spec 4.5):
- Check animado y confeti breve (respetar reduced motion: sin animación si está activo).
- "Has practicado X segundos/minutos": titular de esfuerzo, NUNCA el tiempo como marca a batir.
- Chip "¡Tu mejor marca!" SOLO si `personalBest` de `@/lib/stats` lo confirma para este intento. Si no lo es, no se menciona nada del rendimiento.
- Mensaje motivacional del backend: corregir A6 para que `getMotivationMessages` filtre por tipo de actividad (`field_activity_types`) y elegir uno aleatorio del tipo correcto.
- Botones: "Siguiente actividad" (si quedan pendientes en la sesión) y "Volver a la sesión".
- Variante sesión completa: celebración mayor con resumen del día (actividades y minutos de hoy).

Prohibido (regla 2.3 del plan): comparaciones negativas, tiempos "peores", flechas rojas, la palabra "error" o "fallo".

Verificación:
- `npx tsc --noEmit` y `npm run lint` limpios.
- Completar una actividad con peor tiempo que el histórico: la pantalla no lo delata.
- Completar con mejor tiempo: aparece el chip de mejor marca.
- Los mensajes de motivación mostrados corresponden al tipo de actividad jugado.

Al terminar: marca los checkboxes (incluido el de mensajes A6) en `plans/02-plan-ejecucion-v1.md` y actualiza `docs/progress.md`.
