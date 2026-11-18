# Code exploration checklist (before writing the spec)

Read the live code, not just the knowledge base: domain files may be stubs. For every area the card
touches, find the current implementation and note the exact file (and line, for code).

> Add your stack's key locations to "Where to look" when you set up a new project.

## Always read first

- `Meta/constraints.md`
- Every `Meta/domains/*.md`
- The dependency manifest (which libraries/packages are available). Don't plan on a dependency that
  isn't installed without asking.

## Where to look

- **Source code** under `{{CODE_DIR}}/`: the modules, classes and functions related to the card.
- **Configuration:** settings files, environment variables, feature flags.
- **Data & assets:** data files, content, media the card touches.
- **Entry points:** how the area is reached (routes, scenes, commands, input bindings).
- **Existing tests** for the area, if any.

## Questions the spec must answer

1. **What exists today?** What already does part of this, and what would block the new behavior?
2. **Where does the new logic live?** New module vs extending an existing one.
3. **Where do configuration values live?**
4. **What needs the user?** Manual steps in an editor, console or account.
5. **What does it interact with?** Other domains' code, shared state, events, interfaces.

## Common blockers to call out

- Validation or required fields that block the new path.
- Logic that assumes a single user/instance.
- References that are only set up by hand (break silently when something moves).
- A dependency the feature needs that isn't installed.
