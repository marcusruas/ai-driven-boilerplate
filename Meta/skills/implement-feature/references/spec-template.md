# Spec template

Write `Meta/specs/<slug>.md` with this outline. Scale each section to the task; if one genuinely
doesn't apply, write "None" and why.

```markdown
# {{PROJECT_NAME}} — <Feature title> — Spec

> **Status: In development.** Trello card: [<card title>](<card url>) (YYYY-MM-DD).
> Branch: `feature/<slug>`.
>
> **Purpose:** <One short paragraph: what this adds or fixes, for the user.>

---

## 1. Summary

<What we're building. A before/after table helps when it changes existing behavior.>

## 2. Decisions

| # | Decision |
| --- | --- |
| 1 | <From the card's Decisions + conversation> |

## 3. Behavior

<How it works, step by step, from the trigger to the result. Include states and transitions (a
mermaid diagram helps for non-obvious flows), timings, and how it interacts with other domains.>

### 3.x <Sub-area per domain touched>

## 4. Configuration values

| Value | Default | Lives in |
| --- | --- | --- |
| <name> | <value> | <file / setting> |

## 5. Assets / content

| Asset | Type | Source | Status |
| --- | --- | --- | --- |
| <name> | <art / copy / data> | <where it comes from> | <Existing / To make / Placeholder> |

## 6. Edge cases

<Each edge case from the card and what happens.>

## 7. Out of scope

<From the card.>

## 8. File checklist

| # | File | Change |
| --- | --- | --- |
| 1 | `{{CODE_DIR}}/...` | <one-line change> |

Mark steps only the user can do (editor actions, account settings) as **Manual**.

## 9. Tests (optional)

<Automated tests worth writing, if any, and what they assert. "None" is fine when the manual test
covers it.>

## 10. Acceptance criteria

<1:1 from the Trello checklist.>

1. Given …, when …, then …

## 11. Manual test

1. <How to run it.>
2. <Steps that exercise each acceptance criterion.>

## 12. Revision log

| Date | Change |
| --- | --- |
| YYYY-MM-DD | Initial spec from refined card |
```

## Quality bar

The spec is ready when an implementer can:

- Find every file to touch from §8 without searching.
- Set every value from §4 without guessing.
- Check done-ness against §10 and run §11 without asking anything.
