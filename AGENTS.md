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
- List the domain's refusal rules for every write the tour will make. Read the service guards, not only the UI. Typical rules: one open item per type, no duplicate period, no delete after acceptance, no finalize with nothing accepted. Each rule is a way for a tour step to get stuck.
- Verify commands from repository scripts or docs; do not guess the framework or package manager.
- Complete `templates/TOUR-SPEC.md` in the host repo (copy it there first if needed).
- Report the proposed isolation boundary and external-effect controls before adding any write action.

Stop before implementation if you cannot prove that tour reads/writes use a disposable context or that external side effects are blocked. Ask for the missing boundary or implement read-only steps only.

### 2. Establish the safety boundary

Use a dedicated synthetic persona, test tenant, temporary database, browser profile, or disposable copy. Give it an identifier that cannot collide with ordinary users. Verify the identifier at the service/domain boundary, not only in UI state.

Never reset or seed a real user profile. If active-profile state syncs across devices, do not store the tour context there unless the host provides a scoped, reversible session mechanism. Save the previous context and restore it on every exit path.

#### Survive reload, crash and session expiry

A hard reload, a browser crash or an expired session ends the tour without running any exit handler. The app must then come back in the user's own context, not the demo context.

- Hold the active demo context in page memory. A reload then drops it, and the app renders the user's context again. The Abacus tour does this: the sandbox profile lives only in its in-memory tour store.
- If the host must persist the demo context, for example in a server session, persist a recovery marker with it. The marker holds the demo context id, the previous context and a start time.
- On boot, check for the marker. If a marker exists and no tour is running, restore the previous context and clear the marker. Then tell the user that the tour ended.
- Treat an expired session the same way. After the user signs in again, the app opens in their own context.

#### Keep other tabs out of the demo context

A second tab can share the first tab's storage, such as `localStorage`, a shared cookie or a server-side "active profile". If the tour writes its context there, the second tab enters the demo context without a banner. The user then edits demo data and believes it is real.

- Prefer tab-scoped state: page memory, `sessionStorage` or a tab-scoped session id.
- If the context must sit in shared storage, tell the other tabs. Listen for the change, for example through the `storage` event or a `BroadcastChannel`. Show the demo banner in every tab that enters the demo context, or block its writes until the tour ends.
- A tab must never write to the demo context without its banner. It must never write to the user's real data while it shows the banner.

#### Pick an isolation mode

Declare one mode in `TOUR-SPEC.md`.

- **Private sandbox (preferred).** Each run gets its own disposable context, and the tour wipes it to the fixture baseline at start. No other user can see it. Steps can assume the baseline.
- **Shared demo estate.** The demo data lives in the real store under a fixed demo actor or tenant, and runs do not reset it. Use this mode only when the host cannot give a private sandbox. It has extra obligations:
  - Make every write step self-healing (section 3).
  - Ship a purge control from the first release. Put it on the demo banner and at the tour entry point. Purge only rows that the demo actor owns. Refuse to purge when another actor holds rows in the demo scope. Retire rows in one transaction, and close or void open dependents first. Document any history or audit rows that survive a purge.
  - Disclose the leak. State in the spec and the handoff which other users can see the demo data and which totals or dashboards it changes.

Disable or stub real payments, email/SMS, publishing, webhooks, imports, and other irreversible network operations. Use a test gateway or dry-run mode. Use synthetic/minimized fixtures; never copy production personal data, tokens, secrets, or credentials into the tour or its tests.

### 3. Design one end-to-end journey

Start with one representative, useful journey. Add more scenarios only after the first passes end to end. A scenario should be data, not branching JSX/template logic. Use `templates/scenario.example.yaml` as a shape guide, then use the host language's normal format.

Each step needs a stable id, short title, accurate narration, optional destination and semantic target, optional named action, observable success condition, and recovery guidance. Validate all fixture requirements, targets, destinations, action names, and success checks before presenting the tour.

