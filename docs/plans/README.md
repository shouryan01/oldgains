# Agent Implementation Plans (`v0.1` – `v1.5`)

Each file in this directory is a **self-contained, step-by-step implementation plan** designed to be handed directly to an AI coding agent (e.g., *"Implement `docs/plans/v0.1-foundation-and-local-sqlite.md`"*).

After the agent finishes a release and passes all automated checks, use the **Manual Verification Checklist** at the bottom of that plan to test the features yourself before moving to the next release.

---

## Release Plan Index

| Release | Implementation Plan File | Linear Milestone | Focus |
| :--- | :--- | :--- | :--- |
| **`v0.1`** | [v0.1-foundation-and-local-sqlite.md](file:///Users/shouryan/Programming/gains/docs/plans/v0.1-foundation-and-local-sqlite.md) | `v0.1` | Monorepo, Oxc, UI Shell, OPFS SQLite Web Worker & Demo Mode |
| **`v0.2`** | [v0.2-exercise-catalog.md](file:///Users/shouryan/Programming/gains/docs/plans/v0.2-exercise-catalog.md) | `v0.2` | 5,632-Exercise Catalog FTS5, 17-Region SVG Body Map & Custom Exercises |
| **`v0.3`** | [v0.3-routine-builder-and-scheduling.md](file:///Users/shouryan/Programming/gains/docs/plans/v0.3-routine-builder-and-scheduling.md) | `v0.3` | Routine CRUD, Starter Plans, Weekday/Rotation Schedule & PDF Export |
| **`v0.4`** | [v0.4-active-workout-and-progression.md](file:///Users/shouryan/Programming/gains/docs/plans/v0.4-active-workout-and-progression.md) | `v0.4` | Active Workout Player, Rest Timer, Plate Math, Pocket Coach & Progression |
| **`v0.5`** | [v0.5-gym-analytics-and-body-map.md](file:///Users/shouryan/Programming/gains/docs/plans/v0.5-gym-analytics-and-body-map.md) | `v0.5` | 3-Mode Muscle Map (MEV/MAV/MRV, Fatigue, Untrained), 1RM Charts & Heatmap |
| **`v0.6`** | [v0.6-body-weight-and-csv-import.md](file:///Users/shouryan/Programming/gains/docs/plans/v0.6-body-weight-and-csv-import.md) | `v0.6` | MacroFactor Trend Weight (EWMA), Tape Measurements & CSV Import/Export |
| **`v0.7`** | [v0.7-nutrition-core-and-tdee.md](file:///Users/shouryan/Programming/gains/docs/plans/v0.7-nutrition-core-and-tdee.md) | `v0.7` | Calories/Protein UI, Gram Scaling, Barcode Scanner & Dynamic TDEE Engine |
| **`v0.8`** | [v0.8-nutrition-ai-vision-and-ocr.md](file:///Users/shouryan/Programming/gains/docs/plans/v0.8-nutrition-ai-vision-and-ocr.md) | `v0.8` | BYOK Gemini Flash-Lite Photo-to-Meal, Label OCR & Text Food Logger |
| **`v0.9`** | [v0.9-ai-coach-and-mcp.md](file:///Users/shouryan/Programming/gains/docs/plans/v0.9-ai-coach-and-mcp.md) | `v0.9` | AI Coach (Proposal Diffs + Undo), Local/Remote MCP Server & OpenAPI Plugin |
| **`v0.10`** | [v0.10-cloud-sync-auth-and-ci.md](file:///Users/shouryan/Programming/gains/docs/plans/v0.10-cloud-sync-auth-and-ci.md) | `v0.10` | Cloudflare Worker + D1, Better Auth (Passkeys), HLC Delta Sync & CI/CD |
| **`v1.0`** | [v1.0-ios-app-core.md](file:///Users/shouryan/Programming/gains/docs/plans/v1.0-ios-app-core.md) | `v1.0` | Native SwiftUI iOS App + `SQLiteData` (`GRDB.swift`) + Workouts & Sync |
| **`v1.1`** | [v1.1-ios-healthkit-live-activities-nutrition.md](file:///Users/shouryan/Programming/gains/docs/plans/v1.1-ios-healthkit-live-activities-nutrition.md) | `v1.1` | Apple HealthKit, Lock Screen Live Activities, Widgets & Mobile Nutrition |
| **`v1.2`** | [v1.2-apple-watch-companion.md](file:///Users/shouryan/Programming/gains/docs/plans/v1.2-apple-watch-companion.md) | `v1.2` | watchOS App: Digital Crown Logging, Multi-Stage Haptics & Live HR |
| **`v1.3`** | [v1.3-fitness-wrapped.md](file:///Users/shouryan/Programming/gains/docs/plans/v1.3-fitness-wrapped.md) | `v1.3` | Port `ios/fitness-wrapped` to Web & iOS Story Slides + Shareable Cards |
| **`v1.4`** | [v1.4-exercise-media-cdn.md](file:///Users/shouryan/Programming/gains/docs/plans/v1.4-exercise-media-cdn.md) | `v1.4` | Cloudflare R2 Exercise Media CDN (`still/*.webp` & `clip/*.mp4`) |
| **`v1.5`** | [v1.5-progress-photos.md](file:///Users/shouryan/Programming/gains/docs/plans/v1.5-progress-photos.md) | `v1.5` | On-Device EXIF Stripper, Private R2 Upload & Timeline Scrubber (Backlog) |
