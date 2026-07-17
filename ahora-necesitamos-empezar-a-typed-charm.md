# Plan: Statistics Dashboard — Charts per activity type

## Context

The statistics page at `app/(auth)/statistics/index.tsx` only shows `summary_vertical_text` and a generic session table. We need to add dynamic charts showing evolution per exercise type:
- **Text activities** (summary_vertical_text): time per iteration — one chart per activity UUID
- **Image position** (summary_image_position): time AND accuracy % per iteration

**Real data in Drupal (uid=1):**
- 63 `summary_vertical_text` records: 50+ iterations activity node 2, ~8 for node 5
- 2 `summary_image_position` records: iteration=1 for activity node 6, 100% accuracy (2/2 correct)
- 4 `logimageposition` records with `field_correct` (boolean), `field_time`
- `getMySummaries` currently caps at 50 — misses 13 records; must raise to 100

**victory-native status:** NOT in `package.json` (only `react-native-svg: 15.12.1` is). The user must install it before implementation runs.

---

## Prerequisite (user runs this in terminal)

```bash
cd shine-app && nvm use 21 && npx expo install victory-native
```

---

## Files to create / modify

| Action | File |
|---|---|
| Modify | `shine-app/src/lib/api/statistics.ts` |
| Create | `shine-app/src/components/statistics/ImagePositionEvolutionChart.tsx` |
| Modify | `shine-app/app/(auth)/statistics/index.tsx` |

---

## Step 1 — `src/lib/api/statistics.ts`

**Changes:**
1. Change default limit from `50` to `100` in `getMySummaries`.
2. Add `getMyImageSummaries` (fetches summaries + includes logs for accuracy):

```typescript
import { ImageSummaryResource } from '@/types/activitysummary';
import { JsonApiResource } from '@/types/jsonapi';

export async function getMyImageSummaries(
  userUuid: string
): Promise<{ summaries: ImageSummaryResource[]; included: JsonApiResource[] }> {
  const res = await apiFetch<JsonApiCollection<ImageSummaryResource>>(
    `/activitysummary/summary_image_position?filter[uid.id]=${userUuid}&include=field_activitylogs&sort=created`
  );
  return { summaries: res.data, included: res.included ?? [] };
}
```

3. Add `computeAccuracy` helper:

```typescript
export function computeAccuracy(
  summary: ImageSummaryResource,
  included: JsonApiResource[]
): number {
  const logIds = (
    (summary.relationships?.field_activitylogs?.data as Array<{ id: string }>) ?? []
  ).map((d) => d.id);
  const logs = included.filter((r) => logIds.includes(r.id));
  if (!logs.length) return 0;
  const correct = logs.filter((l) => l.attributes.field_correct === true).length;
  return Math.round((correct / logs.length) * 100);
}
```

---

## Step 2 — New `ImagePositionEvolutionChart.tsx`

Mirrors `ActivityEvolutionChart` but shows **two** `VictoryBar` charts:
- Top: tiempo (ms → seconds) per iteration
- Bottom: accuracy % per iteration

```typescript
// src/components/statistics/ImagePositionEvolutionChart.tsx
import { useMemo } from 'react';
import { View, Text, StyleSheet } from 'react-native';
import { VictoryBar, VictoryChart, VictoryAxis } from 'victory-native';
import { ImageSummaryResource } from '@/types/activitysummary';
import { JsonApiResource } from '@/types/jsonapi';
import { msToSeconds, computeAccuracy } from '@/lib/api/statistics';
import StatsDataTable from './StatsDataTable';

interface Props {
  summaries: ImageSummaryResource[];
  included: JsonApiResource[];
}

export default function ImagePositionEvolutionChart({ summaries, included }: Props) {
  const timeData = useMemo(
    () => summaries.map((s, i) => ({
      x: s.attributes.field_iteration ?? i + 1,
      y: msToSeconds(s.attributes.field_time ?? 0),
    })),
    [summaries]
  );

  const accuracyData = useMemo(
    () => summaries.map((s, i) => ({
      x: s.attributes.field_iteration ?? i + 1,
      y: computeAccuracy(s, included),
    })),
    [summaries, included]
  );

  const tableRows = summaries.map((s, i) => ({
    label: `Intento ${s.attributes.field_iteration ?? i + 1}`,
    value: `${msToSeconds(s.attributes.field_time ?? 0)}s · ${accuracyData[i].y}% aciertos`,
  }));

  if (!summaries.length) {
    return <Text style={styles.empty}>Aún no hay datos para esta actividad.</Text>;
  }

  return (
    <View>
      <Text style={styles.chartLabel}>Tiempo (segundos)</Text>
      <VictoryChart height={180} padding={{ left: 55, right: 20, top: 10, bottom: 40 }}>
        <VictoryAxis
          label="Intento"
          tickFormat={(t) => `#${t}`}
          style={{ axisLabel: { fontSize: 12, padding: 26 }, tickLabels: { fontSize: 11 } }}
        />
        <VictoryAxis
          dependentAxis
          label="Segundos"
          style={{ axisLabel: { fontSize: 12, padding: 38 }, tickLabels: { fontSize: 11 } }}
        />
        <VictoryBar
          data={timeData}
          style={{ data: { fill: '#1a5276' } }}
          barWidth={28}
        />
      </VictoryChart>

      <Text style={styles.chartLabel}>Precisión (%)</Text>
      <VictoryChart height={180} padding={{ left: 55, right: 20, top: 10, bottom: 40 }}>
        <VictoryAxis
          label="Intento"
          tickFormat={(t) => `#${t}`}
          style={{ axisLabel: { fontSize: 12, padding: 26 }, tickLabels: { fontSize: 11 } }}
        />
        <VictoryAxis
          dependentAxis
          label="%"
          domain={[0, 100]}
          style={{ axisLabel: { fontSize: 12, padding: 34 }, tickLabels: { fontSize: 11 } }}
        />
        <VictoryBar
          data={accuracyData}
          style={{ data: { fill: '#27ae60' } }}
          barWidth={28}
        />
      </VictoryChart>

      <StatsDataTable rows={tableRows} />
    </View>
  );
}

