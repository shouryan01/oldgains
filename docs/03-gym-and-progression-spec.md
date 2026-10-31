# 03 — Gym Tracking, Progression & Analytics Specification

## 1. Exercise Library & Catalog (`catalog/`)

### Dataset Breakdown
- **Source**: `catalog/exercises/*.json` (6,554 JSON files: 5,632 base exercises + 922 linked female variants).
- **11 Body Parts**: `waist`, `upper arms`, `back`, `chest`, `upper legs`, `shoulders`, `lower arms`, `lower legs`, `neck`, `full body`, `cardio`
- **34 Equipment Types**: `barbell`, `dumbbell`, `cable`, `leverage machine`, `smith machine`, `ez barbell`, `trap bar`, `kettlebell`, `body weight`, `assisted`, `weighted`, `band`, `resistance band`, `suspension trainer`, `medicine ball`, `stability ball`, `bosu ball`, `wheel roller`, `landmine`, `weight plate`, `sled machine`, `rope`, etc.
- **12 Categories**: `strength`, `calisthenics`, `mobility`, `olympic`, `plyometrics`, `cardio`, `isometric`, `yoga`, `stretching`, `pilates`, `rehab`, `combat`
- **17 Anatomical `muscleMap` Keys**: `chest`, `upper-back`, `lower-back`, `trapezius`, `deltoids`, `biceps`, `triceps`, `forearm`, `abs`, `obliques`, `serratus`, `gluteal`, `quadriceps`, `hamstring`, `adductors`, `hip-flexors`, `calves`

### Exercise Detail View
Clicking any exercise opens a rich modal or route showing:
1. **Interactive SVG Muscle Map**: Highlights primary (`1.0`) and secondary (`0.2–0.6`) muscles using `muscleMap.weights`.
2. **Step-by-Step Instructions**: Numbered execution steps from `instructions[]` + description.
3. **Personal History & Records**:
   - Last session's logged sets (Weight $\times$ Reps @ RPE/RIR).
   - Best Weight PR, Best Volume Set PR, and Best Estimated 1RM.
   - **Estimated 1RM Progression Chart** (gradient area chart over time).
4. **Custom Exercise Creation**: Users can create custom exercises, pick primary/secondary muscles on the interactive body map, choose equipment/category, mark as timed (`hold: true`) or unilateral (`perSide: true`), and attach custom notes.

---

## 2. Routine Builder & Scheduling

