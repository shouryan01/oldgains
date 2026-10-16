# Versioned Release Plan

Date: 2026-10-14
Status: proposed split of [initial-release.md](initial-release.md) and [future-releases.md](future-releases.md). Each version is a separate, shippable increment. Decisions still open are listed at the end.

## Rules

- Versions are `v0.NN`. The initial release is v0.01 to v0.10. Later releases start at v0.11 and follow the Release 2 to 6 groups in [future-releases.md](future-releases.md).
- Each version lists scope, dependencies, and exit criteria. Anything not listed is out of scope for that version.
- Every version keeps existing user data. Schema changes are additive and go through versioned migrations.
- No public distribution before v0.10 (TestFlight first, per the initial plan). Earlier versions are private builds.

## Gates before v0.01

No app code ships in these. Each produces a written record in `docs/`.

- **G1 Inventory and licensing.** Pin the upstream openGym commit, record AGPL and NOTICE obligations, and decide what (if anything) is reused. The v0.01 exercise list is written in-house, so the upstream catalogue and media are not needed to start.
- **G2 Cloud provider decision record.** Compare options against the criteria in initial-release.md task 2 and record provider, hosting model, quotas, cost estimate (under $25/month), and recovery procedure. Must be complete before v0.03. v0.01 uses provider-neutral IDs, so it does not depend on this choice.
- **G3 Branding and repository.** Name, domain, repository destination, and distribution details. Needed before the first artifact leaves a private machine.

## Architecture seams fixed in v0.01

These are the parts that must be right at the start so later versions add features without rewrites.

| Seam | Decided in v0.01 | What it makes cheap later |
|---|---|---|
| Interchange contract | Versioned JSON schema (v1) for exercises, routines, sessions, sets, settings. Every record has `id` (UUID), `revision`, `updatedAt`, and `deletedAt` (tombstone). | Backup, import, sync (v0.06), and AI proposals reuse the same shapes. `revision` and `deletedAt` are unused until v0.06 but present, so no migration is needed. |
| Schema version and migrations | `schemaVersion` stored with the data. An ordered list of migration functions; v1 is the only one. | Each later schema change is one added migration. |
| Storage port | Domain code talks to a `StoragePort` (`read`, `write`). The store holds the snapshot in memory and writes it through the port; save failures are surfaced in the UI. v0.01 adapter: localStorage. | v0.02 SwiftData adapter, v0.04 account-scoped adapter. v0.06 moves to record-level get/put/delete with an outbox behind the same port. |
| Domain calculations | Pure functions (set volume, session totals, completion) in one module, tested against shared JSON fixtures in `fixtures/`. | Swift port (v0.02) and statistics (v0.03) use the same fixtures. |
| Feature modules | One folder per feature (`routines`, `workouts`, `history`, `backup`). Routes registered in one place. | New features add a folder and routes; core files stay unchanged. |
| Settings | Typed settings object with defaults. Unknown keys are preserved on import. | New settings (week start in v0.11, rest defaults, and so on) are additive. |
| Validation | One validator library, chosen in v0.01 and used for every import path. Malformed input is rejected before any write. | Backup, CSV imports (v0.22), and proposal import (v0.10) use the same path. |
| Public-build check | CI fails if restricted upstream media or catalogue files appear in the source tree or build output. | Extended to release artifacts in v0.25. |
| Account scope | No accounts in v0.01. Local data is one implicit profile. | Account isolation (v0.04) scopes the storage adapter to an account namespace; no record format change. |

## Initial release (v0.01 to v0.10)

### v0.01: Web, local-only minimum

**Scope**
- Seed exercise list (in-house names, no upstream media) with text search.
- Create, edit, and delete routines: name, ordered exercises, target sets and reps.
- Start a workout from a routine. Log reps and weight per set, mark sets done, run a fixed-duration rest timer.
- Finish a workout. History list of sessions with per-session volume.
- Export all data to JSON. Import with validation; a malformed file is rejected and stored data is unchanged.
- Visual direction taken from upstream. No color options.
- Stack: React, Vite, TypeScript, Tailwind, shadcn, TanStack Router (no TanStack Query or Start).

**Not yet:** statistics beyond session volume, accounts, Apple apps, Watch, AI, notifications.

**Depends on:** G1 (in-house exercise list), G3 (name).

**Exit criteria**
- A full workout completes offline; reloading the page restores every record.
- Export, then import, round-trips without change. An invalid file is rejected without writing.
- Calculation fixtures pass. CI public-build check passes.

### v0.02: iPhone local app

**Scope**
- SwiftUI and SwiftData app with the v0.01 feature set and visual direction.
- Swift calculation port that passes the same fixtures as the TypeScript code.
- Export and import of the v0.01 JSON file. Minimum iOS version is set here.

**Not yet:** statistics, Watch, sign-in.

**Depends on:** v0.01.

