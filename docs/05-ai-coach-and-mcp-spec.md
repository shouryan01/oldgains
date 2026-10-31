# 05 — AI Coach, MCP Server & Plugins Specification

## 1. Core AI Design Principles

1. **100% Optional & Off-by-Default**: Every feature in Gains works completely without AI. Enabling AI is a toggle in Settings.
2. **Never Mutates Data Without Consent (Approve / Reject / Undo)**:
   - Whether triggered in-app or via MCP/ChatGPT/Claude, the AI Coach **never** overwrites your routines or targets directly.
   - Instead, it creates a structured `ai_proposals` record containing a `before_snapshot_json`, `after_snapshot_json`, and human-readable `rationale_markdown`.
   - The UI renders a visual **Proposal Diff Card** (showing added/removed exercises, set/rep changes, or calorie/protein adjustments).
   - Once approved, the `before_snapshot_json` is retained so you can click **Undo** at any time to restore the exact prior state.
3. **Bring-Your-Own-Key (BYOK) & Free-Tier Default**:
   - Built on the **Vercel AI SDK (`ai`)** provider abstraction.
   - **Default Primary Provider**: **Google Gemini 2.5 Flash-Lite** (`@ai-sdk/google`), using Google AI Studio's free tier (1,000 free requests/day).
   - **Optional Providers**: OpenAI (`@ai-sdk/openai`), Anthropic (`@ai-sdk/anthropic`), or any OpenAI-compatible local/custom endpoint (e.g., **Ollama** running locally on Mac).
   - API keys can be stored either encrypted in the user's cloud profile or **strictly locally in browser/device storage** (so keys never touch the server if running client-direct calls).

---

## 2. What the AI Coach Reads (Context Window Payload)

When generating a new plan or running a weekly review, a deterministic context assembler (`packages/shared/src/ai-context.ts`) builds a compact, token-efficient JSON summary of:
1. **Your Plan & Equipment**:
   - Active routines, exercises, sets/reps, progression rules, weekly schedule/rotation, and available equipment profile.
2. **Your Logged Training (Review Window: Last 7–28 Days)**:
   - Completed sessions, weights, reps, times, RPE/RIR effort ratings, session durations, PRs hit, stalled exercises, fractional weekly sets per muscle group vs. MEV/MAV/MRV landmarks, and user session notes.
3. **Body Weight & Composition**:
   - Scale weigh-ins, **Trend Weight** trajectory, Body Fat % estimates, and goal weight / rate of gain or loss.
4. **Nutritional & TDEE History**:
   - Rolling calorie and protein averages, estimated Dynamic TDEE, and adherence consistency.

---

## 3. AI Coach Workflows

1. **Interactive Plan Architect (Onboarding / Split Redesign)**:
   - Answers 4–5 quick questions: *Days per week available*, *Session time cap (e.g., 45m vs 75m)*, *Available equipment*, *Primary goal & weak-point priority muscles*, and *Joint limitations*.
   - Generates a complete multi-day routine split using valid exercise IDs from the 5,632-exercise catalog, shows the combined weekly Muscle Map coverage, and lets the user import/save the plan.
2. **Weekly Training & Nutrition Review**:
   - Summarizes wins, identifies volume gaps (e.g., *"Rear delts and hamstrings only received 4 fractional sets this week"*), proposes exercise swaps for stalled lifts, and aligns next week's calorie/protein targets with the TDEE engine.

---

## 4. Model Context Protocol (MCP) & External AI Plugins

Gains supports connecting your fitness data to **Claude Desktop, Claude Mobile/Web, ChatGPT, Gemini CLI, and Cursor** through three interfaces:

### 4.1 Local STDIO MCP Server (`packages/mcp`)
Runs locally on your Mac (`npx @gains/mcp`) reading either your local SQLite database file or connecting to your Cloudflare Worker API via a Personal API Token.

### 4.2 Remote Cloudflare Worker MCP Endpoint (`/api/mcp`)
Hosted directly on your Cloudflare Worker using Bearer API Token authentication (generated via Better Auth in Settings $\rightarrow$ API Keys).

### 4.3 OpenAPI 3.1 Action Schema (`/api/openapi.json`)
Allows adding Gains as a **Custom GPT Action in ChatGPT** or a **Gemini Extension** using your Personal API Token.

### Exposed MCP Tools & Resources
| Tool Name | Mode | Description |
| :--- | :--- | :--- |
| `search_exercises` | Read | Search the 5,632-exercise catalog by query, target muscle, equipment, or category. |
| `get_training_summary` | Read | Returns routines, recent workout logs, 1RM estimates, and weekly fractional muscle volume for a date range. |
| `get_nutrition_and_tdee` | Read | Returns daily calorie/protein logs, Scale vs. Trend Weight, and current Dynamic TDEE estimate. |
| `log_workout_session` | Write | Logs a completed workout described in natural language (e.g., *"Log bench press 185x8, 185x8, 185x7 at RPE 9"*). |
| `log_nutrition_items` | Write | Logs food items, gram weights, calories, and protein to a specified meal slot and date. |
| `log_body_weight` | Write | Logs a scale weigh-in and optional body fat % or tape measurements, updating Trend Weight. |
| `propose_routine_change` | Write (Proposal) | Creates a reviewable `ai_proposals` diff for a new routine or progression tweak (requires user approval or provides undo token). |