### Starter Routine Templates
Bundled out-of-the-box as editable routines:
- **Push / Pull / Legs (PPL — 6-Day or Rotating 3-Day)**
- **Upper / Lower (4-Day Split)**
- **Full Body Hypertrophy (3-Day Split)**
- **5×5 Linear Strength (Workout A / Workout B)**
- **Greyskull LP (Phrak's Variant with AMRAP final sets)**

### Routine Capabilities
- **Schedule Modes**:
  1. **Fixed Weekday Schedule**: Assign routines to specific days of the week (supports Monday or Sunday week start). Supports cycling multiple routines on the same weekday (e.g., Monday = Upper cycles `Upper A → Upper B → Upper C`). Sessions can be dragged to another day for the current week without modifying the base template.
  2. **Rolling Rotation Queue (`A, B, C, A...`)**: No fixed weekdays—the home screen always queues up whichever routine is next in the sequence.
- **Exercise Slot Configuration**:
  - Drag-and-drop reordering of exercises.
  - **Supersets & Circuits**: Link 2+ exercises into a color-coded circuit block so completing Set 1 of Exercise A immediately advances focus to Set 1 of Exercise B before starting the rest timer.
  - **Set Types**: `Warmup (W)`, `Working (1, 2, 3...)`, `Drop Set (D)`, `Rest-Pause (RP)`, `AMRAP (+)`, `Timed Hold / Stretch (mm:ss)`, `Per-Side (L/R)`, and `0-weight Bodyweight` (with optional added dip-belt weight or assisted negative weight).
  - **Rest Timers**: Separate configurable default rest for warmup sets (e.g., 60s) vs. working sets (e.g., 120–180s), or custom per-set rest overrides.
  - **Backup Exercise**: Pre-assign a backup exercise per slot (or pick a smart muscle-matched swap on the fly) if a gym machine is occupied.
  - **Live Routine Muscle Map & Volume Preview**: As you add sets to a routine, a live mini body map and fractional set counter update to show which muscles the routine hits.
  - **Share & Export**: Duplicate routine, export/import as JSON, or print/save a clean formatted **PDF Workout Sheet**.

---

## 3. Active Workout Player & Smart Progression

### Guided & Freestyle Modes
- **Guided Session**: Pre-fills target weights and reps from the routine's progression rule + last completed performance.
- **Freestyle Workout**: Start an empty workout and add exercises/sets as you go; optionally save a completed freestyle session as a new routine at the end.

### In-Workout Features
- **Previous Performance Column**: Displays `Last: 185 lb × 8 @ RIR 1` right next to each set row.
- **Rest Timer Engine**:
  - Auto-starts when a set is checked off.
  - Keeps screen awake via the Screen Wake Lock API (`navigator.wakeLock`).
  - Optional **Visual Screen Flash** for loud gyms when the timer hits `0:00`, plus audio beep and browser notification.
  - Quick `-15s`, `+15s`, and `Skip` controls.
- **Plate Math & Custom Machine Stack Calculator**:
  - Tap any weight input on a barbell, EZ bar, trap bar, or Smith machine exercise to see the exact per-side plate breakdown based on your owned plates.
  - For leverage/cable machines with odd weight steps, snaps progression targets to the machine's configured weight stack.
- **BuiltWithScience+ "Pocket Coach" Intra-Workout Nudge**:
  - After logging a working set with an effort rating (`RIR` or `RPE`), if the user hits failure early (`RIR = 0` or `reps < targetRangeMin`) before the final set, a subtle inline coach badge suggests either **adding +60s rest** or **reducing weight by ~5%** on the next set to remain in the target hypertrophy rep range.
- **1-Tap Smart Exercise Swap**:
  - Swap to the pre-selected backup exercise or filter alternatives ranked by cosine similarity of `muscleMap.weights` (including a "Joint-Friendly / Non-Barbell" filter).

### Multi-Mode Progression Engine
Evaluated automatically upon finishing a workout to compute next session's targets:
1. **Linear Progression**: If all working sets meet or exceed target reps, increase load by `incrementLb` (+2.5 lb upper / +5 lb lower).
2. **Greyskull LP**: Final working set is `AMRAP (5+)`.
   - `< 5 reps`: Trigger automatic `-10%` deload for next session.
   - `5–9 reps`: Add standard `incrementLb` (+2.5 lb / +5 lb).
   - `10+ reps`: Add double increment (`2 * incrementLb`).
3. **Double Progression**: Target a rep range `[minReps, maxReps]` (e.g., `8–12`). Increment target reps each session until all sets hit `maxReps` at `RIR >= 1`, then add `incrementLb` and reset target reps to `minReps`.
4. **Triple Progression**: Progress reps to `maxReps` $\rightarrow$ add 1 working set up to `maxSets` $\rightarrow$ add `incrementLb` and reset to base sets and `minReps`.
5. **Timed Hold Progression**: Increases hold duration by `+5s` or `+10s` once all sets hit the target hold time.
6. **Stall Detection & Deloads**: Missed reps never increase load. If an exercise fails to progress for `stallSessionsThreshold` (default 2–3 consecutive sessions), the completion screen recommends a `-10%` deload with a 1-tap toggle to accept or hold weight.

---

## 4. Analytics, Charts & Anatomical Body Map

### 1. 3-Mode Anatomical Muscle Map (17 SVG Regions)
- **Mode A — Weekly Fractional Volume vs. Landmarks (BWS+ / RP Style)**:
  - Sums fractional hard sets (`setWeight = muscleMap.weights[muscle]` for working sets) over the last 7 days or current training week.
  - Color-codes muscles and renders a horizontal bar chart against hypertrophy landmarks:
    - **Under MEV (`< 6 sets`)**: Cool Blue / Slate
    - **Productive Hypertrophy Zone (`6–18 sets`, MAV)**: Vibrant Emerald / Green Gradient
    - **Approaching/Exceeding MRV (`18+ sets`)**: Amber / Crimson Alert
- **Mode B — Muscle Fatigue & Recovery State**:
  - Models local muscular fatigue using an exponential decay curve ($F(t) = F_0 \cdot e^{-t / \tau}$, where $\tau$ scales between 36h and 72h based on fractional volume and average set RPE). Shows which muscles are **Fresh (Green)**, **Recovering (Yellow)**, or **Fatigued (Red)** right now.
- **Mode C — Untrained / Neglected Muscles**:
  - Highlights muscles with zero effective volume over the past 10–14 days.

### 2. Strength & Volume Charts
- **Estimated 1RM Formula (Epley / Brzycki selectable)**:
  $$\text{1RM}_{\text{Epley}} = w \times \left(1 + \frac{r}{30}\right) \quad \text{(incorporating RIR: } r_{\text{eff}} = r + \text{RIR}\text{)}$$
- **Structural Balance Radar & Ratio Cards**:
  - Compares Horizontal Push vs. Horizontal Pull, Vertical Push vs. Vertical Pull, Quad vs. Hamstring volume, and classic strength ratios (Bench : Squat : Deadlift : Overhead Press, plus Lifts as a % of Body Weight).
- **Year Activity Heatmap**: GitHub-style 52-week contribution grid colored by workout volume or duration.
