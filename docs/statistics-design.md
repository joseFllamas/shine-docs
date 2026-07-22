# Diseño del sistema de estadísticas — Shine

## Objetivo

Mostrar al usuario (y opcionalmente a un terapeuta/supervisor) la evolución de su rendimiento en las actividades a lo largo del tiempo. El sistema debe responder a preguntas concretas del usuario:

- "¿Estoy mejorando?"
- "¿Cuánto tardo comparado con lo normal?"
- "¿Cuántas actividades he hecho esta semana?"

---

## Fuente de datos

Todas las estadísticas se construyen a partir de:

```
activitysummary (field_time, field_iteration, created,
                 field_correct_count, field_items_count)     ← FASE 10
  ├── field_activityid → node actividad (para agrupar y para el nombre real)
  ├── field_sessionid → node.session (para agrupar y para el nombre real)
  ├── field_main_area → taxonomy areas (denormalizado, FASE 10)
  ├── field_activity_level → taxonomy activity_level (denormalizado, FASE 10)
  └── uid → user (para filtrar por usuario)
```

> **Contrato FASE 10** (ver `docs/metrics-tracking.md`): área, nivel y contadores de acierto viven denormalizados en el summary; la app los rellena al crear cada summary. **Una única query** de summaries del usuario, con `include=field_activityid,field_sessionid` para los nombres reales, alimenta toda la pantalla de progreso.

Opcionalmente:
```
activitylog (field_time individual por elemento)
  └── para gráficas de distribución dentro de un intento (vista terapeuta, no v1)
```

---

## Módulo de cálculo `src/lib/stats/` (FASE 10)

Módulo puro y testeable (sin React ni capa API). Las pantallas convierten los recursos JSON:API con `toStatsSummary()` y derivan todo en cliente:

| Función | Qué devuelve |
|---|---|
| `movingAverage(values, window=5)` | Serie suavizada de la misma longitud (línea protagonista de las gráficas) |
| `personalBest(summaries)` | Mejor marca: mayor precisión si el ejercicio la tiene, menor tiempo si no |
| `baselineVsRecent(summaries)` | % de mejora (3 primeros vs 5 últimos) SOLO si es > 10%; nunca negativo; null con < 5 intentos |
| `streak(dates)` | Racha de días consecutivos; vigente si el último día es hoy o ayer |
| `weeklyBuckets(summaries)` | Actividades y minutos por día de la semana actual (L-D) + total |
| `areaAggregation(summaries)` | Dedicación por área (nº, minutos, share), ascendente: la menos trabajada primero |
| `accuracy(summary)` | % de acierto desde los contadores; null si el ejercicio no aplica |

Todas toleran listas vacías y summaries antiguos incompletos (sin área/fecha/contadores).

---

## Pantallas del sistema de estadísticas

### 1. Dashboard personal
**Ruta Expo**: `app/(auth)/dashboard.tsx`

Muestra un resumen rápido:
- Total de actividades completadas (todas las iteraciones)
- Tiempo total acumulado de práctica
- Actividades completadas esta semana (sparkline)
- Acceso rápido a la última sesión

**Query JSON:API** (con nombres reales en la misma request, FASE 10):
```
GET /jsonapi/activitysummary/summary_vertical_text
  ?filter[uid.id]={user-uuid}
  &include=field_activityid,field_sessionid
  &sort=created
  &page[limit]=100
```
Implementada en `getMySummaries()` (`src/lib/api/statistics.ts`), que devuelve `{ summaries, included }`; `titlesFromIncluded()` construye el mapa id → título. Los nodos borrados no vienen en `included` (prever fallback).

---

### 2. Evolución en una actividad
**Ruta Expo**: `app/(auth)/activity/[id]/evolution.tsx`
**Componente**: `src/components/statistics/ActivityEvolutionChart.tsx`

**Qué muestra**: Gráfica de líneas con el tiempo total (`field_time`) de cada intento (`field_iteration`) para una actividad concreta.

```
Tiempo (ms)
│
│    ●
│        ●
│              ●
│                   ●──●
└────────────────────────── Iteración
  1    2    3    4    5
```

**Interpretación**: La línea baja → el usuario mejora (tarda menos).

**Query JSON:API**:
```
GET /jsonapi/activitysummary/summary_vertical_text
  ?filter[uid.id]={user-uuid}
  &filter[field_activityid.id]={activity-uuid}
  &sort=created
  &fields[activitysummary--summary_vertical_text]=field_time,field_iteration,created
```

**Componente Victory Native**:
```tsx
import { VictoryLine, VictoryChart, VictoryAxis } from 'victory-native';

<VictoryChart>
  <VictoryAxis label="Intento" />
  <VictoryAxis dependentAxis label="Tiempo (s)" />
  <VictoryLine
    data={summaries.map(s => ({ x: s.field_iteration, y: s.field_time / 1000 }))}
  />
</VictoryChart>
```

---

### 3. Comparativa con el estándar — ⛔ OBSOLETA (no implementar)

> Retirada en FASE 9/10: el componente `StandardComparison` se eliminó (09d), `field_standard_time` nunca existió (el campo real es `field_seconds`, que se presenta como "zona objetivo" sombreada, nunca como comparación) y las reglas de producto (`plans/02-plan-ejecucion-v1.md` § 2.3) prohíben comparar contra estándares externos: solo contra uno mismo (`personalBest`, `baselineVsRecent`). Se conserva la sección como registro de la decisión.

