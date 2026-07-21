# 12e. Pantalla: Mi progreso (estadísticas)

- **Modelo recomendado: Fable.** La pantalla central del producto: gráficas con criterio pedagógico (suavizado, zona objetivo, frases interpretativas), accesibilidad WCAG y muchos estados límite. Es donde más juicio fino hace falta.
- Fase: 12 (pantallas v1). Alcance: app (`shine-app/`).
- Dependencias: 10b (stats lib) y 11a (tokens).

## Prompt

Lee `plans/02-plan-ejecucion-v1.md` (secciones 2.3, 4.6 y FASE 12), el diseño `plans/design/06-progreso-estadisticas.md` y `docs/statistics-design.md`. Reescribe `shine-app/app/(auth)/statistics/index.tsx` sobre `@/lib/stats` (una sola descarga de summaries del usuario alimenta todo) y victory-native (ya instalado).

Las 5 secciones (spec 4.6):

1. **Esfuerzo**: StatTiles con total de actividades, actividades de la semana (con sparkline de los 7 días), tiempo total practicado y racha.
2. **Áreas**: tarjeta por término de `areas` con dedicación relativa y etiqueta positiva. Ordenación: dedicación baja + distancia a la zona objetivo primero, con etiqueta invitadora "Para seguir practicando" (así se responde a "dónde trabajar más" sin señalar debilidades).
3. **Evolución**: selector de ejercicio; línea protagonista de media móvil (ventana 5), puntos crudos de fondo sin protagonismo, zona objetivo (`field_seconds`) como banda sombreada (nunca línea de aprobado; si no hay dato fiable, no se muestra), estrella en la mejor marca; frase interpretativa generada (mejora solo si es mayor del 10%; tendencia plana o negativa: texto neutro orientado a la acción, por ejemplo "Esta área se resiste un poco. Es normal: sigue practicando"); tabla accesible colapsable con los datos crudos (WCAG: la gráfica tiene alternativa textual).
4. **Aciertos**: barras de porcentaje de acierto para ejercicios con datos de precisión, siempre formulado en positivo ("7 de 8 aciertos").
5. **Historial**: sesiones con nombre real, fecha, actividades y tiempo.

Estados: vacío total (invitación cálida a empezar), pocos datos (ocultar tendencias con menos de 5 intentos, mostrar solo esfuerzo), carga y error.

Reglas duras (sección 2.3): nunca flechas rojas, ni porcentajes negativos, ni comparaciones con otros, ni "error/fallo". Comparar solo contra uno mismo. Antes de pintar cualquier gráfica, consulta la skill de dataviz si está disponible.

Verificación:
- `npx tsc --noEmit`, `npm run lint` y `npm test` limpios.
- Con un usuario con más de 5 intentos en un ejercicio: media móvil, mejor marca y zona objetivo visibles y correctas contrastadas a mano.
- Con 2 intentos: la sección de evolución no muestra tendencia.
- Revisión de accesibilidad: tamaños mínimos (14), contraste AA con los tokens, tabla alternativa, reduced motion.

Al terminar: marca el checkbox en `plans/02-plan-ejecucion-v1.md`, actualiza `docs/progress.md` y `docs/statistics-design.md` (que quede alineado con lo construido).
