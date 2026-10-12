# Changelog

Each release gets a git tag of the form `vMAJOR.MINOR.PATCH`. Record the tag you copied in `TOUR-SPEC.md` as `blueprint_version`.

- **Major:** a rule now forbids something an earlier version allowed, or a template field changes meaning.
- **Minor:** a new rule, section, template field or check.
- **Patch:** wording, examples and fixes that change no rule.

## v1.0.0 (2026-10-12)

First tagged release. It contains the work below.

### Base blueprint (2026-10-02)

- Ordered workflow: map the host, prove isolation, build one journey, then add more.
- Safety boundary, explicit lifecycle, exact-change undo, replay and exit on every path.
- Templates: tour spec, scenario example and acceptance checklist.

### Drive cursor ([#2](https://github.com/Synx-x/agent-tour-blueprint/issues/2), [#3](https://github.com/Synx-x/agent-tour-blueprint/pull/3))

- A visible cursor drives each "Show me" step from drive scripts, through one product-neutral engine.
- Visible-copy hit-testing, row scoping, pointer-free overlays and a typed miss that triggers the direct fallback.
- A live check fails a step when no cursor appears, and records the `screen` or `direct` path.

### Shared demo estate and step resilience ([#1](https://github.com/Synx-x/agent-tour-blueprint/issues/1), [#4](https://github.com/Synx-x/agent-tour-blueprint/pull/4))

- Isolation modes: private sandbox, or a shared demo estate with a guarded purge and leak disclosure.
- Self-healing write steps, domain refusal rules, fallback spotlight targets and dirty-state tests.

### Hardening ([#5](https://github.com/Synx-x/agent-tour-blueprint/issues/5))

- Reload, crash and session expiry return the user to their own context.
- Other tabs never show demo data without the demo banner.
- An in-product assistant that shares the engine runs registered scripts only.
- Tour contract checks run in CI on every host change.
- Localization rules for anchors, tour text and typed values.
- `templates/scenario.schema.json`, a JSON Schema for scenario files.
- Tour telemetry events with no personal data, and the `direct` path rate as a drift signal.
- This changelog, tagged releases and the `blueprint_version` spec field.