**Prerequisito en Drupal**: ~~campo `field_standard_time` (integer, ms) en `node.activity`~~.

**Qué muestra**: Gráfica de barras comparando el mejor tiempo del usuario vs el tiempo estándar.

```
Tiempo (s)
│  ████  Standard
│  ████
│  ████  ▓▓▓▓  Mi mejor tiempo
│         ▓▓▓▓
└──────────────
```

**Si `field_standard_time` no está definido**: ocultar esta pantalla o mostrar mensaje "Sin referencia estándar configurada para esta actividad".

**Query JSON:API**:
```
GET /jsonapi/node/activity/{uuid}?fields[node--activity]=field_standard_time

GET /jsonapi/activitysummary/summary_vertical_text
  ?filter[uid.id]={user-uuid}
  &filter[field_activityid.id]={activity-uuid}
  &sort=field_time    (para obtener el mínimo)
  &page[limit]=1
```

---

### 4. Progreso de una sesión
**Ruta Expo**: `app/(auth)/session/[id]/progress.tsx`
**Componente**: `src/components/statistics/SessionProgressBar.tsx`

**Qué muestra**: Cuántas actividades de la sesión ha completado el usuario (al menos una vez).

```
Sesión "Palabras básicas"
████████████░░░░  7 / 10 actividades completadas (70%)
```

**Query JSON:API**:
```
GET /jsonapi/node/session/{uuid}?include=field_activities

GET /jsonapi/activitysummary/summary_vertical_text
  ?filter[uid.id]={user-uuid}
  &filter[field_sessionid.id]={session-uuid}
  &fields[activitysummary--summary_vertical_text]=field_activityid
```

Contar actividades únicas en los summaries = actividades completadas.

---

### 5. Comparativa entre sesiones
**Ruta Expo**: `app/(auth)/statistics/sessions.tsx`
**Componente**: `src/components/statistics/SessionComparisonTable.tsx`

**Qué muestra**: Tabla con el tiempo total y número de actividades completadas por sesión, ordenadas cronológicamente.

| Sesión | Fecha | Actividades | Tiempo total |
|---|---|---|---|
| Palabras básicas | 15 mar | 10/10 | 8 min |
| Frases cortas | 18 mar | 7/10 | 6 min |

**Query JSON:API**:
```
GET /jsonapi/activitysummary/summary_vertical_text
  ?filter[uid.id]={user-uuid}
  &sort=-created
  &include=field_sessionid
  &page[limit]=20
```

Agrupar en el cliente por `field_sessionid`.

---

## Accesibilidad en las gráficas

Dado que los usuarios tienen dislexia, las gráficas deben:

- Usar colores con alto contraste (WCAG AA mínimo)
- Incluir etiquetas de datos visibles directamente en la gráfica (no solo en leyenda)
- Fuente sans-serif grande (mínimo 16sp)
- Ofrecer alternativa textual: bajo cada gráfica, mostrar la tabla de datos equivalente (colapsable)
- No depender únicamente del color para comunicar información (añadir formas o patrones)

---

## Estructura de componentes

```
src/lib/stats/                      Cálculo puro (FASE 10, ver sección arriba)
├── types.ts / adapter.ts           StatsSummary + toStatsSummary
├── movingAverage.ts · personalBest.ts · baselineVsRecent.ts
├── streak.ts · weeklyBuckets.ts · areaAggregation.ts · accuracy.ts
└── __tests__/                      Un test por función

src/components/statistics/
├── ActivityEvolutionChart.tsx        Gráfica de líneas — evolución temporal
├── ImagePositionEvolutionChart.tsx   Evolución de precisión (imágenes)
├── SessionComparisonTable.tsx        Tabla — historial por sesión
└── StatsDataTable.tsx                Componente base — tabla accesible bajo gráficas
```

> `StandardComparison` y `SessionProgressBar` se eliminaron en la limpieza A7 (09d); el progreso de sesión usa el `ProgressBar` del design system.

---

## Datos calculados en cliente vs servidor

| Métrica | Dónde se calcula |
|---|---|
| Tiempo total por intento | Servidor (guardado en `field_time` del summary) |
| Mejor tiempo del usuario | Cliente (min de los summaries) |
| Comparativa vs estándar | Cliente (datos del nodo + datos del summary) |
| Actividades completadas en sesión | Cliente (distinct count de activityid en summaries) |
| Puntuación normalizada (futuro) | Cliente o servidor según complejidad |

Preferir cálculos en cliente para reducir carga en Drupal y aprovechar los datos ya descargados via JSON:API.

---

## Notas de implementación

- Victory Native requiere `react-native-svg`: `npx expo install react-native-svg victory-native`
- Para el dashboard, hacer una única query con `page[limit]=50` y derivar todas las métricas del cliente (evitar múltiples requests)
- Usar `useMemo` para los cálculos derivados (mínimos, agrupaciones) en los componentes de estadísticas
- Cachear las queries de estadísticas con una TTL de 5 minutos (los datos históricos no cambian)