**Exit criteria**
- Fixtures pass in both TypeScript and Swift.
- A web export imports on iPhone, and an iPhone export imports on web.
- Data survives an app restart. Tested on a Mac simulator and a physical iPhone.

### v0.03: Basic progress statistics

**Scope**
- Statistics spec written first. Proposed starting set: session volume trend, per-exercise top set, workouts per week.
- Shown on web and iPhone. Backup covers all stored records, including statistics inputs.

**Depends on:** v0.02.

**Exit criteria**
- Both platforms produce identical statistics for the shared fixtures.
- Editing a set updates the affected statistics on both platforms.

### v0.04: Accounts

**Scope**
- Optional sign-in with Apple and with a passwordless email link.
- Account-scoped storage. Encrypted transport and storage.
- Sign-out. Account data export (JSON). Account deletion, including server records, within a retention window documented in G2.
- Account-free use is unchanged.

**Depends on:** G2, v0.03.

**Exit criteria**
- Automated test: one account cannot read another account's data.
- Deletion removes the account and its server records within the documented window.
- Local-only use works with no network.

### v0.05: Local-to-account migration (one-time upload)

**Scope**
- On first sign-in, show a preview: counts of routines, sessions, and sets to upload, plus any conflicts with existing account data.
- Upload happens only after the user confirms. Local data stays on the device until the user chooses to clear it.
- Uploads are idempotent (stable IDs), so a retry after interruption creates no duplicates.

**Depends on:** v0.04.

**Exit criteria**
- Cancelling the preview changes nothing on either side.
- A confirmed upload appears on a second device after reload.
- An interrupted upload retried produces no duplicate records.

### v0.06: Sync engine (web and iPhone)

**Scope**
- Use `revision` and `deletedAt` from v0.01. Outbox queue; retry-safe writes.
- Deletion propagates across devices.
- Visible connection status and count of unsent changes.
- Conflict policy decided at the start of this version (see open decisions). Candidate: merge at set level rather than overwrite whole records.
- Local copy stays usable when disconnected. Expired sign-in queues writes instead of discarding them.

**Depends on:** v0.05, G2 finalized.

**Exit criteria**
- Offline edits on both devices converge with no lost sets.
- An expired token queues writes and surfaces them; none are dropped.
- Disconnecting shows the unsent count and keeps the local copy intact.

### v0.07: Apple Watch, phone-free workouts

**Scope**
- watchOS app. Routines transferred from iPhone in advance (WatchConnectivity).
- Start a stored routine, edit reps and weight, complete sets, run rest timers, finish with the phone away.
- Unfinished and completed sessions persist on the Watch and queue for sync.
- Workout session and heart-rate behavior verified on hardware, with explicit Health permission. Broader Health integration is later (v0.21).
- Minimum watchOS version is set here.

**Depends on:** v0.02, v0.06.

**Exit criteria**
- A full workout on the Watch with the phone off.
- After reconnecting, exactly one intact session exists.
- Denying heart-rate permission does not block logging.

### v0.08: Phone and Watch live-session reconciliation

**Scope**
- Both devices can edit the same live session.
- Stable session and set IDs. Each operation carries an idempotency key, so replay has no effect.
- Duplicate sessions are merged, not duplicated.

**Depends on:** v0.07.

**Exit criteria**
- Simultaneous edits produce one session containing all sets.
- Replaying the same operation twice changes nothing.
- An app killed mid-workout recovers the session on relaunch.

### v0.09: External coaching, read-only

**Scope**
- Authenticated remote MCP server with OAuth.
- Scoped read tools: training history and routines. No write tools.
- Connection list and revocation.
- Setup instructions and packaging for ChatGPT, Claude, and eligible Gemini accounts.

**Depends on:** v0.04, v0.06.

**Exit criteria**
- Each supported assistant connects on a real account and reads authorized history.
- A revoked connection is denied on the next request.
- No tool can change routines or delete history.
- Assistant publication and Gemini eligibility are release gates. Gemini eligibility is rechecked at the start of this version.

### v0.10: Proposals, initial-release completion, public launch

**Scope**
- Draft-program create and revise tools. Drafts only.
- Proposal inbox: validation, change preview, stale-base detection (against routine revision), and explicit approval before any active program changes.
- Local-only users export selected context and import a proposal file with review. No automatic upload.
- Shared-data and revocation explanations in the UI.
- Basic accessibility checks on core flows (keyboard use, labels for VoiceOver).

**Depends on:** v0.09, v0.06.

**Exit criteria**
- Assistants submit drafts and change no routines before in-app approval.
- A proposal based on an outdated routine revision is flagged.
- Restricted media is absent from source and artifacts.
- TestFlight passes on physical devices. Initial release is public.

## Later releases (v0.11 onward)

Each row is a separate milestone and depends on the previous version unless noted.

### Release 2: Advanced training and planning

