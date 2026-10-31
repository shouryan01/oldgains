# 02 — Database Schema & Offline Sync Specification

## 1. Shared SQLite Schema (`packages/db`)

All tables are defined in `packages/db/src/schema.ts` using `drizzle-orm/sqlite-core`. The generated SQL migrations run identically across:
1. **Browser SQLite** (`@sqlite.org/sqlite-wasm` OPFS Web Worker)
2. **Cloudflare D1** (Serverless SQLite)
3. **iOS & watchOS** (`SQLiteData` / `GRDB.swift`)

### Sync Metadata Convention
Every user-mutable table includes these standard columns:
- `id`: `text('id').primaryKey()` — UUIDv7 (time-ordered UUIDs for index locality)
- `userId`: `text('user_id').notNull()` — `'demo'` in Web Demo / offline-only mode, or the authenticated user's ID
- `createdAt`: `integer('created_at').notNull()` — Unix epoch milliseconds
- `updatedAt`: `integer('updated_at').notNull()` — Unix epoch milliseconds
- `hlc`: `text('hlc').notNull()` — Hybrid Logical Clock timestamp (`<iso_ms>:<counter>:<node_id>`)
- `deletedAt`: `integer('deleted_at')` — Nullable Unix epoch milliseconds (soft delete tombstone for cross-device sync)

---

## 2. Table Definitions

### 2.1 Exercise Catalog & Equipment (`exercises`, `equipment_profiles`)

```typescript
// exercises: Seeded from catalog/exercises/*.json (5,632 base exercises) + user custom exercises
export const exercises = sqliteTable('exercises', {
  id: text('id').primaryKey(), // '0025' for catalog entries, UUIDv7 for custom exercises
  userId: text('user_id'),     // null for built-in catalog, set for custom user exercises
  name: text('name').notNull(),
  bodyPart: text('body_part').notNull(), // waist | upper arms | back | chest | upper legs | shoulders | lower arms | lower legs | neck | full body | cardio
  equipment: text('equipment').notNull(), // barbell | dumbbell | cable | body weight | leverage machine | ...
  target: text('target').notNull(),       // pectorals | lats | quads | biceps | triceps | delts | ...
  secondaryMuscles: text('secondary_muscles', { mode: 'json' }).$type<string[]>().notNull(),
  category: text('category').notNull(),   // strength | calisthenics | mobility | cardio | olympic | stretching | ...
  description: text('description').notNull(),
  instructions: text('instructions', { mode: 'json' }).$type<string[]>().notNull(),
  muscleMap: text('muscle_map', { mode: 'json' }).$type<{
    primaries: string[];
    secondaries: string[];
    weights: Record<string, number>; // 17 anatomical keys weighted 0..1
  }>().notNull(),
  isHold: integer('is_hold', { mode: 'boolean' }).default(false).notNull(),
  isPerSide: integer('is_per_side', { mode: 'boolean' }).default(false).notNull(),
  needs: text('needs', { mode: 'json' }).$type<string[]>(),
  femaleVariantId: text('female_variant_id'),
  customMediaUrl: text('custom_media_url'),
  isCustom: integer('is_custom', { mode: 'boolean' }).default(false).notNull(),
  createdAt: integer('created_at').notNull(),
  updatedAt: integer('updated_at').notNull(),
  hlc: text('hlc').notNull(),
  deletedAt: integer('deleted_at'),
});
```

#### Full-Text Search Virtual Table (`exercises_fts`)
```sql
CREATE VIRTUAL TABLE IF NOT EXISTS exercises_fts USING fts5(
  id UNINDEXED,
  name,
  target,
  body_part,
  equipment,
  category,
  description,
  content='exercises',
  content_rowid='rowid',
  tokenize='porter unicode61'
);
```

