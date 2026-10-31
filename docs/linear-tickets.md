# Linear Project, Milestones & Tickets Specification

> **How to use this file**: Copy the prompt and specification below and paste it to an AI agent with Linear MCP access (or into Linear's import/agent interface) to automatically create the **Gains** project, its **15 Release Milestones (`v0.1` – `v1.5`)**, and all **54 granular engineering tickets**.

---

## Copy-Paste Agent Prompt for Linear

```markdown
Please set up the Linear project and tickets for **Gains** (our all-in-one local-first fitness & nutrition tracker) using the specification below:

1. Create (or select) the Project named **"Gains"** in my Linear team.
2. Create the **15 Project Milestones** (`v0.1` through `v1.5`) in exact chronological order.
3. Under each Milestone, create the listed Issues with their Title, Description, Acceptance Criteria, Labels, and Priority.
4. Preserve the logical dependencies between issues (each release builds on the previous one).

Reminder for our Git & PR workflow when working on these tickets:
- Branch Naming: Include the Linear ticket ID in the branch name (e.g., `ENG-101-monorepo-setup` or `feature/ENG-101-monorepo-setup`).
- PR Title: Keep it short and in an affirmative/imperative voice. Leave the Linear ticket ID out of the PR title.
- PR Description: Keep it descriptive, short, and memorable. Leave Linear ticket references out of the PR description and do not include a "Verification" section.
```

---

## Milestone 1: `v0.1 — Foundation, Monorepo, UI Shell & Local SQLite` (Priority: Urgent / P1)

### Issue 1.1: Initialize pnpm monorepo, TypeScript, Vitest, and Oxc toolchain
- **Labels**: `infra`, `dx`
- **Description**: Set up the root `pnpm-workspace.yaml` with `apps/web`, `apps/api`, `packages/db`, and `packages/shared`. Configure `oxlint`, `oxfmt`, shared `tsconfig.base.json`, and `vitest` workspace runners.
- **Acceptance Criteria**:
  - `pnpm lint`, `pnpm format:check`, `pnpm typecheck`, and `pnpm test` execute cleanly across all workspaces.

### Issue 1.2: Implement shared Drizzle SQLite schema and migration bundle (`packages/db`)
- **Labels**: `database`, `sqlite`
- **Description**: Define all core SQLite tables (`exercises`, `equipment_profiles`, `routines`, `routine_exercises`, `schedule_config`, `workouts`, `workout_exercises`, `workout_sets`, `body_measurements`, `food_items`, `saved_meals`, `nutrition_days`, `nutrition_entries`, `tdee_snapshots`, `ai_proposals`, `progress_photos`, `sync_outbox`) in `packages/db` with UUIDv7, `updated_at`, `hlc`, and `deleted_at` columns.
- **Acceptance Criteria**:
  - Drizzle generates clean SQLite migrations and exports TypeScript/Zod types for all tables.

### Issue 1.3: Build browser OPFS SQLite Web Worker engine & reactive query hook (`apps/web`)
- **Labels**: `web`, `database`
- **Description**: Set up `@sqlite.org/sqlite-wasm` inside a dedicated Web Worker using the `opfs-sahpool` VFS (no COOP/COEP headers required) connected to `drizzle-orm/sqlite-proxy`. Create a reactive `useLiveSQLiteQuery()` hook that invalidates subscribed queries on table mutations.
- **Acceptance Criteria**:
  - Database persists across page reloads in OPFS and executes read/write queries in `<5ms`.

### Issue 1.4: Scaffold Web App UI shell, Tailwind v4 theme, and Demo Mode switcher
- **Labels**: `web`, `ui`
- **Description**: Initialize `apps/web` with React 19, Vite, Tailwind CSS v4, Shadcn UI (`@base-ui/react`), and TanStack Router. Build the responsive iOS-inspired navigation shell (Bottom Tab Bar on mobile, Sidebar on desktop) and the **Web Demo Mode** banner with 1-click sample data seeding and database reset.
- **Acceptance Criteria**:
  - App renders all top-level routes (`Home`, `Workout`, `Routines`, `Exercises`, `Nutrition`, `Analytics`, `Settings`) and toggles Demo seed data cleanly.

---

## Milestone 2: `v0.2 — Exercise Catalog (5,632 Exercises) & Custom Exercises` (Priority: High / P1)

### Issue 2.1: Build exercise catalog compiler and SQLite FTS5 search index
- **Labels**: `catalog`, `database`
- **Description**: Create `scripts/build-catalog.ts` to validate and compile the 5,632 base exercises (and link the 922 female variants) from `catalog/exercises/*.json` into a compact seed bundle and populate the `exercises_fts` SQLite FTS5 virtual table.
- **Acceptance Criteria**:
  - Searching across all 5,632 exercises by keyword, target muscle, equipment, or category returns ranked results in `<5ms`.

### Issue 2.2: Create interactive 17-region Front/Back SVG Anatomical Muscle Map component
- **Labels**: `web`, `ui`, `charts`
- **Description**: Build a reusable SVG anatomical body map supporting all 17 `muscleMap` keys (`chest`, `upper-back`, `lower-back`, `trapezius`, `deltoids`, `biceps`, `triceps`, `forearm`, `abs`, `obliques`, `serratus`, `gluteal`, `quadriceps`, `hamstring`, `adductors`, `hip-flexors`, `calves`) with continuous `0.0–1.0` color gradient intensities and interactive region selection.
- **Acceptance Criteria**:
  - Renders both large detail views and compact mini-icons for routines and workouts.

### Issue 2.3: Build Exercise Library page with multi-facet filtering and Exercise Detail view
- **Labels**: `web`, `feature`
- **Description**: Implement the searchable Exercise Library with filters for Body Part, Target Muscle (clickable on the SVG body map), Equipment, and Category. Build the Exercise Detail drawer displaying instructions, description, muscle involvement map, and history/1RM tabs.
- **Acceptance Criteria**:
  - Virtualized list scrolls smoothly through 5,600+ exercises; clicking any exercise shows its full metadata and muscle map.

### Issue 2.4: Implement Custom Exercise CRUD modal
- **Labels**: `web`, `feature`
- **Description**: Allow users to create, edit, and soft-delete custom exercises with custom names, equipment, category, timed-hold or unilateral flags, instructions, and interactive primary/secondary muscle selection on the body map.
- **Acceptance Criteria**:
  - Custom exercises appear immediately in FTS5 search and routine pickers.

---

## Milestone 3: `v0.3 — Routine Builder, Starter Plans & Scheduling` (Priority: High / P1)

### Issue 3.1: Build Routine CRUD editor with set templates, supersets, and backup exercises
- **Labels**: `web`, `feature`
- **Description**: Create the Routine Builder supporting exercise reordering, superset/circuit grouping, warmup/working rest timers, pre-assigned Backup Exercise slots, set types (`warmup`, `working`, `dropset`, `rest_pause`, `amrap`, `timed`, `per_side`, `bodyweight`), and a live Routine Muscle Map summary.
- **Acceptance Criteria**:
  - Users can create, duplicate, edit, and delete routines with live fractional muscle volume feedback.

### Issue 3.2: Configure per-exercise Progression Rules and Starter Routine Templates
- **Labels**: `web`, `domain`
- **Description**: Add the Progression Rule selector per routine exercise (`Linear`, `Greyskull LP`, `Double Progression`, `Triple Progression`, `Timed`, `None`) and bundle 5 editable starter plans (`Push/Pull/Legs`, `Upper/Lower`, `Full Body`, `5×5`, `Greyskull LP`).
- **Acceptance Criteria**:
  - Starter plans can be imported in 1 click as standard editable routines.

### Issue 3.3: Implement Schedule Planner (Weekday Mode vs. Rolling Rotation Mode)
- **Labels**: `web`, `feature`
- **Description**: Build the Plan & Schedule screen supporting Fixed Weekday schedules (with Monday/Sunday start, A–D routine cycling per weekday, and single-week session moving) and Rolling Rotation queues (`A → B → C`).
- **Acceptance Criteria**:
  - Home screen accurately displays today's scheduled routine (with mini muscle map) or next rotation session.

### Issue 3.4: Add Routine JSON sharing and printable PDF Workout Sheet export
- **Labels**: `web`, `feature`
- **Description**: Allow exporting any routine or split as a compact shareable JSON file or a formatted printable PDF workout log sheet.
- **Acceptance Criteria**:
  - Exported JSON imports cleanly; PDF view prints a clean table of exercises, sets, target reps, and rest times.

---

## Milestone 4: `v0.4 — Active Workout Player, Pocket Coach & Progression Engine` (Priority: Urgent / P1)

### Issue 4.1: Build Active Workout Player (Guided & Freestyle modes)
- **Labels**: `web`, `feature`
- **Description**: Implement the live workout logging screen supporting Guided Routine sessions and Freestyle workouts. Pre-fill target weights/reps, show `Last Time: Weight × Reps @ RIR`, support swipe/tap set actions (copy set, delete set with Undo, change set type), colour-coded RPE/RIR picker, and live running volume progress bar.
- **Acceptance Criteria**:
  - All set edits persist immediately to local SQLite; an unfinished workout resumes automatically if the browser tab is closed and reopened.

### Issue 4.2: Implement Rest Timer with Screen Wake Lock and visual gym flash
- **Labels**: `web`, `feature`
- **Description**: Build the automatic countdown timer triggered on set completion (using warmup vs. working rest defaults), `navigator.wakeLock` screen-awake integration, `+15s / -15s / Skip` controls, audio chime, browser notification, and optional full-screen visual flash for loud gyms.
- **Acceptance Criteria**:
  - Rest timer counts down accurately even if the tab is backgrounded.

### Issue 4.3: Build Plate Math Calculator, Machine Stack Snapper, and 1-Tap Exercise Swap
- **Labels**: `web`, `domain`
- **Description**: Implement the interactive Barbell/EZ/Trap/Smith Plate Calculator (based on the user's Equipment Profile), custom machine stack weight increments, and 1-tap mid-workout swap to either the configured Backup Exercise or a muscle-matched alternative.
- **Acceptance Criteria**:
  - Plate calculator renders per-side plates visually; swapping an exercise preserves the slot's set count and progression rules.

### Issue 4.4: Implement Progression Engine, Stall/Deload Detector, and BWS+ Pocket Coach (`packages/shared`)
- **Labels**: `domain`, `algorithms`
- **Description**: Implement and unit-test the progression math in `packages/shared` (Linear, Greyskull AMRAP, Double, Triple, Timed), stall detection (`N` missed sessions $\rightarrow$ `-10%` deload suggestion), and the BuiltWithScience+ intra-workout **Pocket Coach** banner (suggesting +60s rest or -5% load when an early working set hits `RIR 0`).
- **Acceptance Criteria**:
  - 100% unit test coverage in Vitest across all progression rules, stall edge cases, and intra-workout adjustments.

### Issue 4.5: Build Workout Complete Summary screen with PRs, Muscle Map, and Next Targets
- **Labels**: `web`, `ui`
- **Description**: Create the post-workout celebration and summary view showing duration, total volume, sets completed, new PR badges (1RM, Weight, Volume, Reps), session anatomical Muscle Map, and next session's auto-calculated progression targets with 1-tap override toggles.
- **Acceptance Criteria**:
  - Completing a workout updates personal records and writes next session's target weights/reps back to the routine.

---

## Milestone 5: `v0.5 — Gym Analytics, 3-Mode Muscle Map & Gradient Charts` (Priority: High / P1)

### Issue 5.1: Build 3-Mode Anatomical Muscle Map Dashboard (Volume Landmarks, Fatigue, Untrained)
- **Labels**: `web`, `charts`, `domain`
- **Description**: Implement the 3 analytical modes on the SVG Body Map: (1) Weekly Fractional Effective Sets vs. science-based MEV/MAV/MRV hypertrophy zones (`10–20 sets`), (2) Muscle Fatigue & Recovery Decay (48–72h exponential model), and (3) Untrained Muscles (14-day window).
- **Acceptance Criteria**:
  - Secondary muscles are weighted accurately via `muscleMap.weights` and plotted alongside horizontal landmark bars.

### Issue 5.2: Implement Gradient 1RM Curves, Structural Balance Ratios, and Year Heatmap
- **Labels**: `web`, `charts`
- **Description**: Using Shadcn Charts (Recharts) with iOS-style SVG gradients, build the Estimated 1RM progression curves, Structural Balance radar/ratios (Push:Pull, Quad:Hamstring, Squat:Deadlift:Bench:OHP, % of Bodyweight), and the 52-week Workout Activity Heatmap.
- **Acceptance Criteria**:
  - Charts filter dynamically by time range (`1M`, `3M`, `6M`, `1Y`, `All`) and populate exercise history tabs.

### Issue 5.3: Build Completed Workout History editor and backdated workout logger
- **Labels**: `web`, `feature`
- **Description**: Allow viewing, editing, moving dates, or deleting any past completed workout, as well as logging a backdated paper workout. Automatically recalculate personal records (`personal_records`) when historical sets are modified.
- **Acceptance Criteria**:
  - Editing a historical set updates 1RM curves and PR badges deterministically.

---

## Milestone 6: `v0.6 — Body Weight (Trend Weight), Measurements & CSV Import/Export` (Priority: Medium / P1)

### Issue 6.1: Implement MacroFactor Trend Weight (EWMA) engine and Weight Goal Chart
- **Labels**: `domain`, `charts`
- **Description**: Build the time-gap-aware Exponentially Weighted Moving Average (`Trend Weight`) calculator in `packages/shared` and render the dual-line Scale Weight (dots) vs. Trend Weight (gradient curve) chart against the user's Goal Weight and weekly rate target.
- **Acceptance Criteria**:
  - Unit tests verify EWMA smoothing across daily and multi-day gaps; chart displays weekly trend rate of change.

### Issue 6.2: Build Body Fat % and Tape Measurements tracker
- **Labels**: `web`, `feature`
- **Description**: Create the Body Measurements logging view for Body Fat % and 13 anatomical circumference sites (`waist`, `chest`, `shoulders`, `neck`, `hips`, `biceps`, `forearms`, `thighs`, `calves`) with historical trend sparklines.
- **Acceptance Criteria**:
  - Measurements persist to `body_measurements` and plot over time.

### Issue 6.3: Build Universal CSV/JSON Importer (Strong, Hevy, FitNotes, Apple Health) & Backup Exporter
- **Labels**: `web`, `domain`
- **Description**: Implement parsers in `packages/shared` to import workout histories from Strong, Hevy, and FitNotes CSVs (mapping exercise names to catalog IDs with fuzzy matching) and weigh-ins from Apple Health exports. Add 1-click full JSON, CSV, and raw `.sqlite` database backup export.
- **Acceptance Criteria**:
  - Sample CSVs from Strong, Hevy, and FitNotes import cleanly with accurate volume and PR computation.

---

## Milestone 7: `v0.7 — Nutrition Core, Barcode Scanner & Adaptive TDEE` (Priority: High / P2)

### Issue 7.1: Build Calories & Protein-First Dashboard, Gram Scaling, and Saved Meals
- **Labels**: `web`, `nutrition`
- **Description**: Create the Nutrition dashboard with dual gradient progress rings for Daily Calories & Protein (with optional Carbs/Fat toggle in Settings), gram-weight food scaling, Custom Foods, Saved Meals ("Copy Yesterday", "Explode Meal" into individual ingredients), and Quick Add.
- **Acceptance Criteria**:
  - Entering gram weights scales calories and protein instantaneously; saved meals can be logged or exploded in 1 tap.

### Issue 7.2: Implement Browser Camera Barcode Scanner & Open Food Facts integration
- **Labels**: `web`, `nutrition`
- **Description**: Build the camera barcode scanner modal using native `BarcodeDetector` + `zxing-wasm` fallback. Query Open Food Facts (`world.openfoodfacts.org`), normalize per-100g and serving nutrition, and cache results locally in SQLite `food_items`.
- **Acceptance Criteria**:
  - Scanning a product barcode populates its name, brand, serving size, calories, and protein and caches it for offline reuse.

### Issue 7.3: Implement Dynamic TDEE Expenditure Engine, Partial-Day Guardrail, and Weekly Check-In
- **Labels**: `domain`, `nutrition`, `charts`
- **Description**: Implement the physics-based Rolling 14-Day Expenditure (TDEE) calculator ($\overline{\text{Calories}} - \Delta\text{TrendWeight} \times 3500$), the Partial-Day logging guardrail (explicit toggle + `<50%` target auto-detection), the Expenditure & Confidence chart, and the Weekly Strategy Check-In card (scaling protein by Lean Body Mass / Trend Weight).
- **Acceptance Criteria**:
  - Partial logging days are excluded from TDEE math; Weekly Check-In accurately computes updated calorie and protein targets for Cut, Lean Bulk, or Maintenance phases.

---

## Milestone 8: `v0.8 — Nutrition AI (Gemini Flash-Lite Vision, Label OCR & Text)` (Priority: High / P2)

### Issue 8.1: Build BYOK AI Provider Layer & Settings (`Gemini 2.5 Flash-Lite`, OpenAI, Anthropic, Ollama)
- **Labels**: `ai`, `web`, `api`
- **Description**: Configure the Vercel AI SDK (`ai`) provider router supporting Google Gemini 2.5 Flash-Lite (default free tier), OpenAI, Anthropic, and local Ollama endpoints, with a master "Enable AI Features" switch in Settings.
- **Acceptance Criteria**:
  - Users can test their API key in Settings or toggle all AI UI elements completely off.

### Issue 8.2: Implement AI Photo-to-Meal Estimator with portion adjustment sliders
- **Labels**: `ai`, `nutrition`
- **Description**: Allow snapping/uploading a meal photo (with optional text note) to Gemini 2.5 Flash-Lite, returning a structured Zod array of detected foods, estimated gram weights, calories, protein, and confidence scores with interactive gram sliders before saving.
- **Acceptance Criteria**:
  - Photo analysis returns structured ingredient breakdowns that scale live as the user adjusts gram weights.

### Issue 8.3: Implement Nutrition Facts Label Camera OCR and Natural Language Meal Logger
- **Labels**: `ai`, `nutrition`
- **Description**: Add a "Scan Nutrition Label" camera mode that extracts exact serving size (g), calories, and protein from package tables via Gemini Flash-Lite OCR, plus a text box for natural language food logging (e.g., *"250g chicken breast and 150g rice"*).
- **Acceptance Criteria**:
  - Label OCR saves reusable custom `food_items`; natural language input logs multi-item meals in one step.

---

## Milestone 9: `v0.9 — AI Coach, MCP Server & External Assistant Plugins` (Priority: Medium / P2)

### Issue 9.1: Build AI Coach Context Assembler & Proposal Diff / Undo Engine
- **Labels**: `ai`, `domain`
- **Description**: Implement `packages/shared/src/ai-context.ts` to summarize the user's active plan, review-window workout logs, RPE/stalls, muscle volume, Trend Weight, and TDEE. Build the `ai_proposals` state machine (`pending`, `approved`, `rejected`, `undone`) and visual Proposal Diff Card with 1-click Undo rollback.
- **Acceptance Criteria**:
  - AI Coach proposals never mutate user routines or nutrition targets until approved, and clicking Undo restores the exact prior snapshot.

### Issue 9.2: Build In-App AI Plan Architect and Weekly Training/Nutrition Review
- **Labels**: `ai`, `web`
- **Description**: Create the interactive 5-question Plan Architect (generating full multi-day splits using valid catalog exercise IDs) and the Weekly Review workflow that suggests exercise swaps, volume tweaks, and deloads.
- **Acceptance Criteria**:
  - Generated routines validate against catalog exercise IDs and render a combined weekly Muscle Map preview before import.

### Issue 9.3: Implement Local STDIO MCP Server (`packages/mcp`), Remote Worker MCP, and OpenAPI Plugin Spec
- **Labels**: `ai`, `mcp`, `api`
- **Description**: Build `packages/mcp` exposing tools (`search_exercises`, `get_training_summary`, `get_nutrition_and_tdee`, `log_workout_session`, `log_nutrition_items`, `log_body_weight`, `propose_routine_change`) over both local STDIO and `/api/mcp` on Cloudflare Workers, plus `/api/openapi.json` for ChatGPT Custom GPT Actions.
- **Acceptance Criteria**:
  - Claude Desktop, Gemini CLI, and ChatGPT can query training/nutrition history and log workouts/meals via API Token auth.

---

## Milestone 10: `v0.10 — Cloudflare D1 Delta Sync, Better Auth & CI/CD` (Priority: High / P2)

### Issue 10.1: Stand up Cloudflare Worker (`apps/api`), D1 database binding, and Better Auth
- **Labels**: `api`, `auth`, `cloudflare`
- **Description**: Configure `apps/api` with Hono and Cloudflare D1. Integrate **Better Auth** supporting Passkeys (Face ID / Touch ID), Email/Password, QR/code device pairing, and Personal API Tokens (`@better-auth/api-key`).
- **Acceptance Criteria**:
  - Users can sign up/sign in with Passkeys or Email and generate API tokens for MCP/plugins.

### Issue 10.2: Implement HLC Field-Level Delta Sync (`POST /api/sync/push`, `GET /api/sync/pull`)
- **Labels**: `api`, `database`, `sync`
- **Description**: Implement the Hybrid Logical Clock (HLC) push/pull sync endpoints on Hono + D1 and connect the client `sync_outbox` background worker. Include the "Merge Demo Data into Account" onboarding prompt when a Demo user signs up.
- **Acceptance Criteria**:
  - Concurrent offline edits across two browser sessions merge field-by-field without overwriting non-conflicting columns.

### Issue 10.3: Configure GitHub Actions CI/CD pipeline and Cloudflare Wrangler deployment
- **Labels**: `infra`, `ci`
- **Description**: Create `.github/workflows/ci.yml` running `oxlint`, `oxfmt --check`, `tsc`, and `vitest` on all PRs, plus automated D1 migrations and Worker deployment on merges to `main`.
- **Acceptance Criteria**:
  - CI pipeline runs in `<2 minutes` and deploys the SPA + Worker API to Cloudflare's free tier.

---

## Milestone 11: `v1.0 — Native iOS App Core (SwiftUI + SQLiteData / GRDB.swift)` (Priority: Medium / P3)

### Issue 11.1: Initialize Xcode project (`ios/Gains`) with `SQLiteData` (`GRDB.swift`) & HLC Sync
- **Labels**: `ios`, `database`
- **Description**: Create the native Swift 6 + SwiftUI iOS project configured with `SQLiteData` (`GRDB.swift`), running the shared SQLite schema, bundled 5,632-exercise FTS5 catalog, Better Auth sign-in/Passkeys, and background HLC Delta Sync to Cloudflare D1.
- **Acceptance Criteria**:
  - iOS app syncs workouts, routines, and nutrition bi-directionally with the Web app via Cloudflare D1.

### Issue 11.2: Build native SwiftUI Exercise Library, SVG Muscle Map, and Routine Builder
- **Labels**: `ios`, `ui`
- **Description**: Implement the native iOS Exercise Library (FTS5 search, equipment/muscle filters, instructions, 1RM charts), SwiftUI 17-region Muscle Map view, and Routine/Schedule Builder.
- **Acceptance Criteria**:
  - iPhone users can browse all 5,632 exercises offline and build/edit routines natively.

### Issue 11.3: Build native SwiftUI Active Workout Player, Rest Timer, and Progression Summary
- **Labels**: `ios`, `feature`
- **Description**: Implement the native iPhone Active Workout experience with haptic set completion, plate calculator sheet, RPE/RIR picker, 1-tap backup exercise swap, Pocket Coach nudges, and post-workout progression target calculation.
- **Acceptance Criteria**:
  - Full parity with the Web Active Workout player with native iOS haptics and fluid sheet transitions.

---

## Milestone 12: `v1.1 — iOS HealthKit, Lock Screen Live Activities, Widgets & Nutrition` (Priority: Medium / P3)

### Issue 12.1: Implement Lock Screen Live Activity & Dynamic Island workout controls (`ActivityKit`)
- **Labels**: `ios`, `feature`
- **Description**: Build an `ActivityKit` Live Activity displaying the active exercise, target weight/reps, live rest countdown, and interactive `AppIntent` buttons (`Complete Set`, `+15s`, `Skip Rest`) on the iPhone Lock Screen and Dynamic Island.
- **Acceptance Criteria**:
  - Users can tick off sets and control the rest timer directly from the Lock Screen without unlocking their phone.

### Issue 12.2: Integrate bi-directional Apple Health (`HealthKit`) sync
- **Labels**: `ios`, `healthkit`
- **Description**: Export completed workouts (`HKWorkout`), active calories, and dietary energy/protein to Apple Health; automatically import morning scale weigh-ins, body fat %, and workout heart rate samples into Gains.
- **Acceptance Criteria**:
  - Apple Health weigh-ins automatically update the user's Trend Weight and Dynamic TDEE.

### Issue 12.3: Build native iOS Nutrition Logger, Camera Barcode/OCR Scanner, and Home Screen Widgets
- **Labels**: `ios`, `nutrition`, `widgets`
- **Description**: Implement the native SwiftUI Nutrition & Adaptive TDEE views, `DataScannerViewController` / `AVFoundation` barcode & nutrition label camera scanner, and `WidgetKit` Home/Lock Screen widgets (Today's Routine, Calories/Protein Rings, Streak Heatmap).
- **Acceptance Criteria**:
  - Native camera scans barcodes in `<0.5s` and widgets update immediately when meals or workouts are logged.

---

## Milestone 13: `v1.2 — Apple Watch Companion App (ios/GainsWatch)` (Priority: Medium / P3)

### Issue 13.1: Build watchOS Workout Session engine with `WatchConnectivity` and `HKWorkoutSession` mirroring
- **Labels**: `watchos`, `healthkit`
- **Description**: Create the `ios/GainsWatch` target supporting both standalone wrist execution (when iPhone is in the locker) and real-time two-way mirroring with the iPhone app via `WatchConnectivity` and `HKWorkoutSession`.
- **Acceptance Criteria**:
  - Starting a workout or completing a set on the Watch immediately updates the iPhone app and Live Activity (and vice versa).

### Issue 13.2: Implement Digital Crown Set Logger, Multi-Stage Haptic Rest Timer, and Crown Lock
- **Labels**: `watchos`, `ui`
- **Description**: Build the wrist workout UI featuring Digital Crown weight/rep scrolling with haptic detents, `Last` vs. `Target` display, 1-tap RPE/RIR selector, Crown/Sweat Lock button, 1-tap Backup Exercise swap, and the multi-stage haptic rest timer (15s warning tap + 0:00 completion buzz).
- **Acceptance Criteria**:
  - Users can complete an entire routine from their wrist without touching their phone.

### Issue 13.3: Add per-set Live Heart Rate capture and Watch Face Complications (`WidgetKit`)
- **Labels**: `watchos`, `healthkit`, `widgets`
- **Description**: Capture live heart rate during each set via `HKLiveWorkoutBuilder` (saving `heart_rate_avg` to `workout_sets` and closing Apple Watch Move/Exercise rings) and build Watch Face Complications for Next Routine, Gym Streak, and Remaining Protein/Calories.
- **Acceptance Criteria**:
  - Completed sets display average HR and Watch complications refresh after each session.

---

## Milestone 14: `v1.3 — Fitness Wrapped (Ported to Web & iOS)` (Priority: Medium / P4)

### Issue 14.1: Build shared Fitness Wrapped aggregation engine (`packages/shared` & Swift)
- **Labels**: `wrapped`, `domain`
- **Description**: Port the metrics logic from `ios/fitness-wrapped` into a shared calculation engine computing Total Tonnage (with fun weight comparisons), Total Hours, Top Exercises, Muscle Archetype body map, PR Hall of Fame, Consistency Streaks, and Nutrition/Trend Weight highlights for any date range.
- **Acceptance Criteria**:
  - Engine generates complete Wrapped summary JSON from local SQLite workout and nutrition logs.

### Issue 14.2: Build interactive Story Slide Player and 9:16 Share Card Exporter on Web & iOS
- **Labels**: `wrapped`, `web`, `ios`
- **Description**: Create the full-screen animated story-slide experience (using `motion/react` on Web and SwiftUI animations on iOS) with tap/swipe navigation and 1-tap export of high-resolution 9:16 social share cards.
- **Acceptance Criteria**:
  - Users can replay their Year/Quarter Fitness Wrapped and download shareable summary cards on both Web and iPhone.

---

## Milestone 15: `v1.4 & v1.5 — Exercise Media CDN & Progress Photos` (Priority: Low / P5–P6 Backlog)

### Issue 15.1 (`v1.4`): Upload `catalog/media` (`webp` stills & `mp4` clips) to Cloudflare R2 and add lazy previews
- **Labels**: `cloudflare`, `catalog`, `web`, `ios`
- **Description**: Create a deployment script to sync `catalog/media/still/*.webp` (46 MB) and `catalog/media/clip/*.mp4` (170 MB) to Cloudflare R2 with immutable cache headers, and add lazy-loaded visual previews to the Exercise Library and Active Workout views.
- **Acceptance Criteria**:
  - Exercise thumbnails and looping demo clips load on demand from R2 CDN with zero impact on initial app bundle size.

### Issue 15.2 (`v1.5` — Low Priority Backlog): Implement on-device EXIF stripping, private R2 upload, and Progress Photo Timeline Scrubber
- **Labels**: `backlog`, `photos`, `web`, `ios`
- **Description**: Build the Progress Photos module with client-side EXIF/GPS metadata stripping, WebP compression, authenticated Cloudflare R2 upload, interactive video-style timeline scrubbing linked to Trend Weight on each date, and a Before/After split comparison slider.
- **Acceptance Criteria**:
  - Uploaded photos contain zero EXIF/location metadata and can be scrubbed chronologically alongside body weight trends.