| Version | Scope |
|---|---|
| v0.11 | Weekday schedules, rolling rotations, configurable week start, rescheduling, rest days, one-day overrides. |
| v0.12 | Supersets, warm-up, drop, rest-pause, and pyramid sets; exercise-level and set-level rest. |
| v0.13 | Timed, cardio (duration or speed), bodyweight with optional added load, per-side reps; optional RPE/RIR with units kept on history. |
| v0.14 | Progression policies (linear, Greyskull LP, double, triple, time-based); stall detection; planned deloads; explanations for targets. |
| v0.15 | Plate calculator (bar type, bar weight, owned plates); workout-day reminders and missed-workout nudges that respect quiet hours and permissions. |
| v0.16 | Watch execution for all Release 2 set and exercise types; Release 2 acceptance (equivalent targets on web and Apple; no duplicate Watch sessions). Depends on v0.08. |

### Release 3: Progress analytics and body tracking

| Version | Scope |
|---|---|
| v0.17 | Estimated 1RM trends with selectable formulas, PR timeline, activity heatmap, exercise volume trends. |
| v0.18 | Muscle-volume, recovery/fatigue, and detraining maps, with calculation explanations and accuracy limits; structural-balance views. |
| v0.19 | Bodyweight logging, goal lines, measurement history, progress photos (in file or object storage, not localStorage); media-inclusive backup format. |
| v0.20 | Editing historical workouts, adding past sessions, correcting dates; recalculation of all derived records. |
| v0.21 | Opt-in Apple Health integration for supported workouts and bodyweight. Writes are deduplicated; denied permission does not interrupt tracking. Raw heart-rate, Health data, and photos stay out of AI access. |

### Release 4: Imports, exercise customization, sharing

| Version | Scope |
|---|---|
| v0.22 | Imports from FitNotes, Strong, Hevy CSV, and Apple Health weight exports. Preview, exercise matching, unresolved-item review, duplicate detection, downloadable error report. |
| v0.23 | openGym backup migration adapter for the pinned upstream data version. Hevy API import as a separate, optional connection. |
| v0.24 | Custom exercises, equipment filters, notes, optional media (location metadata stripped; type and size limits), reference links. |
| v0.25 | Routine sharing as a portable file and printable PDF. Complete backup and restore including media. Automatic backup options. Public-build check extended to release artifacts. |

### Release 5: Themes, accessibility, Apple polish

| Version | Scope |
|---|---|
| v0.26 | Curated accent palettes; light, dark, and system appearance; shared semantic tokens across web, iPhone, and Watch; previews; appearance sync for cloud users. |
| v0.27 | Accessibility expansion: VoiceOver, Dynamic Type, keyboard navigation, reduced motion, contrast, non-color status indicators. |
| v0.28 | iPhone widgets, Live Activities, Watch complications, configurable workout-screen layouts. |
| v0.29 | Gym membership QR and barcode storage and display. Kept local, excluded from AI access. |
| v0.30 | Localization infrastructure, including right-to-left layout support. |
| v0.31 | Translation of app and exercise text to the pinned upstream language set. |

### Release 6: Coaching extensions and account maturity

| Version | Scope |
|---|---|
| v0.32 | Proposal history (provenance, changes, approval or rejection, resulting program). Stale-version proposals require renewed review, never overwrite newer edits. |
| v0.33 | Richer coaching tools: progression explanations, training summaries, program-revision proposals. |
| v0.34 | Revocable connections with scoped permissions, user-visible tool activity history, assistant-specific setup and reusable coaching guidance. |
| v0.35 | Embedded coaching feasibility study only: supported inference mechanism and funding model. No build unless the study passes and is approved. |
| v0.36 | Device and session management, account recovery, disconnect-to-local behavior, extended account export and deletion. |
| v0.37 | Docker-free self-hosting documentation, migrations, backup and restore procedures, update guidance. |
| v0.38 | Invite-only registration and administrator controls for self-hosted deployments. Administrative logs avoid health data. |

## Open decisions and where they get resolved

| Decision | Resolve at |
|---|---|
| Cloud provider and deployment | G2, before v0.03 |
| Conflict-resolution policy | Start of v0.06 |
| Core statistics definitions | Start of v0.03 |
| Upstream catalogue subset | G1 (v0.01 uses in-house list) |
| Starter routines | v0.01 |
| Minimum iOS and watchOS versions | v0.02 and v0.07 |
| Health scope | v0.07 (workout and heart rate); broader in v0.21 |
| Branding, domain, repository | G3 |
| Interchange contract detail | v0.01 (v1); extended per version |
| Migration and proposal UX | v0.05 (migration); v0.10 (proposals) |
| Gemini eligibility | Recheck at start of v0.09 |

## Sources

- [initial-release.md](initial-release.md): initial-release task list and acceptance criteria.
- [future-releases.md](future-releases.md): Release 2 to 6 scope and gates.
