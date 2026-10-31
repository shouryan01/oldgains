# 04 — Nutrition, Trend Weight & Adaptive TDEE Specification

## 1. Calories & Protein-First Philosophy

Unlike bloated macro trackers that clutter the screen with 30 micronutrients, Gains is streamlined around the two variables that drive 95% of body composition results: **Calories** and **Protein** (plus precise gram-weight food entry).
- **Primary Dashboard**: Twin iOS-style gradient rings/bars for **Daily Calories (Remaining / Target)** and **Daily Protein (g / Target)**.
- **Optional Carbs & Fat**: Stored when available from barcodes/AI, hidden by default on the main view, and toggleable in Settings for users who want full macro visibility.

---

## 2. Fast Food Logging Flows

1. **Gram-Weight Scaling & Saved Meals**:
   - Enter any food item and type its weight on your kitchen scale in **grams** (e.g., `240g`) to scale calories and protein linearly from its per-100g or per-serving base.
   - **Saved Meals / Combos**: Group frequent items (e.g., *"Morning Oatmeal & Whey"*) for 1-tap logging.
   - **"Explode Meal"**: Log a saved meal and tap **Explode** to break it into its individual ingredient rows so you can tweak today's gram weights without altering the saved template.
   - **1-Tap Copy Yesterday**: Copy an entire meal slot or full day from yesterday.
   - **Quick Add**: Directly type raw `Calories` + `Protein (g)` when dining out.
2. **Browser Camera Barcode Scanner + Open Food Facts**:
   - Uses the browser's native `BarcodeDetector` API (with a lightweight `zxing-wasm` fallback for browsers lacking native support) to scan UPC/EAN barcodes via phone or webcam.
   - Queries the free **Open Food Facts API** (`https://world.openfoodfacts.org/api/v2/product/{barcode}.json`), caches the normalized `calories_per_100g`, `protein_per_100g`, and serving size locally in SQLite `food_items`, and immediately opens the gram-weight input.
3. **Gemini 2.5 Flash-Lite AI Logging (Photo, Label OCR & Text)**:
   - **Photo-to-Meal Estimation**: Snap or upload a photo of a plate (optionally with a short note like *"cooked in 1 tbsp olive oil, ~250g chicken"*). Gemini 2.5 Flash-Lite returns a structured Zod array of detected items with estimated gram weights, calories, protein, and confidence ranges so you can adjust sliders before saving.
   - **Nutrition Facts Label OCR**: When a barcode isn't in Open Food Facts (or has no barcode), snap a photo of the package's **Nutrition Facts table**. Gemini Flash-Lite extracts exact serving size (grams), calories, and protein in `<2 seconds` and saves it to your local `food_items` library.
   - **Natural Language Text Entry**: Type *"3 eggs, 2 slices sourdough toast, and 150g Greek yogurt"* to parse into structured meal entries.

---

## 3. MacroFactor-Inspired Weight & Adaptive TDEE Physics Engine

### 3.1 Scale Weight vs. Exponentially Smoothed "Trend Weight"
Daily scale weight fluctuates wildly by $1\text{–}5\text{ lbs}$ due to glycogen, water retention, sodium, and gut weight.
Gains calculates and plots both **Scale Weight** (dots/dashed line) and **Trend Weight** (solid gradient curve) using an **Exponentially Weighted Moving Average (EWMA)** with time-gap awareness:

$$\text{Trend}_t = \alpha_d \cdot \text{Scale}_t + (1 - \alpha_d) \cdot \text{Trend}_{t-1}$$

where $\alpha_d = 1 - (1 - \alpha_0)^{\Delta d}$, $\alpha_0 = 0.15$ for daily weigh-ins, and $\Delta d$ is the number of days since the previous weigh-in.
- **True Weekly Rate of Change**: Computed from the slope of $\text{Trend Weight}$ over the last 7 and 14 days.

### 3.2 Dynamic Expenditure (TDEE) Algorithm
Instead of guessing activity multipliers, Gains continuously back-calculates your true daily energy expenditure from conservation of energy over a rolling 14-day window:

$$\Delta E_{\text{tissue}} = \frac{\text{Trend}_t - \text{Trend}_{t-14}}{14} \times 3500\text{ kcal/lb}$$

$$\text{Raw TDEE}_t = \overline{\text{Calories}}_{14\text{d, complete}} - \Delta E_{\text{tissue}}$$

To prevent sudden spikes, $\text{Raw TDEE}_t$ is smoothed with a secondary EWMA ($\beta = 0.2$) to produce the displayed **Daily Expenditure (kcal/day)** curve.

### 3.3 Partial-Day Logging Guardrail (Adherence-Neutral)
A common failure mode of naive TDEE calculators is when a user logs breakfast (`450 kcal`), forgets to log dinner, and the algorithm assumes they ate `450 kcal`—crashing their estimated TDEE.
- **Explicit Toggle**: Every day has a **"Mark Day Incomplete / Partial"** toggle.
- **Automatic Heuristic Warning**: If a past day's logged calories are $< 50\%$ of `targetCalories`, the app flags it with a prompt (*"Looks like an incomplete log — exclude from TDEE math?"*) and substitutes the rolling average intake for that day in the expenditure equation.

### 3.4 Diet Phases, Protein Scaling & Weekly Strategy Check-In
- **Goal Modes**: `Cut` (e.g., $-0.5\%$ to $-0.75\%$ BW/week), `Lean Bulk` ($+0.2\%$ to $+0.35\%$ BW/week), `Maintenance / Recomp` ($0.0\%$ BW/week).
- **Protein Recommendation**:
  - If **Body Fat %** is logged: $1.0\text{–}1.2\text{g per lb of Lean Body Mass (LBM)}$, where $\text{LBM} = \text{TrendWeight} \times (1 - \text{BF\%})$.
  - Otherwise: $0.8\text{–}1.0\text{g per lb of Trend Weight}$ (or goal weight if overweight).
- **Weekly Check-In Card**: Every Monday (or user-selected check-in day), shows a non-judgmental review card:
  - Previous vs. New Estimated TDEE (with Confidence Score $0\text{–}100\%$)
  - 7-Day Trend Weight change vs. Goal Rate
  - **1-Tap "Accept New Targets"** or **"Keep Current Targets"**.
