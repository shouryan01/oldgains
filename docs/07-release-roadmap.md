# 07 — Phased Release Roadmap (`v0.1` – `v1.5`)

This roadmap slices **Gains** into **15 incremental, shippable releases** ordered strictly by priority:
1. **Priority 1 (`v0.1` – `v0.6`)**: Gym Tracking, Progression, Muscle Map & Analytics
2. **Priority 2 (`v0.7` – `v0.10`)**: Nutrition, Adaptive TDEE, AI Coach/MCP & Cloud Sync
3. **Priority 3 (`v1.0` – `v1.2`)**: Native iOS & Apple Watch Companion Apps
4. **Priority 4 (`v1.3`)**: Fitness Wrapped (Web + iOS)
5. **Priority 5 & 6 (`v1.4` – `v1.5`)**: Exercise Media CDN & Progress Photos (Backlog)

---

## Release `v0.1` — Foundation, Monorepo, UI Shell & Local SQLite (Demo Mode)
**Goal**: Stand up the monorepo, linting/formatting toolchain, iOS-inspired web navigation shell, and the in-browser OPFS SQLite database engine with 1-click Demo Mode seeding.
- **Deliverables**:
  - `pnpm` workspace (`apps/web`, `apps/api`, `packages/db`, `packages/shared`)
  - Oxc (`oxlint` + `oxfmt`), TypeScript strict config, and Vitest harness
  - React 19 + Vite + Tailwind CSS v4 + Shadcn Base UI + TanStack Router layout (Home, Workout, Routines, Exercises, Nutrition, Analytics, Settings)
  - `@sqlite.org/sqlite-wasm` Web Worker with `opfs-sahpool` VFS + Drizzle ORM schema & migration runner
  - **Web Demo Mode** banner with "Load Realistic Demo Data" and "Reset Demo Database" actions

---

## Release `v0.2` — Exercise Catalog (5,632 Exercises) & Custom Exercises
**Goal**: Compile the local `catalog/exercises/*.json` dataset into SQLite with FTS5 full-text search and build the Exercise Library & Custom Exercise flows.
- **Deliverables**:
  - Catalog compiler script (`scripts/build-catalog.ts`) normalizing all 5,632 base exercises (and linking the 922 female variants)
  - SQLite FTS5 virtual table (`exercises_fts`) for `<1ms` search by name, muscle, equipment, or category
  - Filterable Exercise Library page (Body Part, Target Muscle, Equipment, Category, Owned Equipment filter)
  - Interactive 17-region Front/Back **SVG Anatomical Muscle Map** component highlighting primary & secondary muscles
  - Exercise Detail drawer/page showing instructions, target muscles, and placeholder history/1RM tabs
  - Custom Exercise CRUD modal with interactive muscle weight selector

---

## Release `v0.3` — Routine Builder, Starter Plans & Scheduling
**Goal**: Enable building, organizing, scheduling, and exporting workout routines.
- **Deliverables**:
  - Routine CRUD with drag-and-drop exercise reordering and live Routine Muscle Map preview
  - 5 bundled Starter Plans (Push/Pull/Legs, Upper/Lower, Full Body, 5×5, Greyskull LP)
  - Set template configuration: Warmup, Working, Drop Set, Rest-Pause, AMRAP, Timed Hold, Per-Side, and 0-weight Bodyweight
  - Supersets / Circuit linking (`superset_group_id`), default warmup/working rest timers, and pre-assigned Backup Exercise slot
  - Progression Rule configuration per exercise (Linear, Greyskull LP, Double Progression, Triple Progression, Timed)
  - Schedule Planner supporting **Fixed Weekday Mode** (Monday/Sunday start, drag session to another day, cycle routines A–D on a weekday) and **Rolling Rotation Mode** (`A → B → C`)
  - Copy routine, JSON import/export, and printable **PDF Workout Sheet** generator

---

