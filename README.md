# Agent Product Tour Blueprint

Give a coding agent this blueprint to add a guided product tour to an unfamiliar application, regardless of its language or framework. It makes the agent inspect the host system first, isolate demo state, implement one real workflow, and prove that the tour leaves user data untouched.

## Use it

1. Copy `AGENTS.md` and `templates/` into the target repository (or point your coding agent at this repository).
2. Ask the agent: **“Implement a guided tour using the instructions in `AGENTS.md`. Start by mapping the app and completing `templates/TOUR-SPEC.md`; do not code until you prove the demo-data isolation boundary.”**
3. Review the completed tour spec and safety boundary before approving write actions or live tests.
4. Run the host project's tests and follow the agent's handoff report.

No runtime or framework dependency is required. The blueprint instructs the agent to use the target project's existing UI, data layer, browser automation, and test stack.

## What it enforces

- Dedicated demo context; no resets or writes to real user profiles.
- External side effects disabled during the tour.
- Scenario-driven steps with explicit postconditions and bounded waits.
- Real UI interaction where practical, with a verified safe fallback.
- Undo scoped to the tour's own change, replay from baseline, and safe exit.
- Unit/contract and live journey checks, including a before/after data-isolation assertion.

## Files

- `AGENTS.md` — ordered implementation workflow, safety rules, architecture guidance, and acceptance gates for coding agents.
- `templates/TOUR-SPEC.md` — host-specific discovery and design contract for the agent to complete before coding.
- `templates/scenario.example.yaml` — stack-neutral example of scenario data and observable success checks.
- `templates/acceptance-checklist.md` — handoff checklist for review and live verification.

## Scope and origin

This blueprint abstracts design lessons from Abacus's guided tour: a disposable persona sandbox, scenario data, real UI actions, explicit tour state, exact-change undo, replay, and live tests that verify the user's own data is unchanged. It contains no Abacus source code, financial fixtures, credentials, or product-specific selectors. It is a starting contract, not a drop-in tour library; the receiving agent must adapt it to the target system.

## License

MIT. See [`LICENSE`](LICENSE).
