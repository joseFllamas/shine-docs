# Plan de ejecución: primera versión publicable de Shine

Fecha: 2026-07-21. Parte de la auditoría `01-auditoria.md`. Continúa la numeración del roadmap existente (las fases 0 a 7b están completadas; este plan sustituye y detalla la antigua Fase 8).

---

## 1. Definición de la v1

Una app instalable (TestFlight / Google Play internal / web) con la que un usuario real pueda:

1. Entrar con su cuenta.
2. Ver sus sesiones asignadas y su progreso en ellas.
3. Completar los 3 tipos de ejercicio existentes más 2 o 3 ejercicios nuevos del docx.
4. Ver su progreso en una pantalla de estadísticas motivadora, con agregación por área.
5. Todo con el design system de dislexia aplicado (Nunito, teal, WCAG AA).

Queda fuera de la v1: vista de terapeuta/tutor, modo offline, multilingüe, reconocimiento de voz, subida del audio grabado, los 7 ejercicios restantes del docx.

---

## 2. Estrategia de estadísticas: de activitylog a gráficas claras

Es la preocupación central del producto, así que se fija aquí el criterio antes de las fases.

### 2.1 Qué se puede derivar de los datos actuales

| Métrica | Fuente | Naturaleza |
|---|---|---|
| Tiempo por intento | `activitysummary.field_time` | Rendimiento (volátil, no lineal) |
| Tiempo por ítem y distribución dentro de un intento | `activitylog.field_time` + `field_weight` | Rendimiento fino (uso terapeuta, no v1) |
| Aciertos | `activitylog.field_correct` (imágenes; extensible a nuevos ejercicios) | Rendimiento |
| Nº de intentos por actividad | `field_iteration` / conteo de summaries | Esfuerzo (siempre crece) |
| Actividades y minutos practicados por día/semana | summaries `created` + `field_time` | Esfuerzo |
| Racha de días | summaries `created` | Esfuerzo |
| Dedicación y evolución por área y por nivel | summary + área/nivel (requiere denormalización, ver 2.2) | Mixta |
| Zona objetivo | `field_seconds` del nodo actividad | Referencia |

### 2.2 Cambio de modelo: denormalizar área y nivel en el summary

Problema: logs y summaries solo referencian el nodo actividad; agregar por área desde la app exige resolver cada actividad. Solución elegida (simple y compatible con el patrón actual):

- Añadir a los 3 bundles de `activitysummary` (storage compartido): `field_main_area` (ref taxonomy `areas`) y `field_activity_level` (ref taxonomy `activity_level`).
- La app los rellena al crear el summary (ya tiene cargado el nodo actividad en ese momento; coste cero).
- Añadir también `field_correct_count` y `field_items_count` (integers) al summary para que la precisión no exija descargar los logs.
- Los datos históricos se rellenan una vez con un script drush (hook_update o php-eval) que recorre summaries y copia el área/nivel desde su actividad.

Con esto, TODA la pantalla de estadísticas se resuelve con una única query de summaries del usuario (más los nombres de actividad via include).

### 2.3 Criterio de presentación: la evolución no es lineal

Reglas de producto que aplican a dashboard y estadísticas (detalladas visualmente en `plans/design/06-progreso-estadisticas.md`):

1. Titulares de esfuerzo, no de rendimiento: actividades hechas, minutos, racha, áreas trabajadas. Son métricas que nunca retroceden.
2. Rendimiento siempre suavizado: la línea protagonista es la media móvil de 5 intentos; los puntos crudos quedan de fondo sin protagonismo.
3. Comparar solo contra uno mismo: mejor marca personal y "cuando empezaste" (media de los 3 primeros intentos vs media de los 5 últimos). El porcentaje de mejora solo se muestra si es positivo y significativo (mayor del 10%).
4. `field_seconds` se presenta como "zona objetivo" sombreada, nunca como línea de aprobado.
5. Tendencia plana o negativa: texto neutro orientado a la acción ("Esta área se resiste un poco. Es normal: sigue practicando") y ahí es donde el producto responde a "dónde trabajar más": la sección de áreas ordena por dedicación baja + distancia a la zona objetivo, con etiqueta invitadora "Para seguir practicando".
6. Nunca: flechas rojas, porcentajes negativos, comparaciones con otros usuarios, la palabra "error/fallo" (usar "aciertos").
7. Los datos crudos por intento quedan disponibles en la tabla accesible colapsable (que además cumple WCAG) y, en el futuro, en la vista de terapeuta.