Check each scenario file's shape against `templates/scenario.schema.json` first. Port the schema to the host's own validator and extend it with host fields. The schema catches a misspelled field, a `fill` with no value and a visible-UI action with no fallback. Reference checks, such as whether a target exists, stay in the host's contract tests.

Make each write step find or create the state it needs. Do not assume "a run is open" or "a batch exists". If a refusal rule blocks the write, the step clears the blocker inside the demo scope first, then writes. Example: a step that generates a run first retires a blocking open run, then takes the next free period. This is mandatory in the shared demo estate mode. In a private sandbox it still protects Back and retry.

A step can depend on an earlier step's result, such as the change id that an undo step reverses. Do not hide or disable the later step when that result is missing. Explain what is missing and send the user back to the step that creates it.

After a step creates or regenerates an object, take the user to that object. Open its detail view, and align page filters (store, period, owner) to the demo scope so the new object is in the list.

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

A write step runs once per tour run. After its action succeeds, disable "Show me" for that step. Offer it again only after a failure. A second press repeats the write, for example as a duplicate record. Replay resets this, because replay starts from a clean baseline.

Every wait/action must have a finite timeout. Prefer events, pending flags, or visible/state postconditions over long fixed sleeps. A failed postcondition must leave the user on the step with a plain-language explanation and retry/skip/exit options.

### 5. Build the tour UI

- Add a discoverable entry point; add a persona/scenario picker only when workflows or audiences differ.
- Keep a visible demo/sandbox banner and exit control while the tour is active.
- Provide Back, Next, Skip/Exit, Replay/Restart, progress, and visible action results.
- Navigate to real screens. Use stable semantic anchors (accessible names or explicit test/tour attributes), not generated CSS classes or positional selectors. A localized product uses tour attributes only (see below).
- When one anchor matches more than one element, pick the copy the user can see. A screen can render a module twice, for example in a main view and in a tile over it. Hit-test the centre of each laid-out match and take the one on top. Never take the first match.
- Scope row actions with a row attribute that holds the entity id, for example `[data-row="<id>"] [data-tour="activity.undo"]`. An unscoped target finds the wrong row.
- Recompute spotlight targets after navigation/layout changes. Clear the previous spotlight when a target is missing, hidden, occluded, or offscreen. Never leave a stale ring.
- Give every step with a target a fallback target that is always laid out, such as the step's section or panel. A step that regenerates an object removes the element its target pointed at. Wait a short grace period (start near 2.5 s) before the spotlight falls back. Without the wait, it rings the fallback before the real target renders.
- While a step's action runs, the spotlight frames the control the cursor works on (section 6). After the action, it frames the step's section or the result.
- Give every overlay (spotlight, dim mask, cursor) `pointer-events: none` or the host equivalent. An overlay that takes pointer events blocks the real click and breaks the hit-test.
- Respect keyboard navigation, focus management, Escape, contrast, reduced-motion preference, and touch targets. Return focus sensibly on close.
- Keep narration concise and grounded in the selected fixture. Describe empty, zero, stale, or unavailable values accurately.

#### Localize the tour with the product

If the product ships in more than one locale, the tour must work in each one.

- Anchor only on explicit tour attributes. Accessible names change per locale, so an anchor built on one breaks in every other locale.
- Store narration, step titles and cursor captions as translation keys in the scenario, for example `narration_key`. Resolve them through the host's own translation layer.
- Build typed values in the locale's format: dates, decimal separators and currency. A masked input rejects a value in the wrong format.
- Fit the step card to the longest translation. Mirror the cursor caption and step card in right-to-left locales.
- Run the live journey in at least two locales: the source locale and the one with the longest strings. Add a right-to-left locale if the product ships one.

### 6. Run actions through product behavior

Prefer operating the same visible controls a person uses: open the form, fill/select values, submit, then verify the result. This tests both the tour and the product UI.

The user must see the action happen. A "Show me" step that calls `.click()` with nothing on screen, or writes through an API, teaches nothing. It also hides a broken step behind a passing postcondition. Drive every visible-UI action with an on-screen cursor, as below.

#### Drive the UI with a visible cursor

