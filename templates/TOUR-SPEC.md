# Tour specification

The receiving agent completes this document before coding. Replace every `TODO`; mark non-applicable fields `N/A` with a reason.

## Product map

- Product and user goal: TODO
- Main app entry/shell: TODO
- Supported roles/auth states: TODO
- Relevant screens/routes/panels: TODO
- Existing onboarding/tour or demo mode: TODO
- Data stores and ownership boundary: TODO
- Existing audit/undo mechanism: TODO
- External integrations that can cause side effects: TODO
- Existing test, browser automation, and run commands: TODO
- Relevant accessibility/responsive conventions: TODO

## Tour contract

```yaml
entry_points:
  - TODO
audience:
  - TODO
demo_data_source: TODO # synthetic fixture / disposable tenant / temporary copy
isolation_mode: TODO # private-sandbox (preferred) / shared-demo-estate, with reason
shared_estate: TODO / N/A # who else sees demo data, totals it changes, purge control and its guard
isolation_boundary: TODO # exact namespace/identifier and where enforced
context_before_tour: TODO # what is saved/restored
context_storage: TODO # page memory (preferred) / persisted with a recovery marker; what boot does with a stale marker
cross_tab: TODO # tab-scoped, or how other tabs learn about the demo context and show the banner
reset_policy: TODO # disposable scope only; specify replay baseline
external_effects_blocked:
  - TODO
journeys:
  - TODO
failure_paths:
  - TODO
validation_commands:
  - TODO # commands verified from host docs/scripts
known_limits:
  - TODO
```

## Domain refusal rules

List every service guard that can refuse a tour write. Each one needs a self-healing step and a dirty-state test.

| Write | Refusal rule (from service code) | How the step clears it inside the demo scope | Dirty-state test |
|---|---|---|---|
| TODO | TODO | TODO | TODO |

## Scenario inventory

| Scenario ID | Audience | Fixture requirements | Safe writes? | Undo/recovery | Pass condition |
|---|---|---|---|---|---|
| TODO | TODO | TODO | TODO | TODO | TODO |

## Step design

For each journey, list the steps with stable ids, narration, destination, semantic target, fallback target, optional action, observable postcondition, timeout, and retry/skip/exit guidance. Keep fixture-specific text out of UI code.

| Step ID | Narration goal | Destination/target | Action | Postcondition | Timeout/recovery |
|---|---|---|---|---|---|
| TODO | TODO | TODO | TODO | TODO | TODO |

## Drive and cursor design

Answer these before you build any visible-UI action. See `AGENTS.md` section 6.

```yaml
anchor_attribute: TODO # e.g. data-tour; one attribute for every driven control
row_attribute: TODO # e.g. data-row holding the entity id; how row actions are scoped
duplicate_renders: TODO # screens where one anchor matches twice; how the visible copy is picked
stage_hook: TODO # how the host brings a screen/panel forward for the engine
input_path: TODO # how a scripted value fires the framework's own change handlers
cursor_layer: TODO # where the cursor mounts; confirm pointer-events none and aria-hidden
pace_ms: { move: TODO, settle: TODO, per_char: TODO, max_type: TODO, press: TODO, after: TODO }
reduced_motion: TODO # every step kept, every wait zero
expected_path_by_viewport: TODO # per step: screen or direct at each supported viewport
shared_with_assistant: TODO / N/A # does an in-product agent reuse the engine and scripts?
assistant_registry: TODO / N/A # where scripts are registered by write name; which writes need user confirmation
```

| Step ID | Drive steps (do, target, row, caption) | Fallback when a control is missing | Runs once? |
|---|---|---|---|
| TODO | TODO | TODO | TODO |

## Safety proof

- How is a tour context distinguished from a real user context? TODO
- Which layer rejects reset/write attempts against real contexts? TODO
- How are real network side effects prevented in development and tests? TODO
- What before/after snapshot proves the user's state is unchanged? TODO
- How is exact-change undo scoped? TODO / N/A with reason
- What cleanup runs on exit, unmount, and async failure? TODO
- What happens on hard reload, crash and session expiry mid-tour? TODO
- What does a second tab show while the tour runs? TODO
- Which half-filled forms does the fallback close before a direct write? TODO

## Acceptance commands and evidence

| Command/journey | Environment | Expected checks | Result/evidence |
|---|---|---|---|
| TODO | TODO | TODO | TODO |

## Approval gate

Do not add write actions until the safety proof above is complete and the disposable boundary is verified. If any answer is unknown, keep the tour read-only and report the blocker.
