# Future-Release Plan: openGym Parity and Product Extensions

Date: 2026-10-14
Status: roadmap; each release becomes a separate implementation milestone.

## 1. Summary and release boundary

Build toward major openGym feature parity, adapted to native SwiftUI iPhone and Apple Watch apps and the React web companion. Deliver this through several future releases rather than one oversized update.

Initial-release baseline—not duplicated in this backlog:
- Core routines, exercise search, workout logging, rest timers, history, basic progress statistics, and backup/export.
- Account-free local use: SwiftData on Apple devices and localStorage on the web.
- Optional cloud account with Apple and email sign-in, cross-platform sync, and simultaneous iPhone/Watch workout controls.
- Phone-free Watch workouts.
- Live ChatGPT, Claude, and eligible Gemini integrations; AI-generated programs require review inside the app.
- Context export/program import for local-only AI use.
- Existing visual style, without user-selectable color schemes.

Future scope: advanced training, richer analysis, migration tools, body tracking, customization, embedded coaching, and relevant upstream account/administration capabilities. Android and social networking are not implied by feature parity.

## 2. Sequenced future releases

### Release 2 — Advanced training and planning

- Add weekday schedules and rolling routine rotations, including configurable week start, rescheduling, rest days, and one-day overrides.
- Add supersets, warm-up sets, drop sets, rest-pause, pyramid sets, and exercise-specific or set-specific rest periods.
- Support timed exercises, cardio duration/speed, bodyweight movements with optional added load, and per-side repetitions.
- Add optional RPE/RIR logging with clear explanations and preserved effort units in historical records.
- Implement progression policies: linear, Greyskull LP, double progression, triple progression, and time-based progression.
- Add stall detection, planned deloads, and explanations for prescribed targets.
- Add plate calculations based on bar type, bar weight, and owned plates.
- Extend Watch execution to support these workout types; keep complex program editing on iPhone/web.
- Add configurable workout-day reminders and missed-workout nudges. Respect quiet hours and platform notification permissions.

Acceptance: the same routine and history produce equivalent targets on web and Apple apps; offline Watch sessions preserve all supported set types and synchronize without duplicates.

### Release 3 — Progress analytics and body tracking

- Add estimated 1RM trends, selectable supported formulas, PR timelines, activity heatmaps, and exercise-level volume trends.
- Add muscle-volume, recovery/fatigue, and detraining maps, with calculation explanations and clear limits on their accuracy.
- Add structural-balance views matching upstream-supported comparisons.
- Add bodyweight logging, goal lines, measurement history, and progress photos with timeline and comparison views.
- Support editing historical workouts, adding past sessions, and correcting dates; recalculate derived records afterward.
- Add opt-in Apple Health integration for supported workout and bodyweight records.
- Keep raw heart-rate/Health data and progress photos outside AI access by default; require separate explicit sharing if introduced later.
- Store photos in platform-appropriate files/object storage, not localStorage. Preserve local-only use and provide media-inclusive backups.

Acceptance: corrected history updates statistics consistently; repeated Health writes do not create duplicates; denied Health permissions do not interrupt tracking.

### Release 4 — Imports, exercise customization, and sharing

- Add imports from FitNotes, Strong, Hevy CSV, and supported Apple Health weight exports.
- Add Hevy API import as a separate optional connection, subject to current API availability and terms.
- Add an openGym backup migration adapter for the pinned upstream data version.
- Provide import preview, exercise matching, unresolved-item review, duplicate detection, and a downloadable error report.
- Add custom exercises, equipment filters, notes, optional media, and reference links.
- Strip supported location/metadata from uploaded media and enforce file-type/size limits.
- Add routine/program sharing as a portable file and printable PDF.
- Add complete backup/restore including optional media, plus platform-appropriate automatic backup options.
- Keep restricted upstream exercise media out of public releases. Public animations remain gated on an independently valid license or original assets.

Acceptance: imports never silently discard unsupported records; repeated imports do not duplicate workouts; backup round trips preserve routines, history, settings, and included media.

### Release 5 — Themes, accessibility, and Apple polish