const styles = StyleSheet.create({
  chartLabel: { fontSize: 14, fontWeight: '600', color: '#2c3e50', marginTop: 8, marginLeft: 4 },
  empty: { color: '#7f8c8d', textAlign: 'center', marginVertical: 24 },
});
```

---

## Step 3 — `app/(auth)/statistics/index.tsx`

**New state:**
```typescript
const [imageSummaries, setImageSummaries] = useState<ImageSummaryResource[]>([]);
const [imageIncluded, setImageIncluded] = useState<JsonApiResource[]>([]);
```

**useEffect — fetch both in parallel:**
```typescript
Promise.all([
  getMySummaries(user.sub, { limit: 100 }),
  getMyImageSummaries(user.sub),
]).then(([textData, { summaries: imgData, included: imgIncluded }]) => {
  setSummaries(textData);
  setImageSummaries(imgData);
  setImageIncluded(imgIncluded);
})
.catch((e) => setError(e.message))
.finally(() => setLoading(false));
```

**Header stats — combine both:**
```typescript
const stats = useMemo(() => ({
  totalAttempts: summaries.length + imageSummaries.length,
  uniqueActivities: new Set([
    ...uniqueActivityIds(summaries),
    ...imageSummaries
      .map((s) => (s.relationships?.field_activityid?.data as any)?.id)
      .filter(Boolean),
  ]).size,
  totalTime: [...summaries, ...imageSummaries]
    .reduce((sum, s) => sum + (s.attributes.field_time ?? 0), 0),
}), [summaries, imageSummaries]);
```

**Group text summaries by activity UUID** (to render one chart per activity):
```typescript
const summariesByActivity = useMemo(() => {
  const map = new Map<string, ActivitySummaryResource[]>();
  for (const s of summaries) {
    const aid = (s.relationships?.field_activityid?.data as any)?.id ?? 'unknown';
    if (!map.has(aid)) map.set(aid, []);
    map.get(aid)!.push(s);
  }
  return map;
}, [summaries]);
```

**Render layout:**
```
Mis estadísticas
[Intentos] [Actividades] [Tiempo total]

── Ejercicios de lectura ──
  (one ActivityEvolutionChart per activity UUID, label "Actividad 1", "Actividad 2"…)

── Posición de imágenes ──
  ImagePositionEvolutionChart (or empty state)

── Historial por sesión ──
  SessionComparisonTable (text summaries only)
```

**New imports needed in `statistics/index.tsx`:**
- `ActivityEvolutionChart` from `@/components/statistics/ActivityEvolutionChart`
- `ImagePositionEvolutionChart` from `@/components/statistics/ImagePositionEvolutionChart`
- `ImageSummaryResource` from `@/types/activitysummary`
- `getMyImageSummaries, computeAccuracy` from `@/lib/api/statistics`

---

## What NOT to change

- `SessionComparisonTable` — leave as-is
- `StandardComparison`, `SessionProgressBar` — not used here, don't touch
- Drupal backend — no changes needed

---

## Verification

1. User installs victory-native: `cd shine-app && nvm use 21 && npx expo install victory-native`
2. Open `http://localhost:8081/statistics`
3. Header cards show combined intentos (≥ 65) and time
4. "Ejercicios de lectura" section shows one `ActivityEvolutionChart` per activity (VictoryLine + data table)
5. "Posición de imágenes" section shows time bar chart + accuracy bar chart (both at 100% for 2 data points)
6. `npx tsc --noEmit` passes with no errors