### 2.4 Funciones de cálculo (cliente, en `src/lib/stats/`)

Módulo puro y testeable: `movingAverage(values, window)`, `personalBest(summaries)`, `baselineVsRecent(summaries)`, `streak(dates)`, `weeklyBuckets(summaries)`, `areaAggregation(summaries)`, `accuracy(summary)`. Una única descarga de summaries alimenta todo.

---

## 3. Fases de trabajo

### FASE 9: Saneamiento técnico (prerequisito de todo)

Backend:
- [ ] Convertir el consumer OAuth en cliente público (confidential: false) y eliminar el uso de client_secret en la app (PKCE ya protege el flujo).
- [ ] Decidir el destino del endpoint legacy `/activitylog-register/add`: retirarlo si el theme acoplado ya no se usa; si se mantiene, corregir el bundle hardcodeado (bug B1).
- [ ] Restringir CORS a los orígenes reales cuando exista dominio de producción.

App:
- [ ] Configuración por entorno: `API_BASE`, `clientId` via `app.config.ts` + `expo-constants` (variables EXPO_PUBLIC_*). Un solo punto de verdad.
- [ ] Aplicar el filtro `filter[uid.id]` en `getMySummaries` y `getMyImageSummaries` (bug A1).
- [ ] Calcular `iteration` por actividad (consulta del último summary de esa actividad) en lugar del contador global del store (bug A4).
- [ ] Derivar el estado "completada" de la sesión desde los summaries del servidor (bug A5) y persistirlo.
- [ ] Manejo de errores: catch en la carga de sesión, feedback de error en login, fallback del ActivityRouter (mensaje "tipo de ejercicio no soportado" en lugar de pantalla en blanco).
- [ ] Limpiar código huérfano (A7) y añadir ESLint + prettier + un primer test del módulo de stats.

Verificación: flujo completo en dos dispositivos con el mismo usuario sin duplicar iteraciones; sin secreto OAuth en el bundle.

### FASE 10: Capa de datos de estadísticas

Backend:
- [ ] Añadir `field_main_area`, `field_activity_level`, `field_correct_count`, `field_items_count` a los 3 bundles de activitysummary. Exportar config.
- [ ] Script de backfill para summaries históricos (área/nivel desde el nodo actividad).
- [ ] Añadir `field_seconds` a `activity_image_position` (o decidir explícitamente que las imágenes no tienen zona objetivo).
- [ ] Índices en `activitysummary_field_data` para (uid, created) y (uid, field_activityid).

App:
- [ ] Rellenar los campos nuevos al crear summaries (los 3 tipos).
- [ ] Módulo `src/lib/stats/` con las funciones de 2.4 y tests.
- [ ] Resolver nombres reales de actividad y sesión en las queries (include o fetch de nodos).

Verificación: una sola query de summaries devuelve todo lo necesario para pintar la pantalla de progreso completa, con nombres reales.

### FASE 11: Design system en código

- [x] Crear `src/theme/` (tokens: colores, tipografía, espaciado, radios, sombras). Hecho el 2026-07-21 portando el proyecto "Shine Design System" de claude.ai/design (fuente de verdad visual).
- [x] Componentes ui/: Button, Card, Icon (SVG propio estilo Lucide), LevelChip, AreaChip, ProgressBar, StatTile, MotivationCard, EmptyState, BottomNav. Portados de los JSX del proyecto Design a React Native.
- [x] Cargar la fuente Nunito: instalada y conectada el 2026-07-21 (useFonts en el layout raíz, familias por peso en el tema).
- [ ] Componente Screen (layout con max-width 480px en web, usando `layout.appMaxWidth`).
- [ ] Migrar las pantallas existentes a los tokens (eliminar los hex duplicados).

