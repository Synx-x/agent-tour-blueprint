# Instructions for the coding agent

You are implementing a guided tour in the host repository. This brief is framework-neutral. Use the repository's existing language, architecture, UI conventions, data boundaries, and test tools. Do not introduce a new stack just to match an example in this file.

## Goal

Teach useful product workflows with realistic demo data. Let users observe and, when safe, perform real product actions. The tour must be repeatable, recoverable, and isolated from real user data and external side effects.

Keep the tour thin: reuse real product screens and domain services. Store scenario-specific values as data. Put navigation, action execution, waits, and cleanup behind a small runner/adapter that can be tested independently of the presentation.

## Mandatory order of work

### 1. Inspect the host app

Before coding:

- Read the host's agent instructions, README, architecture notes for the relevant UI/data areas, and test/run scripts.
- Locate the app shell, navigation, representative user workflows, persistence boundaries, profile/tenant switching, auth states, external integrations, and existing audit/undo behavior.
- Find existing onboarding/tour code, stable semantic UI selectors, accessibility conventions, and browser/e2e tooling.
- Verify commands from repository scripts or docs; do not guess the framework or package manager.
- Complete `templates/TOUR-SPEC.md` in the host repo (copy it there first if needed).
- Report the proposed isolation boundary and external-effect controls before adding any write action.

Stop before implementation if you cannot prove that tour reads/writes use a disposable context or that external side effects are blocked. Ask for the missing boundary or implement read-only steps only.

### 2. Establish the safety boundary

Use a dedicated synthetic persona, test tenant, temporary database, browser profile, or disposable copy. Give it an identifier that cannot collide with ordinary users. Verify the identifier at the service/domain boundary, not only in UI state.

Never reset or seed a real user profile. If active-profile state syncs across devices, do not store the tour context there unless the host provides a scoped, reversible session mechanism. Save the previous context and restore it on every exit path.

Disable or stub real payments, email/SMS, publishing, webhooks, imports, and other irreversible network operations. Use a test gateway or dry-run mode. Use synthetic/minimized fixtures; never copy production personal data, tokens, secrets, or credentials into the tour or its tests.

### 3. Design one end-to-end journey

Start with one representative, useful journey. Add more scenarios only after the first passes end to end. A scenario should be data, not branching JSX/template logic. Use `templates/scenario.example.yaml` as a shape guide, then use the host language's normal format.

Each step needs a stable id, short title, accurate narration, optional destination and semantic target, optional named action, observable success condition, and recovery guidance. Validate all fixture requirements, targets, destinations, action names, and success checks before presenting the tour.

Prefer deterministic fixture-backed examples. If a step depends on AI, network access, or a quota, make that step optional or provide a clearly labeled non-AI fallback. Never fabricate a successful result.

### 4. Implement explicit lifecycle and state

Adapt this lifecycle to the host framework:

```text
closed -> choosing -> preparing -> running -> finishing -> closed
                                  |       |
                                  |       +-> action-running -> running
                                  |                        \-> step-error -> retry/skip/exit
                                  +-> preparation-error -> retry/exit
```

Keep state minimal: active scenario, step index, disposable context id, busy flag, result/error, and any exact change id required for undo. Clamp Back/Next to the valid range. Disable conflicting controls while an action runs. Make start and cleanup idempotent. Ignore or cancel late async results after exit.

Every wait/action must have a finite timeout. Prefer events, pending flags, or visible/state postconditions over long fixed sleeps. A failed postcondition must leave the user on the step with a plain-language explanation and retry/skip/exit options.

### 5. Build the tour UI

- Add a discoverable entry point; add a persona/scenario picker only when workflows or audiences differ.
- Keep a visible demo/sandbox banner and exit control while the tour is active.
- Provide Back, Next, Skip/Exit, Replay/Restart, progress, and visible action results.
- Navigate to real screens. Use stable semantic anchors (accessible names or explicit test/tour attributes), not generated CSS classes or positional selectors.
- Recompute spotlight targets after navigation/layout changes. Clear the previous spotlight when a target is missing, hidden, occluded, or offscreen. Never leave a stale ring.
- Respect keyboard navigation, focus management, Escape, contrast, reduced-motion preference, and touch targets. Return focus sensibly on close.
- Keep narration concise and grounded in the selected fixture. Describe empty, zero, stale, or unavailable values accurately.

### 6. Run actions through product behavior

Prefer operating the same visible controls a person uses: open the form, fill/select values, submit, then verify the result. This tests both the tour and the product UI.

Use a direct domain/API adapter only when visible operation is unreliable or impossible (for example, a control unavailable at a narrow viewport). Keep the fallback explicit, use normal authorization and validation, and check the same postcondition. Never create a tour-only bypass around business rules.

For each write, read the resulting state back from the disposable context. If the host has an audit/change-set mechanism, mark the actor as `tour`/`demo` and retain the exact identifier. A later undo step must target that identifier only. Never undo “the latest” change or an unrelated row.

### 7. Restore and clean up on every exit

On completion, skip, close, failure, route change, and unmount:

- stop pending tour work or make late results harmless;
- remove overlays, spotlight, cursor guidance, and tour-only keyboard handlers;
- discard/retain only the disposable fixture according to the completed spec;
- restore the user's prior profile, route, and relevant UI context;
- verify that user data and external systems remain unchanged.

Test exit during preparation and while an action is pending. If cleanup or undo fails, state that plainly and give a safe next action. Never silently strand the app in the demo context.

## Acceptance gates

Use the host project's existing test stack. Unit tests alone do not prove the live tour works.

### Unit and contract checks

- Lifecycle transitions, busy behavior, step bounds, and error recovery.
- Every scenario reference resolves against its fixture and allowed action registry.
- Every required destination and target exists or is explicitly optional.
- Actions satisfy observable postconditions against the disposable context.
- Tour change is undone by its exact id and data returns to baseline.
- Reset/cleanup rejects ordinary user/account identifiers.
- External-effect adapters are stubbed/disabled in tests.

### Integration and live journey

Run each scenario with a fresh disposable user/profile and verify:

1. Entry/picker behavior in supported guest and signed-in states.
2. Sandbox banner is visible and identifies demo mode.
3. Every step is reachable; navigation and spotlight stay correct.
4. “Show me” uses the real UI when feasible.
5. Success is confirmed from actual product state, not only a toast or narration.
6. Undo reverses only the tour's own change.
7. Replay returns to the declared baseline.
8. Skip/exit restores the previous context and its before/after data snapshot matches.
9. No real external integration receives a request.
10. A missing target and a timed-out action fail visibly and recoverably.

Run focused tests first, then all required host-repo gates. Run at least one real-browser journey if the product has a browser UI. Check the smallest supported viewport and reduced-motion mode where relevant. Save concise evidence (commands, environment, scenarios, pass/fail counts, and limits) without credentials or personal data.

## Handoff report

Report where the tour starts, the isolation boundary and external-effect controls, scenarios covered, which actions used the visible UI versus an adapter, exact tests/results, known gaps, and how to start/replay/exit. Do not claim “fully verified” unless live journeys, data isolation, undo, replay, and exit were exercised.

## First instruction

Do not code immediately. Map the host system, complete `TOUR-SPEC.md`, and prove isolation first. Implement and test one end-to-end scenario before adding more.
