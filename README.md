# ai-driven-boilerplate

A starting point for projects that are developed with AI agents (Claude Code, Cursor) and tracked
on a Trello board. It gives every new project the same structure:

- rules the agents always read (`AGENTS.md`, `CLAUDE.md`, `Meta/constraints.md`);
- a small knowledge base describing the project's **current state** (`Meta/INDEX.md`,
  `Meta/domains/`);
- a three-step task pipeline as Agent Skills (`Meta/skills/`), driven by Trello cards:
  **Backlog → Refined → In Development → Implemented → Shipped**.

This README explains the boilerplate. Once a project is set up from it, replace it with the
project's own README.

## Structure

```
.
├── README.md                 ← this file (boilerplate guide; replace after setup)
├── AGENTS.md                 ← must-follow rules for every agent (Cursor, Codex, Copilot)
├── CLAUDE.md                 ← Claude Code entry point; imports AGENTS.md
├── .gitignore                ← ignores worktrees, installed skill copies, local agent settings
├── Code/                     ← the application itself (engine project, app, service…)
└── Meta/                     ← the knowledge base
    ├── INDEX.md              ← map of the knowledge base; agents read it first
    ├── constraints.md        ← non-negotiable rules (secrets, git, stack, build & verify)
    ├── domains/              ← one concise file per area of the project, current state only
    ├── specs/                ← specs written by implement-feature, one per card
    └── skills/               ← source of truth for the skills
        ├── refine-task/
        ├── implement-feature/
        └── ship-feature/
```

## How the pieces fit

| Piece | Role |
| --- | --- |
| `AGENTS.md` / `CLAUDE.md` | Tell every agent to read `Meta/INDEX.md` and `Meta/constraints.md` first, and how to install the skills. |
| `Meta/INDEX.md` | The map. How to use the knowledge base, how to implement from a spec, how to keep the KB current, and the skill table. |
| `Meta/constraints.md` | Rules agents must never break: mock-only secrets, git workflow, stack hygiene, build/run/test commands. |
| `Meta/domains/` | Short, current-state summaries of each area of the project (e.g. for a game: gameplay, combat, characters). Rewritten on every ship, never appended to. |
| `Meta/specs/` | One spec per card, written by `implement-feature`, marked Shipped by `ship-feature`. |
| `Meta/skills/` | The pipeline. Copied by agents into `.claude/skills/` and `.cursor/skills/` (git-ignored). |

## The pipeline

| Skill | Card must be in | What it does |
| --- | --- | --- |
| `/refine-task <card>` | Backlog | Brainstorms the important questions with you (Superpowers **brainstorming**), writes decisions + acceptance criteria to the card, moves it to **Refined**. |
| `/implement-feature <card>` | Refined | Creates `feature/<slug>`, moves the card to **In Development**, writes the spec, plans (Superpowers **writing-plans**), implements, verifies, commits + pushes, hands off for testing. Moves to **Implemented** only after you confirm. |
| `/ship-feature <card>` | Implemented | After you approve the plan: updates `Meta/domains/` on the branch, merges to `main`, moves the card to **Shipped**, deletes the branch. |

`<card>` is the Trello card number (e.g. `12`) or its name.

Git model: **one worktree per feature**. Each card gets its own branch `feature/<slug>` checked
out at `.worktrees/<slug>/` (git-ignored), and the main checkout stays on `main`. Several features
can be in flight at once without colliding. See `Meta/constraints.md` §2.

> If every checkout of your stack is expensive (e.g. a game engine that re-imports the whole
> project per folder), you can switch to a single feature branch in the main checkout instead;
> update `Meta/constraints.md` §2, `AGENTS.md` rule 3 and the two skills that touch git.

## Requirements

- **Trello MCP** (`trello` server) connected to your agent.
- **Superpowers plugin** (https://github.com/obra/superpowers).
- **GitHub CLI** (`gh`), logged in, if you create the repo from the command line.
- A Trello board with the lists **Backlog**, **Refined**, **In Development**, **Implemented**,
  **Shipped** (exact names).

## Creating a new project from this boilerplate

1. **Create the repo.** On GitHub use **"Use this template"**, or:

   ```bash
   gh repo create <new-project> --private --template <owner>/ai-driven-boilerplate --clone
   ```

2. **Fill in the placeholders.** Search the repo for `{{` and replace every one:

   | Placeholder | Example |
   | --- | --- |
   | `{{PROJECT_NAME}}` | `Acme Shop` |
   | `{{PROJECT_DESCRIPTION}}` | `an e-commerce web app (Next.js + PostgreSQL)` |
   | `{{TRELLO_BOARD_URL}}` | `https://trello.com/b/<id>/<board-name>` |
   | `{{TRELLO_BOARD_NAME}}` | `ACME SHOP` |
   | `{{CODE_DIR}}` | `Code/acme-shop` |
   | `{{SETUP_COMMAND}}` | what a fresh worktree needs before it builds (install dependencies, copy local config), or "none" |
   | `{{BUILD_COMMAND}}` | how to compile/build headless, or "ask the user to check the editor" |
   | `{{TEST_COMMAND}}` | how to run automated tests, or "none yet" |
   | `{{RUN_INSTRUCTIONS}}` | how the user runs the project to test a change |

3. **Write the stack rules.** In `Meta/constraints.md` §3 ("Stack hygiene"), replace the examples
   with your stack's rules (generated folders, files that must travel together, versions you don't
   upgrade). Add matching ignores to `.gitignore`.
4. **Define the domains.** Create one stub per area of the project in `Meta/domains/` (copy the
   shape from `Meta/skills/ship-feature/references/domain-doc-template.md`) and list them in the
   Domains table in `Meta/INDEX.md`.
5. **Tune the refinement questions.** Add stack- or genre-specific questions to
   `Meta/skills/refine-task/references/refinement-checklist.md` and paths to
   `Meta/skills/implement-feature/references/code-exploration-checklist.md`.
6. **Put the application in `Code/`.**
7. **Replace this `README.md`** with the project's own, commit, and push.

The first agent task in the new repo installs the skills into `.claude/skills/` and
`.cursor/skills/` (see `AGENTS.md`). Restart Claude Code afterwards so the `/` commands appear.
