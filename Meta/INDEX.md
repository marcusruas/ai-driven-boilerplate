# {{PROJECT_NAME}} — Project Knowledge Base (INDEX)

> **Read this file first on every task.** It is the map of the {{PROJECT_NAME}} knowledge base.
> It tells you where things live, what each document is for, and the rules you must never break.

{{PROJECT_NAME}} is {{PROJECT_DESCRIPTION}}. The application lives in `{{CODE_DIR}}/`; this `Meta/`
folder is the knowledge base, and every path below is relative to it.

## How to use this knowledge base

1. **Always start here.** Then read `constraints.md` before writing or running anything.
2. For a specific area of the project, read its file in `Meta/domains/`. Each domain file is a short
   summary of **how that part of the project works today** and where it lives in the code. Read it
   before you change that area.
3. The deep design docs for work in progress are in `Meta/specs/`. The `domains/` files are the
   distilled current state; `specs/` holds the detail of how a change was or will be built.

The knowledge base describes **the current state of the project**, not its history. It does not
track every feature ever built. When something changes, the domain file is rewritten to match; old
behavior is removed, not kept alongside.

### Domains

| File                    | Covers                                   | Status      |
| ----------------------- | ---------------------------------------- | ----------- |
| `domains/<domain>.md`   | <What this area of the project covers>   | TODO (stub) |

Add a domain when an area of the project grows big enough to need its own summary; don't create one
for a single feature.

### What a domain file contains

Keep each one concise. It is context for an AI agent, not a design bible. Aim for:

- **Purpose:** what this part of the project is, in two or three sentences.
- **Current behavior:** the rules as they work right now.
- **Where it lives:** the files, modules and assets that implement it, with paths under
  `{{CODE_DIR}}/`.
- **Interactions:** which other domains it depends on or affects.
- **Gotchas:** non-obvious constraints an agent would get wrong without being told.

## Implementing from a spec

When you are **implementing** a change from a spec in `specs/` (not just reading for context):

1. Read the **target spec** end to end, plus the `domains/` files it touches.
2. Read **every other file in `domains/`** so you know how the rest of the project works today. The
   change you are building is one piece of a shared system, not an isolated module.
3. While designing and coding, actively look for **cross-domain interactions**: shared components,
   data and configuration, state, events, public interfaces, persistence, and UI.
4. If you find a **conflict, ambiguity or gap** between the spec and **existing behavior documented
   elsewhere**, and the spec does **not** already resolve it, **stop and ask the user**. Do not
   guess, silently pick one behavior, or paper over the mismatch with a local workaround.
5. When you ask, be concrete: name the domain, describe the conflicting behavior, quote or summarize
   what the spec says (or leaves out), and offer one or two sensible resolutions if you have them.
   Wait for an answer before proceeding on that point.

Treat unresolved cross-domain conflicts as **blockers**, not details you can defer.

## Keeping the knowledge base current

Every time a change is merged into `main`, the knowledge base **must be updated** so it reflects
the project as it now is. Update the affected `domains/` files (and this index if a domain was added
or removed), not only `specs/`.

- **Rewrite, don't append.** Change the domain file to describe the new current behavior and delete
  what is no longer true. Don't add changelog entries or "previously..." notes.
- **Stay summarized.** If a domain file starts turning into a feature list, condense it. Detail
  belongs in `specs/` or in the code.
- Stale docs mislead agents and humans alike; treat the knowledge-base update as part of shipping
  the change, not optional follow-up work.

## Skills (the task pipeline)

Reusable Agent Skills live in `skills/` (the version-controlled source of truth). `AGENTS.md` →
"Skills — ensure installed" copies them into `.claude/skills/` and `.cursor/skills/`.

**Task board:** tasks are tracked on the Trello board **{{TRELLO_BOARD_URL}}**. The skills read and
update cards on this board, so AI agents **must** use the **Trello MCP** (`trello` server) to run
them. If the Trello MCP is not available, tell the user how to install it and stop. Don't work
around it by guessing card contents.

Pipeline: **Backlog → Refined → In Development → Implemented → Shipped**. Each skill takes the
card number or name.

| Skill | Invoke | Card must be in | Purpose |
| --- | --- | --- | --- |
| `skills/refine-task` | `/refine-task <card>` | Backlog | Brainstorm the questions that matter with the user (Superpowers **brainstorming**), write the decisions and acceptance criteria to the card, move it to **Refined**. No code. |
| `skills/implement-feature` | `/implement-feature <card>` | Refined | Create `feature/<slug>`, move the card to **In Development**, write `specs/<slug>.md`, plan (Superpowers **writing-plans**), implement, verify, commit + push, hand off for testing. Moves the card to **Implemented** only after the user confirms. |
| `skills/ship-feature` | `/ship-feature <card>` | Implemented | After the user approves the plan: update `domains/` on the feature branch, merge to `main`, move the card to **Shipped**, delete the branch. |
