# 12a. Pantalla: Dashboard (Inicio)

- **Modelo recomendado: Opus.** Implementación de pantalla con especificación funcional y diseño visual ya validados; la lógica de datos viene hecha de 10b.
- Fase: 12 (pantallas v1). Alcance: app (`shine-app/`).
- Dependencias: 10b (stats lib y queries) y 11a (Screen y tokens).

## Prompt

Lee `plans/02-plan-ejecucion-v1.md` (sección 4.2 y FASE 12) y el diseño `plans/design/02-dashboard.md`. Si hay mockup aprobado en el proyecto "Shine Design System" de claude.ai/design (`ui_kits/shine-app/HomeScreen.jsx`, leer con DesignSync get_file), es la referencia visual; portar, no reinterpretar. Trabaja en `shine-app/app/(auth)/dashboard.tsx` con los componentes de `@/components/ui` y las funciones de `@/lib/stats`.

Muestra (spec 4.2):
- Saludo con nombre del usuario y momento del día (buenos días/tardes/noches).
- Resumen semanal: actividades y minutos de la semana (`weeklyBuckets`).
- Racha si existe (`streak`), formulada en positivo; si no hay racha, no mostrar nada negativo.
- Tarjeta "Continuar": la sesión a medias más reciente (progreso entre 0 y 100%), con CTA directo.
- Lista de sesiones: descripción, chips de área/nivel (AreaChip/LevelChip, derivadas de sus actividades), barra de progreso real (ProgressBar, desde summaries) y badge de completada.
- Estado vacío amable (EmptyState) y estado de carga.

Hace: 1 query de sesiones + 1 query de summaries del usuario; todo lo demás se deriva en cliente. Navega a detalle de sesión y a Mi progreso (BottomNav con las pestañas Inicio/Progreso).

Reglas de producto (sección 2.3 del plan): titulares de esfuerzo, nunca juicio de rendimiento; sin comparaciones; sin la palabra "error".

Verificación:
- `npx tsc --noEmit` y `npm run lint` limpios.
- Con un usuario con datos: resumen semanal y racha correctos contrastados a mano con los summaries.
- Con un usuario nuevo: estado vacío amable, sin ceros tristes ni NaN.

Al terminar: marca el checkbox en `plans/02-plan-ejecucion-v1.md` y actualiza `docs/progress.md`.