## Release `v0.4` — Active Workout Player, Pocket Coach & Progression Engine
**Goal**: Deliver a frictionless mid-workout logging experience with smart timers, plate math, and automatic progression calculations.
- **Deliverables**:
  - Guided Workout (from today's scheduled/next routine) and Freestyle Workout player
  - Pre-filled target weights/reps + inline display of `Last Time: Weight × Reps @ RIR`
  - Auto-starting Rest Timer with Screen Wake Lock, audio alert, optional loud-gym visual screen flash, and `+15s / -15s / Skip` controls
  - Optional colour-coded **RIR / RPE** selector per set with plain-English tooltips
  - **Plate Calculator** (barbell, EZ bar, trap bar, Smith machine) & custom **Machine Weight Stack** snapper
  - 1-Tap **Backup Exercise Swap** & smart similar-muscle replacement picker
  - **BuiltWithScience+ Intra-Workout "Pocket Coach"**: suggests +60s rest or -5% load if the user hits `RIR 0` (failure) on an early working set
  - Live session volume bar & real-time PR detection badges (Weight, Rep, Volume, Est. 1RM)
  - **Workout Complete Screen**: session duration, total tonnage, PR trophies, session Muscle Map, and **Next Session Progression Targets** (with 1-tap override & stall/deload alerts)

---

## Release `v0.5` — Gym Analytics, 3-Mode Muscle Map & Gradient Charts
**Goal**: Rich visual analytics across workout history, muscle volume landmarks, recovery state, and strength balance.
- **Deliverables**:
  - **3-Mode Anatomical Body Map**:
    1. *Weekly Fractional Sets vs. Hypertrophy Landmarks* (MEV / MAV / MRV bar chart & heatmap using `muscleMap.weights`)
    2. *Muscle Fatigue & Recovery State* (48–72h exponential recovery curve)
    3. *Untrained / Neglected Muscles* (14-day detector)
  - iOS-style gradient **Estimated 1RM Progression Charts** and per-exercise history inside the Exercise Detail view
  - **Structural Balance Ratios** (Horizontal Push:Pull, Vertical Push:Pull, Quad:Hamstring, Bench:Squat:Deadlift:OHP, Lifts as % of Bodyweight)
  - **Year Activity Heatmap** (52-week calendar grid)
  - Completed Workout History list with full post-workout editing and backdated paper-workout logging

---

## Release `v0.6` — Body Weight (Trend Weight), Measurements & CSV Import/Export
**Goal**: Track body composition with MacroFactor-style exponential smoothing and migrate historical data from other apps.
- **Deliverables**:
  - Daily Scale Weight logger + **MacroFactor Exponentially Smoothed Trend Weight** engine & dual-line gradient chart against Goal Weight & Weekly Rate
  - Body Fat % and 13-site tape measurement tracker (`waist`, `chest`, `biceps`, `thighs`, etc.)
  - **Universal Importer**: import historical workouts & weight logs from **Strong**, **Hevy**, **FitNotes**, and **Apple Health** CSV/XML exports
  - **Full Backup Exporter**: 1-click export of all SQLite tables as JSON or CSV spreadsheets (plus raw `.sqlite` file download)

---

## Release `v0.7` — Nutrition Core, Barcode Scanner & Adaptive TDEE
**Goal**: Launch the Calories & Protein-first nutrition tracker with barcode scanning and physics-based Dynamic TDEE coaching.
- **Deliverables**:
  - **Calories & Protein First UI**: dual gradient rings/bars, meal slots, and optional Carbs/Fat toggle in Settings
  - **Gram-Weight Food Scaling**, Custom Foods, **Saved Meals / Combos** ("Copy Yesterday", "Explode Meal" into ingredients), and Quick Add
  - Browser camera **Barcode Scanner** (`BarcodeDetector` + `zxing-wasm`) integrated with the free **Open Food Facts API** and local SQLite cache
  - **Dynamic Expenditure (TDEE) Engine**: back-calculates true daily burn from rolling calorie intake vs. $\Delta\text{Trend Weight}$, plotted with a TDEE expenditure curve and Data Confidence score
  - **Partial-Day Logging Guardrail**: explicit "Incomplete Day" toggle + automatic `<50%` calorie alert so partial logs never corrupt TDEE
  - **Weekly Strategy Check-In Card**: recommends updated calorie and lean-mass-scaled protein targets based on goal rate (Cut / Lean Bulk / Recomp)

---

## Release `v0.8` — Nutrition AI (Gemini Flash-Lite Vision, Label OCR & Text)
**Goal**: Add multimodal AI food estimation using the free tier of Gemini 2.5 Flash-Lite (or BYOK).
- **Deliverables**:
  - AI Settings panel with **Bring-Your-Own-Key (BYOK)** support (Gemini 2.5 Flash-Lite default, OpenAI, Anthropic, local Ollama) and master AI off-switch
  - **Photo-to-Meal Estimator**: snap/upload food photos (+ optional text context) to get itemized gram weights, calories, and protein with interactive adjustment sliders
  - **Nutrition Facts Label OCR**: scan any package's nutrition table with the camera to auto-populate serving grams, calories, and protein when a barcode is missing
  - **Natural Language Meal Logger**: parse free-text meal descriptions into structured nutrition entries

---

## Release `v0.9` — AI Coach, MCP Server & External Assistant Plugins
**Goal**: Enable the consent-driven AI Coach inside the app and expose Gains via MCP and OpenAPI for Claude, ChatGPT, and Gemini.
- **Deliverables**:
  - **AI Coach Plan Architect**: generates tailored routines from a 5-question interview (days/week, session duration, equipment, goals)
  - **AI Weekly Training & Nutrition Review**: reads your plan, logged sets/RPE, Trend Weight, and TDEE to propose load/volume/exercise adjustments
  - **Proposal Diff & 1-Click Undo System**: visual diff cards (`Approve` / `Reject`) backed by `ai_proposals` snapshots for instant rollback
  - **Local STDIO MCP Server (`packages/mcp`)** for Claude Desktop, Cursor, and Gemini CLI
  - **Remote Worker MCP Endpoint (`/api/mcp`)** + **OpenAPI 3.1 Action Spec (`/api/openapi.json`)** for ChatGPT Custom GPTs and Claude Web/Mobile

---

## Release `v0.10` — Cloudflare D1 Delta Sync, Better Auth & CI/CD
**Goal**: Connect the offline-first database to Cloudflare's 100% free tier with passkey authentication and conflict-free multi-device sync.
- **Deliverables**:
  - Cloudflare Worker (`apps/api`) with Hono RPC routes and Cloudflare D1 SQLite database binding
  - **Better Auth** integration supporting Passkeys (Face ID / Touch ID), Email/Password, QR/code device pairing, and Personal API Tokens
  - **HLC Field-Level Delta Sync (`POST /api/sync/push`, `GET /api/sync/pull`)** merging concurrent edits across devices without data loss
  - Seamless "Upgrade Demo to Account" flow
  - **GitHub Actions CI/CD** pipeline running `oxlint`, `oxfmt`, `tsc`, `vitest`, and deploying via Wrangler

---

## Release `v1.0` — Native iOS App Core (SwiftUI + `SQLiteData` / `GRDB.swift`)
**Goal**: Launch the native Swift iPhone app sharing the exact same SQLite schema and sync protocol.
- **Deliverables**:
  - Xcode project (`ios/Gains`) with Swift 6, SwiftUI, and `SQLiteData` (`GRDB.swift`) running `packages/db` migrations and `exercises_fts` search
  - Native iOS Exercise Library, Routine Builder, and Active Workout player with HLC background sync to Cloudflare D1

---

## Release `v1.1` — iOS HealthKit, Lock Screen Live Activities, Widgets & Nutrition
**Goal**: Deep iOS system integrations and native mobile nutrition/camera logging.
- **Deliverables**:
  - Bi-directional **Apple HealthKit** sync (workouts and calories out; morning scale weight, BF%, and workout HR in)
  - **Lock Screen Live Activity & Dynamic Island** (`ActivityKit`) showing current exercise, rest countdown, and interactive `Complete Set` / `+15s` buttons
  - **Home Screen & Lock Screen Widgets** (`WidgetKit`) for Today's Routine, Calories/Protein Rings, and Workout Streak
  - Native iOS Nutrition logger with AVFoundation camera barcode scanner and Gemini Flash-Lite Photo/OCR capture
  - Optional on-device **Gym Arrival Geofence Reminder** (off by default)

---

## Release `v1.2` — Apple Watch Companion App (`ios/GainsWatch`)
**Goal**: Full wrist-based workout control, Digital Crown set logging, and biometric tracking.
- **Deliverables**:
  - Standalone and iPhone-mirrored watchOS app via `WatchConnectivity` and `HKWorkoutSession`
  - Routine picker, **Digital Crown** weight/rep dialing with haptic detents, RPE/RIR picker, and **Crown / Sweat Lock** mode
  - **Multi-Stage Haptic Rest Timer** (15s warning tap + 0:00 completion buzz)
  - 1-Tap Backup Exercise Swap and PR celebration badge on the wrist
  - Live Heart Rate capture per working set (`heart_rate_avg`) + Apple Watch Move/Exercise ring closure
  - Watch Face Complications & Smart Stack widgets

---

## Release `v1.3` — Fitness Wrapped (Web + iOS)
**Goal**: Port `ios/fitness-wrapped` into a shared story-style recap experience on both Web and iOS.
- **Deliverables**:
  - Shared Wrapped aggregation engine (`packages/shared/src/wrapped.ts` + Swift) for any date window or full year
  - Interactive animated Story Player (Total Tonnage & fun weight equivalents, Top Exercises, Muscle Archetype body map, PR Hall of Fame, Consistency Heatmap, Nutrition & Trend Weight summary)
  - High-resolution 9:16 shareable card exporter

---

## Release `v1.4` — Exercise Media CDN (Cloudflare R2)
**Goal**: Add visual exercise demonstrations (`webp` stills and `mp4` loops) from `catalog/media/` with zero bundle bloat.
- **Deliverables**:
  - Upload script syncing `catalog/media/still/*.webp` (46 MB) and `catalog/media/clip/*.mp4` (170 MB) to Cloudflare R2 with immutable cache headers
  - Lazy-loaded thumbnail and looping video previews in Exercise Library, Routine Builder, and Active Workout on Web and iOS

---

## Release `v1.5` — Progress Photos & Timeline Scrubber *(Lowest Priority Backlog)*
**Goal**: Private progress photo tracking linked to Trend Weight and body measurements.
- **Deliverables**:
  - On-device EXIF & GPS metadata stripper + client-side WebP compression before uploading to private Cloudflare R2 storage
  - Interactive **Timeline Scrubber** to play back physique progress over time alongside that date's Trend Weight
  - **Before / After Split Comparison Slider** with date and weight overlays
