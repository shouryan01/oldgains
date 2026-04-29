# Gains 🏋️

> *The first workout app that coaches you forward, not just records the past.*

A polished SwiftUI iOS + watchOS workout tracker built to feel like Apple designed it — fluid animations, native components throughout, and a companion Apple Watch app with live workout logging.

---

## What Makes Gains Different

| Feature | Description |
|---|---|
| **Animated Muscle Recovery Map** | Anatomical body silhouette that colors each muscle group green → amber → red based on training recency. Tap any muscle to see volume, last trained, and exercise suggestions. |
| **Strength Fingerprint** | Radar chart across 6 movement patterns (Push / Pull / Squat / Hinge / Carry / Core), normalized to bodyweight. Unique to you. Shareable. |
| **AI Progressive Overload Coach** | Prescribes exactly what to lift next session based on your history. Detects plateaus after 3 stalled sessions. |
| **Milestone Prediction Line** | Dotted projection on every 1RM chart showing when you'll hit your next milestone weight — with a confidence cone. |
| **Gains Wrapped** | Monthly + annual shareable cards (Instagram Stories format) pulling workouts, steps, calories, distance, and PRs from Apple Health. |
| **Volume Landmarks** | Track weekly sets against research-backed MEV / MAV / MRV thresholds per muscle group. |
| **Bevel-Inspired Logging** | Custom keyboard with large steppers, plate calculator, auto-fill from last session, and previous best shown inline. |

---

## Tech Stack

- **UI**: SwiftUI (iOS 17+, watchOS 10+)
- **Persistence**: SwiftData
- **Charts**: Swift Charts
- **Health**: HealthKit (read steps, calories, distance, HR, body weight; write workouts)
- **Phone ↔ Watch**: WatchConnectivity
- **Live Activities**: ActivityKit (Dynamic Island rest timer)
- **Animations**: SwiftUI + custom Canvas/TimelineView particle system
- **Concurrency**: Swift async/await

No third-party dependencies.

---

## Features

### Active Workout
- **Set types**: Warmup (W), Working (1), Drop Set (D), Failure (F), Cooldown (C), Myo-rep (M)
- **Supersets**: Long-press two exercises to group them; rest timer fires after the full round
- **Customizable rest timers**: Global defaults per set type + per-exercise overrides
- **Plate calculator**: Enter target weight → see exactly which plates to load
- **Live Activities**: Rest timer countdown on Lock Screen and Dynamic Island

### Progress & Analytics
- 1RM trend chart with prediction line and PR star markers
- Volume chart (working sets only, warmup excluded)
- Rep range donut (strength / hypertrophy / endurance zones)
- 52-week consistency heatmap
- Muscle recovery map (front + back toggle)
- Strength fingerprint radar
- Volume landmarks (MEV / MAV / MRV zones)
- PR timeline across all exercises

### Apple Health Integration
- Writes completed workouts, active calories
- Reads steps, distance, active calories, heart rate, body weight
- Gains Wrapped pulls all activity data from Health (not just what's logged in Gains)

### Apple Watch App
- View today's plan and coach suggestions
- Log sets from your wrist (crown to adjust reps)
- Full-screen rest timer with Digital Crown scrub
- Live heart rate badge during workouts
- Syncs bidirectionally with iPhone via WatchConnectivity

---

## Project Structure

```
gains/
  Gains.xcodeproj/
  Gains/
    App/              — GainsApp.swift, AppState.swift
    Models/           — SwiftData @Model classes
    ViewModels/       — WorkoutViewModel, ProgressViewModel, PlanViewModel
    Views/
      Today/
      History/
      Plans/
      Progress/
      Profile/
      Workout/        — Active workout full-screen
      Components/     — GainsKeyboard, MuscleBodyMap, StrengthFingerprint, etc.
    Services/         — HealthKit, WatchConnectivity, LiveActivity, OverloadCoach
    Resources/        — exercises.json (bundled exercise library)
  Gains Watch App/
  Scripts/
    transform_exercises.py   — maps ExerciseDB API response to app schema
```

---

## Getting Started

### Prerequisites
- Xcode 15+
- iOS 17+ device or simulator
- Apple Developer account (free works for simulator)
- Python 3 (for the exercise import script)

### 1. Clone the repo

```bash
git clone https://github.com/shouryan01/gains.git
cd gains
```

### 2. Create the Xcode project

See [`PLAN.md`](PLAN.md) → **Xcode Project Setup** for the full step-by-step. The short version:

1. Open Xcode → **File → New → Project → iOS → App**
   - Product Name: `Gains`
   - Interface: SwiftUI, Storage: SwiftData
   - Save **into this folder** (`gains/`)
2. Add a **watchOS App** target: `Gains Watch App`
3. Enable capabilities: HealthKit + Push Notifications (iOS), HealthKit (Watch)
4. Set deployment targets: iOS 17.0, watchOS 10.0

### 3. Import the exercise library

Get a free API key from [RapidAPI ExerciseDB](https://rapidapi.com/justin-WFnsXH_t6/api/exercisedb), then:

```bash
# Fetch 1,300+ exercises
curl -s "https://exercisedb.p.rapidapi.com/exercises?limit=1300&offset=0" \
  -H "X-RapidAPI-Key: YOUR_KEY" \
  -H "X-RapidAPI-Host: exercisedb.p.rapidapi.com" \
  | python3 -m json.tool > exercises_raw.json

# Transform to app schema
python3 Scripts/transform_exercises.py
```

Then add the generated `Gains/Resources/exercises.json` to the Xcode project (drag in, check "Add to target: Gains").

### 4. Build and run

Press **⌘R** in Xcode. On first launch the app seeds the exercise library from the bundled JSON.

---

## Implementation Roadmap

See [`PLAN.md`](PLAN.md) for the complete implementation plan including data models, screen designs, animation specs, and HealthKit integration details.

High-level build order:

1. Project setup + data models
2. Exercise seed data
3. Tab shell
4. Active workout core loop (set logging, keyboard, plate calculator)
5. Set types + supersets + rest timers
6. Workout complete (confetti, PR detection, shareable card)
7. History + Plans tabs
8. Progress charts (1RM prediction, recovery map, fingerprint, landmarks)
9. Overload Coach + plateau detection
10. Gains Wrapped (HealthKit-powered shareable cards)
11. Profile tab
12. HealthKit full integration
13. Live Activities (Dynamic Island)
14. Apple Watch app
15. Polish

---

## Contributing

This is a personal project. Feel free to open issues for ideas or bugs.

---

## License

MIT