```typescript
// equipment_profiles: Configures available plates, bars, and custom machine stacks
export const equipmentProfiles = sqliteTable('equipment_profiles', {
  id: text('id').primaryKey(),
  userId: text('user_id').notNull(),
  name: text('name').notNull(), // e.g., 'Commercial Gym', 'Home Garage'
  unit: text('unit').$type<'lb' | 'kg'>().default('lb').notNull(),
  barWeights: text('bar_weights', { mode: 'json' }).$type<{
    barbell: number;      // default 45 lb / 20 kg
    ezBar: number;        // default 25 lb / 10 kg
    trapBar: number;      // default 45 lb / 20 kg
    smithMachine: number; // default 15 lb / 7 kg
  }>().notNull(),
  availablePlates: text('available_plates', { mode: 'json' }).$type<number[]>().notNull(), // [45, 35, 25, 10, 5, 2.5, 1.25]
  machineStacks: text('machine_stacks', { mode: 'json' }).$type<Record<string, number[]>>(), // custom weight steps per exerciseId
  createdAt: integer('created_at').notNull(),
  updatedAt: integer('updated_at').notNull(),
  hlc: text('hlc').notNull(),
  deletedAt: integer('deleted_at'),
});
```

---

### 2.2 Routines, Plans & Schedules (`routines`, `routine_exercises`, `schedule_config`)

```typescript
export const routines = sqliteTable('routines', {
  id: text('id').primaryKey(),
  userId: text('user_id').notNull(),
  name: text('name').notNull(),          // e.g., 'Push A', 'Upper B'
  folder: text('folder'),                // e.g., 'PPL Split', 'Starter Plans'
  notes: text('notes'),
  sortOrder: integer('sort_order').default(0).notNull(),
  estimatedDurationMin: integer('estimated_duration_min'),
  musclePreview: text('muscle_preview', { mode: 'json' }).$type<Record<string, number>>(), // Cached muscle weights for mini body-map icon
  createdAt: integer('created_at').notNull(),
  updatedAt: integer('updated_at').notNull(),
  hlc: text('hlc').notNull(),
  deletedAt: integer('deleted_at'),
});

export const routineExercises = sqliteTable('routine_exercises', {
  id: text('id').primaryKey(),
  userId: text('user_id').notNull(),
  routineId: text('routine_id').notNull(),
  exerciseId: text('exercise_id').notNull(),
  backupExerciseId: text('backup_exercise_id'), // 1-tap swap when machine is taken
  sortOrder: integer('sort_order').notNull(),
  supersetGroupId: text('superset_group_id'),   // Exercises sharing this ID run back-to-back in a circuit
  restSeconds: integer('rest_seconds').default(120).notNull(),
  warmupRestSeconds: integer('warmup_rest_seconds').default(60).notNull(),
  notes: text('notes'),
  progressionRule: text('progression_rule', { mode: 'json' }).$type<{
    type: 'linear' | 'greyskull' | 'double' | 'triple' | 'time' | 'none';
    incrementLb: number;        // e.g., 5 or 2.5
    repRangeMin?: number;       // e.g., 8
    repRangeMax?: number;       // e.g., 12
    setRangeMax?: number;       // for triple progression, e.g., 4
    deloadPercent?: number;     // default 10%
    stallSessionsThreshold?: number; // default 2
  }>().notNull(),
  templateSets: text('template_sets', { mode: 'json' }).$type<Array<{
    setNumber: number;
    setType: 'warmup' | 'working' | 'dropset' | 'rest_pause' | 'amrap';
    targetWeightLb?: number;
    targetReps?: number;
    targetDurationSec?: number;
    targetRir?: number;
    restSecondsOverride?: number;
  }>>().notNull(),
  createdAt: integer('created_at').notNull(),
  updatedAt: integer('updated_at').notNull(),
  hlc: text('hlc').notNull(),
  deletedAt: integer('deleted_at'),
});

export const scheduleConfig = sqliteTable('schedule_config', {
  id: text('id').primaryKey(),
  userId: text('user_id').notNull(),
  mode: text('mode').$type<'weekday' | 'rotation'>().default('rotation').notNull(),
  weekStartsOn: integer('week_starts_on').default(1).notNull(), // 1 = Monday, 0 = Sunday
  // For weekday mode: maps weekday (0..6) to a list of routine IDs (supports cycling A->B on Upper day)
  weekdayMapping: text('weekday_mapping', { mode: 'json' }).$type<Record<number, string[]>>(),
  // For rotation mode: ordered queue of routine IDs [RoutineA, RoutineB, RoutineC]
  rotationQueue: text('rotation_queue', { mode: 'json' }).$type<string[]>(),
  currentRotationIndex: integer('current_rotation_index').default(0).notNull(),
  dateOverrides: text('date_overrides', { mode: 'json' }).$type<Record<string, string | null>>(), // 'YYYY-MM-DD' -> routineId or null (rest)
  createdAt: integer('created_at').notNull(),
  updatedAt: integer('updated_at').notNull(),
  hlc: text('hlc').notNull(),
  deletedAt: integer('deleted_at'),
});
```

