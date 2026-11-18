---
name: refine-task
description: Refine a task from the project's Trello board into a clear, unambiguous work item ready for implementation. Use when the user wants to refine, groom or clarify a Trello card that is in Backlog. Takes the card number or name, reads the knowledge base and the card, uses the Superpowers brainstorming skill to ask the questions that matter before development, writes the decisions back to the card, and moves it from Backlog to Refined.
disable-model-invocation: true
---

# Refine Task

Turn a rough Trello card in **Backlog** into a refined work item that holds everything needed to
write a spec and implement it without guessing. This skill **does not write a spec or any code**.

Board: `{{TRELLO_BOARD_URL}}` (name `{{TRELLO_BOARD_NAME}}`).

## Pipeline position

```
Backlog → Refined → In Development → Implemented → Shipped
   ↑ THIS SKILL      ↑ implement-feature (→ Implemented)   ↑ ship-feature
```

## Input (required)

The card **number** (Trello's card number, shown in the card URL as `.../c/<id>/<number>-...`, e.g.
`12` or `#12`) or its **name** (exact title or an unambiguous substring):

> `/refine-task 12`
> `/refine-task Password reset`

If nothing matches, or more than one card matches, show the candidates and ask which one.

## Prerequisites (verify FIRST — stop if missing)

1. **Trello MCP** (`trello` server). Verify with `list_boards`. If it fails or is missing, **stop**:
   > "The Trello MCP is not available. Install/enable the `trello` MCP server, then re-run
   > `/refine-task`."
2. **Superpowers plugin**: the `brainstorming` skill must be available. If not, **stop**:
   > "The Superpowers plugin isn't installed, and this skill uses it for refinement. Install it
   > (https://github.com/obra/superpowers), then re-run `/refine-task`."

## Workflow

Copy this checklist and track progress:

```
- [ ] Step 0: Verify prerequisites (Trello MCP + Superpowers)
- [ ] Step 1: Load project context (INDEX, constraints, every domain file)
- [ ] Step 2: Resolve the board and find the card (must be in Backlog)
- [ ] Step 3: Read everything on the card and summarize it
- [ ] Step 4: Refine with the user (Superpowers brainstorming)
- [ ] Step 5: Write the refined result back to the card
- [ ] Step 6: Move the card to Refined
- [ ] Step 7: Report
```

### Step 1: Load project context

Read `Meta/INDEX.md`, `Meta/constraints.md`, and **every** file in `Meta/domains/`. Refinement must
reason about the whole project, so know how each area works today before asking anything. Skim the
relevant code under `{{CODE_DIR}}/` when a domain file is a stub or the card touches something it
doesn't describe.

### Step 2: Resolve the board and find the card

1. `list_boards` → the board named `{{TRELLO_BOARD_NAME}}` (URL `{{TRELLO_BOARD_URL}}`); capture its
   `boardId`, then `set_active_board`.
2. `get_lists` → capture the ids of **`Backlog`** and **`Refined`**.
3. Find the card. `get_cards_by_list_id` on `Backlog`, then match:
   - a **number** against the card's `idShort` (or the number in its URL);
   - a **name** by exact title first, then unambiguous substring.
4. **The card must be in `Backlog`.** If it's found in another list, stop and tell the user where it
   is (e.g. already `Refined` → next step is `/implement-feature`). Don't refine it unless the user
   explicitly asks to re-refine.

### Step 3: Read everything on the card

- `get_card` (with `includeMarkdown: true`): title and description.
- `get_card_comments`: prior discussion and decisions.
- `get_acceptance_criteria`: existing acceptance-criteria items, if any.
- Attachments (mockups, references): note them; they are design input.

Summarize the card back to the user in 2–4 sentences before refining.

### Step 4: Refine with the user (the core of this skill)

**REQUIRED SUB-SKILL:** use `superpowers:brainstorming` to drive the dialogue: one question at a
time, multiple-choice when possible, each with your recommendation. Stop the brainstorming flow
before it writes a design doc or moves into implementation; in this skill the output is the refined
card, not a spec.

Walk `references/refinement-checklist.md` and cover, for this card:

- **User experience:** what the user does, sees and gets; how it's triggered.
- **Domain impact:** which domains it touches and which existing rules it changes or depends on.
- **Data & configuration:** values, limits, and where they should be configurable.
- **Assets / content:** anything that has to be made (art, copy, media) and whether it exists.
- **Edge cases:** interruptions, failures, things happening at the same time, invalid input.
- **Platforms & performance.**
- **Out of scope:** what this card explicitly does not include.

**Ask, don't assume.** Whenever the card doesn't say how something behaves in a given situation,
ask a concrete scenario question, e.g.:

> "If the user does X while Y is in progress, should it wait, cancel Y, or be refused?"

Every question lists the realistic options, **including keeping the current behavior** where one
exists, and ends with your recommendation and why.

Keep going until nothing is left that an implementer would have to guess. The exit bar: someone
could write the spec and build this **without asking anything further**.

### Step 5: Write the refined result back to the card

When the user confirms the refinement is complete:

1. `update_card_details`: write the description using `references/card-template.md`. Keep the
   original intent; integrate it rather than blindly overwriting.
2. Acceptance criteria go in the **"Acceptance Criteria"** checklist, not the description. If it
   doesn't exist, `create_checklist` (name `Acceptance Criteria`), then `add_checklist_item` for each
   criterion. Each one must be testable and phrased Given / When / Then.
3. `add_comment` with a short changelog of the key decisions reached.

### Step 6: Move the card to Refined

`move_card` with the `cardId`, the **`Refined`** list id, and the `boardId`. Only after the user has
confirmed refinement is complete.

### Step 7: Report

Tell the user: the card name and link, what changed (description, number of acceptance criteria),
the main decisions captured, and that it's in **Refined** and ready for `/implement-feature <card>`.

## Trello tools used (`trello` server)

`list_boards`, `set_active_board`, `get_lists`, `get_cards_by_list_id`, `get_card`,
`get_card_comments`, `get_acceptance_criteria`, `update_card_details`, `create_checklist`,
`add_checklist_item`, `add_comment`, `move_card`.

## Hard rules

- **Refinement only.** No spec, no code, no git branches, no commits.
- **The card must start in `Backlog`.** Don't move it to `Refined` until the user confirms the
  refinement is complete.
- **Never ask a bare question.** Give the options and your recommendation.
- Honor `Meta/constraints.md`: never surface real secrets, and respect the stack hygiene rules when
  describing impact.

## References

- `references/refinement-checklist.md`: the coverage checklist that drives the questions.
- `references/card-template.md`: the description structure written back to the card.
