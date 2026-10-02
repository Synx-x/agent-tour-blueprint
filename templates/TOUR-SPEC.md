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
isolation_boundary: TODO # exact namespace/identifier and where enforced
context_before_tour: TODO # what is saved/restored
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

## Scenario inventory

| Scenario ID | Audience | Fixture requirements | Safe writes? | Undo/recovery | Pass condition |
|---|---|---|---|---|---|
| TODO | TODO | TODO | TODO | TODO | TODO |

## Step design

For each journey, list the steps with stable ids, narration, destination, semantic target, optional action, observable postcondition, timeout, and retry/skip/exit guidance. Keep fixture-specific text out of UI code.

| Step ID | Narration goal | Destination/target | Action | Postcondition | Timeout/recovery |
|---|---|---|---|---|---|
| TODO | TODO | TODO | TODO | TODO | TODO |

## Safety proof

- How is a tour context distinguished from a real user context? TODO
- Which layer rejects reset/write attempts against real contexts? TODO
- How are real network side effects prevented in development and tests? TODO
- What before/after snapshot proves the user's state is unchanged? TODO
- How is exact-change undo scoped? TODO / N/A with reason
- What cleanup runs on exit, unmount, and async failure? TODO

## Acceptance commands and evidence

| Command/journey | Environment | Expected checks | Result/evidence |
|---|---|---|---|
| TODO | TODO | TODO | TODO |

## Approval gate

Do not add write actions until the safety proof above is complete and the disposable boundary is verified. If any answer is unknown, keep the tour read-only and report the blocker.