En paralelo (sin código): generar las pantallas con Claude Design usando `plans/design/01` a `06` y validar la dirección visual antes de pulir detalles.

Verificación: cero colores hardcodeados fuera de theme; app visualmente coherente con los mockups aprobados.

### FASE 12: Pantallas v1 (especificación funcional en la sección 4)

- [ ] Dashboard: resumen semanal, racha, "continuar donde lo dejaste", progreso por sesión, chips de área/nivel.
- [ ] Detalle de sesión: barra de progreso real, estados por actividad desde servidor, CTA continuar/empezar/repetir, celebración de sesión completa.
- [ ] Player: cuenta atrás en blink, permiso de micro denegado con fallback amable, guardado resiliente (cola de reintento simple).
- [ ] Actividad completada: mejor marca personal, variante sesión completa, sin comparaciones negativas.
- [ ] Mi progreso: las 5 secciones del prompt 06 (esfuerzo, áreas, evolución suavizada, aciertos, historial).
- [ ] Mensajes de motivación filtrados por tipo de actividad (arreglo A6).

Verificación: recorrido completo con usuario real y datos reales; revisión de accesibilidad (tamaños, contraste, tablas alternativas, reduced motion).

### FASE 13: Nuevos ejercicios (primer lote)

Criterio de selección: máximo valor terapéutico con mínima dependencia de assets de audio (locuciones), porque las locuciones requieren producción de contenido aparte.

| Prioridad | Ejercicio (docx) | Por qué | Modelo de datos |
|---|---|---|---|
| 1 | 7.6 Matriz de discriminación visual | Sin audio, mecánica simple, área discriminación visual hoy sin cubrir | `activity_letter_matrix` + `logletermatrix`/`summary_letter_matrix` con field_correct_count, field_time |
| 2 | 7.8 Reordenamiento de sílabas | Sin audio obligatorio, trabaja decodificación y memoria | `activity_syllable_order` + bundles paralelos |
| 3 | 7.5 Segmentación léxica | Sin audio, contenido facilísimo de crear (frases sin espacios) | `activity_word_split` + bundles paralelos |

Cada uno sigue el checklist de `docs/activity-system.md` y las convenciones de nombres de `docs/conventions.md`. Los 7 restantes (los que exigen audio: omisión silábica, pseudopalabras, dictado inverso, intruso rímico, instrucciones encadenadas, más segmentación silábica y ortografía con colores) pasan a v1.1 junto con la producción de locuciones (valorar TTS de calidad como atajo).

- [ ] Por ejercicio: node type + bundles log/summary + config export + tipos TS + Player + rama en ActivityRouter + tracking.
- [ ] Diseño previo con `plans/design/07-ejercicios-nuevos.md`.
- [ ] Añadir términos de taxonomía `areas` que falten (discriminación visual, conciencia léxica...).

Verificación: los 3 ejercicios jugables dentro de una sesión mixta, con sus métricas en la pantalla de progreso (incluida precisión).

### FASE 14: Publicación (antigua Fase 8)

- [ ] Backend de producción: dominio + SSL, claves RSA nuevas, client_secret fuera, CORS restringido, `settings.php` de prod.
- [ ] app.json: bundle ID, iconos, splash con la marca definitiva.
- [ ] EAS Build: perfiles development/preview/production; builds de preview en dispositivos reales.
- [ ] TestFlight + Google Play internal testing con 3 a 5 familias piloto.
- [ ] Web: `expo export` y despliegue estático.
- [ ] Recogida de feedback del piloto antes del lanzamiento abierto.

---

## 4. Especificación funcional pantalla a pantalla (v1)

Resumen operativo; el detalle visual está en `plans/design/`.

### 4.1 Login
- Muestra: marca, tagline, botón "Entrar", nota de seguridad. Estados: normal, cargando, error con reintento.
- Hace: OAuth2 Authorization Code + PKCE (sin secret), guarda tokens en secure storage, redirige a dashboard.