---

### 2.3 Workouts, Sets & Personal Records (`workouts`, `workout_exercises`, `workout_sets`, `personal_records`)

```typescript
export const workouts = sqliteTable('workouts', {
  id: text('id').primaryKey(),
  userId: text('user_id').notNull(),
  routineId: text('routine_id'), // null if Freestyle workout
  name: text('name').notNull(),
  status: text('status').$type<'in_progress' | 'completed' | 'discarded'>().notNull(),
  startedAt: integer('started_at').notNull(),
  completedAt: integer('completed_at'),
  durationSeconds: integer('duration_seconds'),
  bodyweightLb: real('bodyweight_lb'),
  totalVolumeLb: real('total_volume_lb').default(0).notNull(),
  totalSets: integer('total_sets').default(0).notNull(),
  prCount: integer('pr_count').default(0).notNull(),
  notes: text('notes'),
  muscleSummary: text('muscle_summary', { mode: 'json' }).$type<Record<string, number>>(), // Fractional effective sets per muscle
  createdAt: integer('created_at').notNull(),
  updatedAt: integer('updated_at').notNull(),
  hlc: text('hlc').notNull(),
  deletedAt: integer('deleted_at'),
});

export const workoutExercises = sqliteTable('workout_exercises', {
  id: text('id').primaryKey(),
  userId: text('user_id').notNull(),
  workoutId: text('workout_id').notNull(),
  exerciseId: text('exercise_id').notNull(),
  swappedFromExerciseId: text('swapped_from_exercise_id'),
  sortOrder: integer('sort_order').notNull(),
  supersetGroupId: text('superset_group_id'),
  restSeconds: integer('rest_seconds').default(120).notNull(),
  notes: text('notes'),
  progressionRecommendation: text('progression_recommendation', { mode: 'json' }).$type<{
    nextWeightLb: number;
    nextReps: number;
    reason: string; // e.g., '+5 lb Linear Progression' or '-10% Deload after 2 stalls'
    userOverrideWeightLb?: number;
  }>(),
  createdAt: integer('created_at').notNull(),
  updatedAt: integer('updated_at').notNull(),
  hlc: text('hlc').notNull(),
  deletedAt: integer('deleted_at'),
});

export const workoutSets = sqliteTable('workout_sets', {
  id: text('id').primaryKey(),
  userId: text('user_id').notNull(),
  workoutId: text('workout_id').notNull(),
  workoutExerciseId: text('workout_exercise_id').notNull(),
  exerciseId: text('exercise_id').notNull(),
  setNumber: integer('set_number').notNull(),
  setType: text('set_type').$type<'warmup' | 'working' | 'dropset' | 'rest_pause' | 'amrap'>().default('working').notNull(),
  weightLb: real('weight_lb'),         // 0 or null for bodyweight; positive for added load
  reps: integer('reps'),
  leftReps: integer('left_reps'),      // for unilateral / per-side exercises
  rightReps: integer('right_reps'),
  durationSeconds: integer('duration_seconds'), // for timed holds/stretches/cardio
  distanceMeters: real('distance_meters'),
  rir: real('rir'),                    // Reps in Reserve (0..5)
  rpe: real('rpe'),                    // Rate of Perceived Exertion (5..10)
  estimated1RmLb: real('estimated_1rm_lb'),
  heartRateAvg: integer('heart_rate_avg'), // populated via Apple Watch HealthKit
  isCompleted: integer('is_completed', { mode: 'boolean' }).default(false).notNull(),
  completedAt: integer('completed_at'),
  prTypes: text('pr_types', { mode: 'json' }).$type<Array<'1rm' | 'weight' | 'volume' | 'reps'>>(),
  createdAt: integer('created_at').notNull(),
  updatedAt: integer('updated_at').notNull(),
  hlc: text('hlc').notNull(),
  deletedAt: integer('deleted_at'),
});
```

---

### 2.4 Body Measurements, Nutrition & AI Coach (`body_measurements`, `food_items`, `saved_meals`, `nutrition_days`, `nutrition_entries`, `tdee_snapshots`, `ai_proposals`, `progress_photos`)