- Introduce user-selectable curated accent palettes with light, dark, and system appearance.
- Apply shared semantic design tokens across web, iPhone, and Watch while retaining platform-appropriate controls.
- Keep warning, success, and effort colors distinguishable from decorative accents.
- Add previews and synchronize appearance preferences for cloud users.
- Expand accessibility coverage: VoiceOver, Dynamic Type, keyboard navigation, reduced motion, contrast, and non-color status indicators.
- Add appropriate iPhone widgets, Live Activities, Watch complications, and configurable workout-screen layouts.
- Add upstream-style gym membership QR/barcode storage and display, with local handling and no default AI exposure.
- Introduce localization infrastructure, then translate the app and exercise text to match the pinned upstream language set, including right-to-left layouts.

Acceptance: themes meet contrast requirements; large text does not block workout actions; timers remain accurate after backgrounding; membership codes stay private.

### Release 6 — Coaching extensions and account maturity

- Expand external coaching tools with progression explanations, richer training summaries, and program-revision proposals.
- Add a proposal history showing provenance, changes, approval/rejection, and the resulting saved program.
- Detect proposals based on stale routine versions and require renewed review instead of overwriting newer edits.
- Add revocable assistant connections, scoped permissions, and a user-visible tool activity history.
- Publish assistant-specific setup instructions and reusable coaching guidance; do not assume one instruction format works identically across providers.
- Add embedded coaching only after a separate feasibility milestone establishes a supported inference mechanism and funding model. Existing consumer AI subscriptions must not be assumed to fund embedded API calls.
- Add account export/deletion, device/session management, recovery, and clear disconnect-to-local behavior wherever not already required for launch.
- Provide optional Docker-free self-hosting documentation, migrations, backup/restore procedures, and update guidance.
- Add optional invite-only registration and administrator access controls for self-hosted deployments. Administrative logs must avoid unnecessary health data.

Acceptance: assistants cannot directly apply programs or delete history; revoked connections lose access; cloud/account failures never prevent local workouts.

## 3. Shared implementation and interface work

- Maintain a versioned interchange contract for routines, workouts, exercise references, measurements, settings, and AI proposals.
- Add backward-compatible storage migrations for SwiftData, browser state, and cloud records; never require wiping existing user data.
- Keep training calculations equivalent across TypeScript and Swift using shared input/output test fixtures.
- Extend the authenticated API and MCP tools together, with account isolation and explicit read/proposal-write permissions.
- Keep draft proposals separate from active programs. Applying a proposal is an application action, not an assistant tool.
- Extend offline reconciliation to new records and media, preserving deletions and preventing duplicate Watch/phone sessions.
- Keep web preferences unchanged: current stable React/Vite, TypeScript, Tailwind, shadcn, and TanStack Router as the only TanStack package.
- Add CI checks that public source and release artifacts contain no restricted personal-use exercise media.

## 4. Verification and release gates

For every future release:
- Test fresh installs, upgrades from the previous release, backup restoration, and interrupted migrations.
- Verify local-only and cloud-connected paths on web, iPhone, and physical Watch hardware.
- Exercise offline edits, simultaneous phone/Watch control, retries, disconnects, and conflict recovery.
- Test malformed imports, duplicate requests, unauthorized account access, and assistant permission revocation.
- Run accessibility and visual checks for affected screens.
- Roll out through a test environment and TestFlight before public distribution.
- Document new capabilities, data migrations, known limitations, and recovery steps.

Cloud additions must include a cost estimate against the under-$25/month initial hosting target. Storage-heavy capabilities may require revised quotas or budget approval before release.

## 5. Assumptions and tracking rules

- Parity means equivalent user capabilities, not reproducing Docker deployment or identical controls on every platform.
- Pin an upstream release/commit when implementation begins and maintain a feature checklist against it; do not chase changing upstream scope mid-release.
- Core offline tracking remains available without accounts, AI, or paid media.
- Feature-parity work may reuse appropriately licensed upstream code/text with required notices; restricted media remains segregated.
- Cloud-provider selection is still a prerequisite of the initial architecture, not a future feature.
- Each release above becomes its own implementation milestone; embedded coaching and public exercise animations remain explicitly gated rather than promised without a viable delivery mechanism.
- Later extensions to validation, revocation, accessibility, and proposal handling do not defer essential initial-release safeguards.

Related plan: [initial-release.md](initial-release.md).
Version split (v0.11 onward): [versions.md](versions.md).
