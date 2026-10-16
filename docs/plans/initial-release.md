# Initial Release Implementation Plan

Date: 2026-10-14
Status: agreed planning baseline; unresolved implementation choices are listed below.

## Summary

Build a public, open-source openGym fork/rewrite with native SwiftUI iPhone and Apple Watch apps and a full React web companion. Preserve the existing visual direction and workout flow with platform-appropriate controls. No Docker dependency.

## Agreed architecture

- Apple apps: SwiftUI and SwiftData for durable local storage. Mac, iPhone, and Watch are available for testing.
- Web: current stable React + Vite, TypeScript, Tailwind, shadcn, and TanStack Router only. No TanStack Query or Start.
- No account required: on-device Apple storage and browser localStorage.
- Optional cloud account supports full web/Apple synchronization, not just a download demo.
- Account sign-in: Apple plus passwordless email.
- Standard cloud security with encrypted transport/storage and account isolation. Backend access to authorized training data is allowed; end-to-end encryption is not required.
- Initial hosting budget: under $25/month, excluding inference because users use external assistants.
- Color customization is deferred to future releases.
- Public source and builds exclude restricted upstream exercise media. Any permitted personal-use media remains segregated and untracked.

## Implementation tasks

### 1. Inventory and licensing

Pin an upstream commit; inventory core UI flows, data formats, reusable pure functions, tests, and exercise catalogue rights. Preserve applicable AGPL/third-party notices and verify App Store distribution permissions for reused code. Add public-build checks against restricted media inclusion.

### 2. Cloud provider selection

Compare Docker-free managed and self-operated options before selecting one. Evaluate costs, Swift/browser support, Apple/email auth, recovery, account deletion, offline sync, backups, migrations, OAuth for remote MCP, assistant client registration, account isolation, and maintenance. Record provider, hosting model, quotas, cost estimate, and recovery procedures before provider-specific implementation.

### 3. Domain and local persistence

Define versioned interchange contracts for exercises, routines, sessions, sets, settings, and AI proposals. Use stable identifiers, revisions, and deletion markers for reconciliation. Implement Swift and TypeScript calculation equivalents with shared fixtures. Separate persistence/network adapters from training behavior. Add validated backup/import and visible browser-storage failure handling.

### 4. Core iPhone and web experience

Implement routine creation/editing, exercise search, workout logging, rest timers, history, basic progress statistics, and core-data backup/export. Preserve upstream visual style. Start web users account-free; offer full synced use after sign-in. Preview and confirm local-to-account data migration rather than silently replacing data. Keep logging usable offline and through expired authentication or server failures.

### 5. Phone-free Watch workouts

Transfer routines and necessary context in advance. Start stored routines, edit reps/weight, complete sets, run rest timers, and finish without the phone nearby. Persist unfinished/completed sessions and sync later. Allow phone and Watch to edit the same live session; reconcile offline changes without lost sets or duplicate workouts. Use stable session/set identifiers and retry-safe operations. Verify workout-session and heart-rate behavior on real hardware with explicit Health permission; broader Apple Health integration is future work.

### 6. Optional cloud sync

Synchronize core records across web/iPhone/Watch with offline queues and visible connection status. Implement isolation, retry safety, conflict handling, and deletion propagation. Keep a usable local copy on disconnect and surface unsent changes. Keep credentials out of workout data and backups. Include essential public-launch account export/deletion and assistant revocation; extend administration later.

### 7. External coaching

No embedded chatbot or app-funded inference at launch. Users chat in their existing assistant accounts.

- Deliver tested live ChatGPT, Claude, and eligible Gemini connections through authenticated remote MCP plus assistant-specific packaging and instructions.
- Expose scoped training reads and draft-program creation/revision, not direct program application or destructive history tools.
- Add an in-app proposal inbox with validation, change preview, stale-base detection, and explicit approval before active-program changes.
- Make shared data and connection revocation understandable.
- Local-only users export selected context, then import/review a structured proposal; no automatic upload.
- Test actual supported assistant accounts. Publication, client behavior, and eligibility are release gates, not assumed universal availability.
- Recheck Gemini eligibility at implementation: documented planning-time constraints include US adults, personal Google accounts, English, and Keep Activity enabled.
- Embedded coaching is future work requiring a supported inference/funding decision; do not assume consumer subscriptions cover embedded API calls.

## Verification and acceptance

- Complete offline/account-free workouts and recover saved records after restart.
- Finish phone-free Watch workouts; reconnect and obtain one intact session.
- Reconcile simultaneous phone/Watch edits and retry delivery without lost sets.
- Bring local data into accounts without silent replacement; survive expired auth/cloud downtime.
- Round-trip backups and reject malformed inputs without damaging records.
- Verify Swift/TypeScript calculation parity, account isolation, and revoked-access denial.
- Test all three live assistants: read authorized history, submit drafts, and confirm no routine changes before in-app approval.
- Verify restricted media is absent from public source/artifacts.
- Run web/native/accessibility checks and physical-device testing; use TestFlight before public distribution.

## Open decisions

This is not yet a decision-complete specification. Resolve before implementation:
- Cloud provider, deployment, and exact conflict-resolution policy.
- Precise core statistics, catalogue subset, launch set types, and starter routines.
- Minimum iOS/watchOS versions and exact Health scope.
- Branding, domain, repository/fork destination, and distribution details.
- Detailed interchange/API contracts and migration/proposal UX.

Future features: [future-releases.md](future-releases.md).
Version split (v0.01 onward): [versions.md](versions.md).

## References

- https://github.com/DuarteSantos8/openGym
- https://github.com/DuarteSantos8/openGym/blob/main/NOTICE.md
- https://github.com/DuarteSantos8/openGym/blob/main/docs/MOBILE.md
- https://github.com/DuarteSantos8/openGym/blob/main/mcp/README.md
- https://www.hevyapp.com/features/hevy-chatgpt/
- https://developers.openai.com/plugins/build/mcp-server
- https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp
- https://support.google.com/gemini/answer/17209137?co=GENIE.Platform%3DDesktop&hl=en
