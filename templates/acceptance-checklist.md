# Tour review checklist

## Isolation and side effects

- [ ] Demo context is synthetic/disposable and has a distinct identifier.
- [ ] Reset, writes, reads, and cleanup are scoped to that context at the data/domain boundary.
- [ ] User's pre-tour context is saved and restored on finish, skip, failure, and unmount.
- [ ] A hard reload, crash or expired session returns the user to their own context.
- [ ] Other tabs never show demo data without the demo banner.
- [ ] External effects (payments, messages, publishing, webhooks, real imports) are disabled/stubbed.
- [ ] Tests compare the real user's before/after data snapshot and find no change.
- [ ] Undo targets the exact tour-created change id.
- [ ] Isolation mode is declared. A shared demo estate discloses its leak and ships a guarded purge.

## User experience

- [ ] Entry point is discoverable in supported auth states.
- [ ] Demo banner and exit control remain visible while tour runs.
- [ ] Back, Next, Skip/Exit, Replay, and progress work.
- [ ] Missing/offscreen targets clear highlights instead of leaving stale overlays.
- [ ] Failed actions show a plain, truthful message with retry/skip/exit.
- [ ] Keyboard, focus, reduced motion, contrast, and small-screen behavior are checked.

## Step resilience

- [ ] Every domain refusal rule for a tour write is listed in the spec.
- [ ] Every write step passes twice in a row and once against a conflicting open item.
- [ ] Steps that need an earlier result explain what is missing and point back, instead of hiding.
- [ ] Each targeted step has an always-visible fallback target, used only after a grace period.
- [ ] After a step creates or regenerates an object, the tour opens it with filters aligned to the demo scope.

## Drive cursor

- [ ] Every visible-UI action runs from a drive script (data), through one product-neutral engine.
- [ ] The cursor appears and moves onto each control before the engine types or presses.
- [ ] Typed values fire the product's own change handlers, and the form submits what the cursor typed.
- [ ] Anchors that render twice resolve to the copy on top. Row actions are scoped by entity id.
- [ ] Spotlight, mask and cursor ignore pointer events. The cursor is hidden from assistive tech.
- [ ] While an action runs, the spotlight frames the control the cursor works on.
- [ ] The changed row or record is ringed after the write lands.
- [ ] Reduced motion keeps every step and drops every wait.
- [ ] A missing control raises the typed miss. The fallback closes half-filled forms, then writes directly.
- [ ] The live check fails a step when no cursor appears, and it records `screen` or `direct` per step.
- [ ] A completed write step cannot run twice. No cursor or ring remains after exit.

## Verification and handoff

- [ ] Scenario fixture requirements and semantic anchors validate before use.
- [ ] One complete real UI journey passes in a fresh disposable context.
- [ ] Replay returns to baseline; exit restores the previous context.
- [ ] Timeout and missing-target paths are tested.
- [ ] Relevant unit/integration/full host-repo gates run and results are recorded.
- [ ] Handoff names tested commands, environment, gaps, and how to start/replay/exit.
- [ ] No credentials, personal data, or production fixtures appear in public files/evidence.
