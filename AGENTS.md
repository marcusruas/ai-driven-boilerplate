<!-- agents-pointer -->
# AGENTS.md — {{PROJECT_NAME}}

Guidance for AI agents working on **{{PROJECT_NAME}}**, {{PROJECT_DESCRIPTION}}. This repo is the
whole workspace:

- `{{CODE_DIR}}/` — the application.
- `Meta/` — the project knowledge base (domains, constraints, specs, skills).

## Must-follow rules

1. **Read `Meta/INDEX.md` before any task.** It is the map of the project knowledge base. Read
   `Meta/constraints.md` too. Those rules are non-negotiable.
2. **Specs live in `Meta/specs/`.** Treat them as the authoritative design for in-flight work and
   follow their format when writing or updating specs.
3. **Worktree per feature.** Do new work in an isolated git worktree at `.worktrees/<slug>/` on
   branch `feature/<slug>`. The main checkout stays on `main`; never switch its branch. See
   `Meta/constraints.md` → "Git".
4. **Push after branch work.** When you create or update a feature branch, push it to `origin`
   right after (`git push -u origin HEAD` the first time). Never force-push. Commits on `main` need
   explicit user approval (the `ship-feature` plan confirmation counts).
5. **Use the task pipeline.** Work tracked on the Trello board goes through the skills in
   `Meta/skills/`: `refine-task` → `implement-feature` → `ship-feature`. See `Meta/INDEX.md` →
   "Skills".

## Skills — ensure installed (do this on your first task)

`Meta/skills/` is the version-controlled source of truth for the skills, but it is not a folder any
tool auto-discovers. On your first task, make sure every `Meta/skills/<name>/` folder is copied to:

- `.claude/skills/<name>/` (Claude Code)
- `.cursor/skills/<name>/` (Cursor)

If a copy is missing or its `SKILL.md` differs from the source, copy the whole folder over
(including `references/`). These copies are git-ignored; always edit skills in `Meta/skills/`.

Prerequisites the skills check before running:

- **Trello MCP** (`trello` server): all three skills.
- **Superpowers plugin** (https://github.com/obra/superpowers): `refine-task` (brainstorming) and
  `implement-feature` (writing-plans).

## Quick context

- **Stack:** {{PROJECT_DESCRIPTION}}.
- **Task board:** {{TRELLO_BOARD_URL}}

## Other constraints (see `Meta/constraints.md` for the full list)

- Secrets/config files only ever get **mock/placeholder** values.
- Follow the stack hygiene rules in `Meta/constraints.md` §3.