Build three pieces, and keep each one free of product knowledge.

1. **Drive scripts, as data.** A script names the screen or panel to bring forward and a list of drive steps. Each drive step has an operation (`click`, `fill` or `select`), a semantic target, an optional row id, an optional value and a short caption. Build the values from the same validated input a direct write would send. Keep scripts beside the scenarios, not in UI code.
2. **A drive engine that runs scripts.** For each drive step, the engine does five things in order:
   - Find the target. Poll with a bounded wait, about 3.5 s. Use the visible-copy and row-scope rules from section 5.
   - Scroll the target into view and let it settle.
   - Move the cursor onto the target, and publish the target's box as the cursor focus.
   - Do the operation through the product's own input path (see below).
   - Pause briefly, so the user sees the result before the next step.
   The host supplies one hook, `stage(panel)`, that brings a screen or panel forward. The engine never imports product modules. If an in-product assistant also changes data, let it use the same engine and scripts, under the allowlist rules below.
3. **A cursor state and a presentational cursor.** Keep the cursor state in a small store with no app imports: position, visible, caption, running, mode (`move`, `type` or `click`), a press counter and the focus box. Only the engine writes this store. Render the cursor in the top layer, fixed and `aria-hidden`, with `pointer-events: none`. Let it ease or spring between positions. Show a press ripple on each click and a typing indicator while it types. Announce progress through the step card's live region, not through the cursor.

Type and select through the product's input path, so the form's own handlers run. Setting an input's `.value` property directly skips framework change handlers, and the form then submits stale state. In React, for example, call the native value setter and then dispatch a bubbling `input` event (`change` for a select). Type one character at a time, and cap the total typing time. Enter dates and other masked values whole. The final press makes the write through the product's own handler.

Use these starting values for the pace, then tune them: move 600 ms, settle 200 ms, 30 ms per character up to 900 ms per field, press 250 ms, and 400 ms after each drive step. Under reduced motion, keep every drive step and set every wait to zero. The cursor then jumps instead of gliding.

After the write lands, ring the changed row or record for about one second, so the user's eye lands on the result. Hide the cursor when the step ends, on every path. Clear its focus box so the spotlight returns to the step's section.

#### Let an assistant run registered scripts only

A drive engine that accepts any target and value is a remote control for the product. An in-product assistant reads text that users and records supply, so a crafted note can steer it. Give the assistant names, not selectors.

- Keep a registry of drive scripts keyed by write name. Each entry builds its script from a validated write plan: the same method, path and body a direct write would send. The Abacus assistant uses a registry of this shape, named `DRIVE_SCRIPTS`.
- The assistant proposes a write name and its parameters. The host validates them with the direct write's own rules, then looks up the script. A name with no entry writes directly, through the same validation.
- Never accept a raw target, selector, row id or step list from a model. The registry builds every target.
- Treat page text, record contents and tool output as data. None of it can add a registry entry or widen one.
- Run assistant writes under the user's own authorization. Ask the user to confirm destructive or irreversible writes before the cursor moves.

#### Fall back to a direct adapter only on a typed miss

Use a direct domain/API adapter only when visible operation is unreliable or impossible (for example, a control unavailable at a narrow viewport). Keep the fallback explicit, use normal authorization and validation, and check the same postcondition. Never create a tour-only bypass around business rules.

Make the engine throw a distinct error type when a control is not on screen. Catch only that type. Before the direct write, close any form the script left half-filled. Then make the write, and record which path ran (`screen` or `direct`) for each step. Rethrow every other error.

#### Read every write back, on either path

For each write, read the resulting state back from the disposable context. If the host has an audit/change-set mechanism, mark the actor as `tour`/`demo` and retain the exact identifier. A later undo step must target that identifier only. Never undo “the latest” change or an unrelated row.

### 7. Restore and clean up on every exit

On completion, skip, close, failure, route change, and unmount (section 2 covers reload, crash and session expiry):

