# Refined card template

Write the refined card description with this structure (Markdown). Scale each section to the task.
If a section genuinely doesn't apply, write "None" rather than leaving it blank. Acceptance criteria
do **not** go here; they go in the **"Acceptance Criteria"** checklist (SKILL.md Step 5).

```markdown
## Problem
<One short paragraph: what we're adding or fixing, and why it matters to the user.>

## User experience
<What the user does and perceives: trigger, result, feedback.>

## Decisions (from refinement)
<Bullets: every concrete answer reached during refinement, especially the "how does it behave
when X" clarifications. One line of why per decision.>
- `<decision>`: chose `<option>` over `<alternative>` because `<reason>`.

## Domains affected
<Which domains this touches and what changes in each. Name new domains if this introduces one.>

## Data & configuration
<Starting values, limits, and where they should live (config, constants, data files).>

## Assets / content needed
<Art, copy, media, data. Mark each as existing or to be made.>

## Edge cases
<Interruptions, failures, concurrency, invalid input.>

## Out of scope
<What this card explicitly does not include.>

## Constraints
<Only those from Meta/constraints.md that matter here.>

## Status
Refined: ready for implementation.
```

## Acceptance criteria (checklist, not description)

One testable statement per item, in Given / When / Then form:

- `Given <context>, when <action>, then <observable outcome>.`
