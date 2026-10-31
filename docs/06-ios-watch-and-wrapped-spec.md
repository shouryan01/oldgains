# 06 — Native iOS, Apple Watch & Fitness Wrapped Specification

## 1. Native iOS Application (`ios/Gains`)

### Architecture & Local Persistence
- **Language & UI**: Swift 6 + SwiftUI (iOS 18+ target), built for personal deployment via Xcode / TestFlight and open-source compilation.
- **SQLite Persistence**: **`SQLiteData` (`GRDB.swift`)**
  - Uses `@Table` value-type `struct` models and `@FetchAll` / `@FetchOne` property wrappers in SwiftUI views.
  - Executes the **exact same SQL migration files** from `packages/db/migrations` and the same `exercises_fts` SQLite FTS5 virtual table as the Web and Cloudflare D1 databases.
  - Runs the same HLC Delta Sync (`POST /api/sync/push`, `GET /api/sync/pull`) in the background via `URLSession` and `BGAppRefreshTask`.

### iOS Platform Integrations
1. **Lock Screen Live Activity & Dynamic Island (`ActivityKit`)**:
   - While a workout is active, displays a Live Activity on the iPhone Lock Screen and Dynamic Island showing:
     - Current Exercise Name, Set Number, Target Weight $\times$ Reps
     - Live Rest Timer countdown bar
     - Interactive App Intent buttons (**Complete Set**, **+15s Rest**, **Skip Rest**) so you can log sets without unlocking your phone.
2. **Apple Health (`HealthKit`) Bi-Directional Sync**:
   - **Writes to HealthKit**: Completed workouts (`HKWorkout`), active energy burned, and dietary energy/protein logged in Gains.
   - **Reads from HealthKit**: Morning scale weigh-ins & body fat % (automatically feeding the Gains **Trend Weight & TDEE** engine) and heart rate samples during workouts.
3. **Home Screen & Lock Screen Widgets (`WidgetKit`)**:
   - **Today's Plan Widget**: Shows today's scheduled routine with its mini anatomical muscle map.
   - **Calories & Protein Ring Widget**: Shows remaining daily calories and protein with a 1-tap Quick Log shortcut.
   - **Consistency Streak Heatmap Widget**: Compact 12-week workout grid.
4. **Optional Gym Arrival Reminder (`CoreLocation` Geofence, Off by Default)**:
   - Geofences user-saved gym coordinates strictly on-device; sends a local notification to start today's scheduled routine (max once per day, suppressed if a workout was already logged).

---

## 2. Apple Watch Companion App (`ios/GainsWatch`)

### Architecture
- **Standalone + Mirrored Execution**:
  - Uses `WatchConnectivity` (`WCSession`) + watchOS 10+ `HKWorkoutSession` mirroring.
  - Works seamlessly when paired with the iPhone in real time **or** completely standalone if the iPhone is left in the locker (syncing completed sets when back in range or over Wi-Fi).

### Watch Workout UX
1. **Routine Selection & Start**:
   - Pick today's scheduled routine, next rotation workout, or start a Freestyle session from the wrist.
2. **Digital Crown Set Logging**:
   - Focuses on `Weight` or `Reps` input with crisp haptic detents on every turn of the Digital Crown, snapping to the exercise's configured weight increments.
   - Displays `"Last: 185 × 8"` and `"Target: 185 × 9"` directly above the Complete Set button.
   - Optional 1-tap **RIR / RPE effort selector** right after completing a set.
3. **Multi-Stage Haptic Rest Timer**:
   - Full-screen progress ring countdown.
   - Fires a gentle warning haptic (`WKHapticType.directionUp`) at **15 seconds remaining** ("get under the bar") and a strong completion haptic (`WKHapticType.notification`) at **0:00**.
4. **Mid-Workout Wrist Shortcuts**:
   - **Crown / Sweat Lock**: Locks touch/crown input during pushups or wrist-wrap sets until held to unlock.
   - **1-Tap Backup Exercise Swap**: Swap an occupied machine to its configured backup exercise from the wrist menu.
   - **Instant PR Trophy Alert**: Full-screen celebratory haptic badge when a set breaks a 1RM, weight, or volume PR.
5. **Live Biometrics per Set (`HKLiveWorkoutBuilder`)**:
   - Captures live Heart Rate during each working set (saving `heart_rate_avg` to `workout_sets`), displays current HR zone, and closes Apple Watch Move & Exercise rings.
6. **Watch Face Complications (`WidgetKit`)**:
   - Circular, Corner, and Rectangular complications for: *Next Workout Routine*, *Weekly Workout Streak*, and *Remaining Protein / Calories*.

---

## 3. Fitness Wrapped (`v1.3` — Web & iOS)

Integrates and expands the existing `ios/fitness-wrapped` project into a first-class feature across **both Web and iOS**:

### Shared Analytics Engine (`packages/shared/src/wrapped.ts` + Swift port)
Computes wrapped metrics for any selectable period (*Year in Review*, *Last 90 Days*, *Custom Date Range*):
1. **Total Volume Lifted (Tonnage)** with fun real-world equivalents (e.g., *"You lifted 1,420,000 lbs — equivalent to 3.5 Boeing 747s or 95 African Elephants"*).
2. **Consistency & Streaks**: Total hours in the gym, longest weekly streak, busiest day of the week, and peak training hour ("Early Bird" vs. "Night Owl").
3. **Top Exercises & Favourite Muscle Groups**: Most-logged movements, highest 1RM gainers, and a full-year anatomical Muscle Map showing your "Archetype" (e.g., *"Push Specialist"*, *"Balanced Hypertrophy"*, *"Leg Day Loyalist"*).
4. **PR Hall of Fame**: Biggest breakthroughs of the year with date and weight progression timeline.
5. **Nutrition & Body Composition Highlights** (if nutrition/weight logged): Total protein consumed, most-logged meal, and Trend Weight transformation.
6. **Interactive Story Player & Shareable Cards**:
   - Full-screen tap-through story slides with gradient animations and exporting of high-resolution 9:16 social share cards (PNG).
