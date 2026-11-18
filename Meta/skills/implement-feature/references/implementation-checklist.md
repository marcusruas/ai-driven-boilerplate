# Implementation checklist

Work through this while implementing (Step 8). Skip a row only when it really doesn't apply, and say
so in the report. **Source of truth:** spec > Trello card > existing code patterns. If the spec and
existing behavior conflict and the spec doesn't resolve it, stop and ask the user.

> Add your stack's rules (performance, framework conventions) when you set up a new project.

## Code

- [ ] Files live where the spec's file checklist says; naming and layout match the existing code.
- [ ] Configuration values are exposed where the spec says, with the spec's defaults.
- [ ] Stack hygiene from `Meta/constraints.md` §3 is respected (generated files untouched, files
      that travel together moved together).
- [ ] Mock values only in config/secret files.

## Verification

- [ ] The build check from `Meta/constraints.md` §4 passes with no errors.
- [ ] Optional automated tests pass, if any were written.
- [ ] Every acceptance criterion maps to a manual test step.

## Cross-domain sanity (before hand-off)

- [ ] Nothing in other domains breaks: shared code, data, configuration, interfaces.
- [ ] Scope matches the card's **Out of scope**; nothing extra snuck in.
- [ ] `git status` contains only intended changes.
