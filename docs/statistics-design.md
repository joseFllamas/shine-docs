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
activitysummary (field_time, field_iteration, created)
  ├── field_activityid → node.activity (para agrupar por actividad)
  ├── field_sessionid → node.session (para agrupar por sesión)
  └── uid → user (para filtrar por usuario)
```

Opcionalmente:
```
activitylog (field_time individual por elemento)
  └── para gráficas de distribución dentro de un intento
```

---

## Pantallas del sistema de estadísticas

### 1. Dashboard personal
**Ruta Expo**: `app/(auth)/dashboard.tsx`

Muestra un resumen rápido:
- Total de actividades completadas (todas las iteraciones)
- Tiempo total acumulado de práctica
- Actividades completadas esta semana (sparkline)
- Acceso rápido a la última sesión

**Query JSON:API**:
```
GET /jsonapi/activitysummary/summary_vertical_text
  ?filter[uid.id]={user-uuid}
  &sort=-created
  &page[limit]=50
```

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

### 3. Comparativa con el estándar
**Ruta Expo**: `app/(auth)/activity/[id]/comparison.tsx`
**Componente**: `src/components/statistics/StandardComparison.tsx`

**Prerequisito en Drupal**: campo `field_standard_time` (integer, ms) en `node.activity`.

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
src/components/statistics/
├── ActivityEvolutionChart.tsx      Gráfica de líneas — evolución temporal
├── StandardComparison.tsx          Gráfica de barras — vs estándar
├── SessionProgressBar.tsx          Barra de progreso — sesión actual
├── SessionComparisonTable.tsx      Tabla — comparativa entre sesiones
└── StatsDataTable.tsx              Componente base — tabla accesible bajo gráficas
```

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