### 4.2 Dashboard (Inicio)
- Muestra: saludo con nombre y momento del día; resumen semanal (actividades y minutos); racha si existe; tarjeta "Continuar" con la sesión a medias más reciente; lista de sesiones con descripción, chips de área/nivel, barra de progreso y badge de completada; estado vacío amable.
- Hace: 1 query de sesiones + 1 query de summaries del usuario (de la que deriva progreso, semana y racha).
- Navega a: detalle de sesión, Mi progreso (barra inferior).

### 4.3 Detalle de sesión
- Muestra: título, descripción, chips, barra "X de Y actividades completadas"; lista de actividades con icono por tipo, badge Completada/Pendiente y "Hecha N veces"; CTA fijo Continuar/Empezar/Repetir; banner de celebración al 100%.
- Hace: carga sesión con include de actividades; deriva completadas de los summaries; abre el player.

### 4.4 Player de actividad
- Muestra: intro (título, chips, explicación del backend, botón Comenzar); ejecución según tipo (texto vertical con tap, blink con cuenta atrás 3-2-1 y auto-avance, lectura en voz alta con micro y cronómetro discreto, imágenes con observación y pregunta); indicador de progreso discreto; salida accesible.
- Hace: cronometra por ítem; al terminar crea logs + summary del bundle correcto con área/nivel/aciertos; calcula iteración por actividad; si el guardado falla, celebra igualmente y reintenta después.

### 4.5 Actividad completada
- Muestra: check animado, confeti breve, "Has practicado X segundos" (esfuerzo, no marca), chip "¡Tu mejor marca!" solo si procede, mensaje motivacional del backend filtrado por tipo, botones Siguiente/Volver; variante de sesión completa con resumen del día.
- Nunca muestra: comparaciones negativas ni tiempos "peores".

### 4.6 Mi progreso
- Sección 1 Esfuerzo: stat tiles (total, semana con sparkline, tiempo, racha).
- Sección 2 Áreas: tarjeta por término de taxonomy `areas` con dedicación relativa y etiqueta positiva; responde a "dónde trabajar más".
- Sección 3 Evolución: selector de ejercicio; línea de media móvil + puntos crudos de fondo + zona objetivo (field_seconds) + estrella en mejor marca; frase interpretativa generada; tabla accesible colapsable.
- Sección 4 Aciertos: barras de % de acierto (ejercicios con field_correct), formulado en positivo.
- Sección 5 Historial: sesiones con nombre real, fecha, actividades y tiempo.
- Estados: vacío, pocos datos (ocultar tendencias con menos de 5 intentos), carga.

---

## 5. Orden y dependencias

```
FASE 9 (saneamiento) ──► FASE 10 (datos stats) ──► FASE 12 (pantallas)
        │                                              ▲
        └────────► FASE 11 (design system) ────────────┘
                        (diseño visual con plans/design en paralelo)
FASE 12 ──► FASE 13 (ejercicios nuevos) ──► FASE 14 (publicación)
```

Hito de control recomendado tras la FASE 12: demo con datos reales a 2 o 3 familias/terapeutas antes de invertir en los ejercicios nuevos, para validar sobre todo la pantalla de progreso.

## 6. Riesgos principales

| Riesgo | Mitigación |
|---|---|
| Las gráficas siguen leyéndose como juicio pese al suavizado | Testar la pantalla de progreso con usuarios reales en el hito post FASE 12; tener lista la alternativa de ocultar tiempos y dejar solo esfuerzo |
| Producción de locuciones de audio bloquea los ejercicios fonológicos | Lote 1 sin audio; evaluar TTS neuronal para el lote 2 |
| `field_seconds` mal calibrado (zona objetivo irreal) | Revisar valores con el terapeuta; si no hay dato fiable, la zona objetivo no se muestra |
| Cambios de bundles en Drupal rompen la app | Congelar el contrato JSON:API por fase; los tipos TS son el contrato espejo |