```typescript
export const bodyMeasurements = sqliteTable('body_measurements', {
  id: text('id').primaryKey(),
  userId: text('user_id').notNull(),
  date: text('date').notNull(), // 'YYYY-MM-DD'
  scaleWeightLb: real('scale_weight_lb'),
  trendWeightLb: real('trend_weight_lb'), // MacroFactor EWMA smoothed weight
  bodyFatPct: real('body_fat_pct'),
  measurementsJson: text('measurements_json', { mode: 'json' }).$type<{
    neckIn?: number;
    shouldersIn?: number;
    chestIn?: number;
    waistIn?: number;
    hipsIn?: number;
    leftBicepIn?: number;
    rightBicepIn?: number;
    leftForearmIn?: number;
    rightForearmIn?: number;
    leftThighIn?: number;
    rightThighIn?: number;
    leftCalfIn?: number;
    rightCalfIn?: number;
  }>(),
  notes: text('notes'),
  createdAt: integer('created_at').notNull(),
  updatedAt: integer('updated_at').notNull(),
  hlc: text('hlc').notNull(),
  deletedAt: integer('deleted_at'),
});

export const nutritionDays = sqliteTable('nutrition_days', {
  id: text('id').primaryKey(),
  userId: text('user_id').notNull(),
  date: text('date').notNull(), // 'YYYY-MM-DD'
  targetCalories: integer('target_calories').notNull(),
  targetProteinG: integer('target_protein_g').notNull(),
  totalCalories: integer('total_calories').default(0).notNull(),
  totalProteinG: real('total_protein_g').default(0).notNull(),
  isPartial: integer('is_partial', { mode: 'boolean' }).default(false).notNull(), // Excludes incomplete days from TDEE calculation
  createdAt: integer('created_at').notNull(),
  updatedAt: integer('updated_at').notNull(),
  hlc: text('hlc').notNull(),
  deletedAt: integer('deleted_at'),
});

export const nutritionEntries = sqliteTable('nutrition_entries', {
  id: text('id').primaryKey(),
  userId: text('user_id').notNull(),
  date: text('date').notNull(), // 'YYYY-MM-DD'
  mealSlot: text('meal_slot').$type<'breakfast' | 'lunch' | 'dinner' | 'snack'>().notNull(),
  foodItemId: text('food_item_id'),
  name: text('name').notNull(),
  weightGrams: real('weight_grams'),
  servings: real('servings').default(1).notNull(),
  calories: integer('calories').notNull(),
  proteinG: real('protein_g').notNull(),
  carbsG: real('carbs_g'),
  fatG: real('fat_g'),
  source: text('source').$type<'manual' | 'barcode' | 'ai_photo' | 'ai_text' | 'ai_ocr' | 'saved_meal'>().notNull(),
  aiConfidence: real('ai_confidence'),
  createdAt: integer('created_at').notNull(),
  updatedAt: integer('updated_at').notNull(),
  hlc: text('hlc').notNull(),
  deletedAt: integer('deleted_at'),
});
```

---

## 3. Field-Level Delta Sync Engine (HLC + Outbox)

Inspired by *openGym*'s conflict-free device sync:

1. **Local Mutation & Outbox Recording**:
   - Whenever any record is inserted, updated, or soft-deleted on Web or iOS/Watch, the client generates a monotonically increasing **Hybrid Logical Clock (`hlc`)** string and records the changed column keys in `sync_outbox` (`table_name`, `record_id`, `changed_fields_json`, `hlc`).
2. **Push Phase (`POST /api/sync/push`)**:
   - Sends batched `sync_outbox` changes to the Cloudflare Worker.
   - For each record, the server compares the incoming `hlc` against the stored row in Cloudflare D1 and merges non-conflicting columns (or takes the newer `hlc` per field).
3. **Pull Phase (`GET /api/sync/pull?since_hlc=...`)**:
   - Fetches all rows across user tables where `hlc > last_pulled_hlc` and applies them to the local SQLite database inside a single transaction.
4. **Demo Mode Isolation**:
   - When `authMode === 'demo'`, the background sync loop is paused and all data stays in the browser's `gains-demo.sqlite` OPFS file. If the user later signs up, they can choose to either **Merge Demo Data into Account** or **Start Fresh**.
