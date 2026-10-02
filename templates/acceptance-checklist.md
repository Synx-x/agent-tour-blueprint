# Tour review checklist

## Isolation and side effects

- [ ] Demo context is synthetic/disposable and has a distinct identifier.
- [ ] Reset, writes, reads, and cleanup are scoped to that context at the data/domain boundary.
- [ ] User's pre-tour context is saved and restored on finish, skip, failure, and unmount.
- [ ] External effects (payments, messages, publishing, webhooks, real imports) are disabled/stubbed.
- [ ] Tests compare the real user's before/after data snapshot and find no change.
- [ ] Undo targets the exact tour-created change id.

## User experience

- [ ] Entry point is discoverable in supported auth states.
- [ ] Demo banner and exit control remain visible while tour runs.
- [ ] Back, Next, Skip/Exit, Replay, and progress work.
- [ ] Missing/offscreen targets clear highlights instead of leaving stale overlays.
- [ ] Failed actions show a plain, truthful message with retry/skip/exit.
- [ ] Keyboard, focus, reduced motion, contrast, and small-screen behavior are checked.

## Verification and handoff

- [ ] Scenario fixture requirements and semantic anchors validate before use.
- [ ] One complete real UI journey passes in a fresh disposable context.
- [ ] Replay returns to baseline; exit restores the previous context.
- [ ] Timeout and missing-target paths are tested.
- [ ] Relevant unit/integration/full host-repo gates run and results are recorded.
- [ ] Handoff names tested commands, environment, gaps, and how to start/replay/exit.
- [ ] No credentials, personal data, or production fixtures appear in public files/evidence.