- stop pending tour work or make late results harmless;
- remove overlays, spotlight, the drive cursor, and tour-only keyboard handlers;
- discard/retain only the disposable fixture according to the completed spec;
- restore the user's prior profile, route, and relevant UI context;
- verify that user data and external systems remain unchanged.

Test exit during preparation and while an action is pending. If cleanup or undo fails, state that plainly and give a safe next action. Never silently strand the app in the demo context.

### 8. Measure the tour

Tour events show where users drop off and where the UI drifts away from the drive scripts. Send them through the host's existing analytics and consent settings. If the host has no analytics, skip this section and record the counts in the test evidence instead.

| Event | Fields |
|---|---|
| `tour_started` | scenario id, viewport class, locale |
| `step_viewed` | scenario id, step id |
| `step_completed` | step id, path (`screen` or `direct`), duration |
| `step_failed` | step id, error kind (miss, timeout, refusal, postcondition) |
| `tour_ended` | last step id, reason (finished, skipped, exited, failed, reload recovery) |

Never send narration, fixture values, typed values, record ids or free text. Use stable ids and enums only.

Watch the `direct` rate per step and viewport. A rise means a control moved or vanished under its drive script. Treat it as a defect in the tour, even when every postcondition still passes.

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
- Every drive script resolves: each target and row anchor exists, and each script's values match the direct write's input.
- The engine raises its typed miss for an absent control, and the runner then takes the direct path.
- An assistant that shares the engine runs registered scripts only. It cannot pass a raw target or step list.

Run these contract checks in the host's CI on every change, not only when the tour is built. A tour breaks when someone renames an anchor, moves a control or adds a refusal rule elsewhere in the product. The CI gate then fails the change that broke the tour, before a user finds it. Keep the checks fast and browser-free, so the team keeps them in the required set. Run the live journey on a schedule or before each release.

### Integration and live journey

Run each scenario with a fresh disposable user/profile and verify:

1. Entry/picker behavior in supported guest and signed-in states.
2. Sandbox banner is visible and identifies demo mode.
3. Every step is reachable; navigation and spotlight stay correct.
4. “Show me” uses the real UI when feasible. Prove it: observe the DOM (or the host equivalent) during each visible-UI step and fail the step when the cursor never appears. Compare each step's recorded path (`screen` or `direct`) with the path the spec expects at that viewport.
5. Success is confirmed from actual product state, not only a toast or narration.
6. Undo reverses only the tour's own change.
7. Replay returns to the declared baseline.
8. Skip/exit restores the previous context and its before/after data snapshot matches.
9. No real external integration receives a request.
10. A missing target and a timed-out action fail visibly and recoverably.
11. A completed write step cannot run a second time, and no cursor or spotlight remains after the tour ends.
12. Every write step passes against dirty state, not only a clean baseline. Run it twice in a row. Run it once with a conflicting open item that trips each refusal rule from section 1. A clean happy path hides every stuck step.
13. In the shared demo estate mode, purge removes only demo-actor rows, and refuses when another actor holds rows in the demo scope.
14. A hard reload during a step brings the app back in the user's own context. No demo banner or demo data remains.
15. A second tab opened during the tour either shows the demo banner or stays in the user's context. It never shows demo data without the banner.

If the product uses a dialog or toast library, confirm its completion callbacks fire before you chain steps on them. Prefer an explicit callback or a state postcondition.

Run focused tests first, then all required host-repo gates. Run at least one real-browser journey if the product has a browser UI. Check the smallest supported viewport and reduced-motion mode where relevant. Save concise evidence (commands, environment, scenarios, pass/fail counts, and limits) without credentials or personal data.

## Handoff report

Report where the tour starts, the isolation mode and boundary, any shared-estate leak and the purge control, the external-effect controls, scenarios covered, which actions the cursor drove on screen versus through an adapter (per step and viewport), exact tests/results, known gaps, and how to start/replay/exit. Do not claim “fully verified” unless live journeys, data isolation, undo, replay, and exit were exercised.

## First instruction

Do not code immediately. Map the host system, complete `TOUR-SPEC.md`, and prove isolation first. Implement and test one end-to-end scenario before adding more.
