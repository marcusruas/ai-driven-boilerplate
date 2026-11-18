---
name: ship-feature
description: Ship an implemented task. Use when the user names a Trello card in Implemented (by number or name) and wants it landed. Reads the branch, worktree and spec from the card's Implementation block, updates the Meta knowledge base (domain files, spec status) on the feature branch in its worktree first, merges feature/<slug> into main after the user approves the plan, moves the card to Shipped, and removes the worktree and the merged branch. Needs the Trello MCP.
disable-model-invocation: true
---

# Ship Feature

Ship an **Implemented** card: first **update the knowledge base on the feature branch** (in its
worktree) so `Meta/domains/` describes the project as it is after this change, then **merge
`feature/<slug>` into `main`**, move the card to **Shipped**, and **remove the worktree and the
merged branch**.

Order matters: the knowledge-base update is committed on `feature/<slug>` **before** the merge, so it
lands on `main` together with the code, not as a separate follow-up commit.

This skill updates docs and merges; it does not implement (`implement-feature`) or refine
(`refine-task`).

Board: `{{TRELLO_BOARD_URL}}` (name `{{TRELLO_BOARD_NAME}}`).

## Pipeline position

```
Backlog → Refined → In Development → Implemented → Shipped
                                       ↑ implement-feature   ↑ THIS SKILL
```

## Input (required)

The card **number** (`12` / `#12`, Trello's `idShort`) or **name** (exact title or unambiguous
substring):

> `/ship-feature 12`

The **branch**, **worktree** and **spec** are read from the card's `## Implementation` block (see
`../implement-feature/references/card-metadata.md`), not asked for. If the block or its
`**Branch:**` line is missing, stop and tell the user.

## Prerequisites (verify FIRST — stop if missing)

1. **Trello MCP** (`trello` server). Verify with `list_boards`. If missing, **stop**:
   > "The Trello MCP is not available. Install/enable the `trello` MCP server, then re-run
   > `/ship-feature`."
2. **Clean main checkout**: on `main`, `git status` shows no uncommitted changes. It's the permanent
   hub where the merge happens. If dirty or on another branch, stop and ask. Never stash, discard or
   reset changes.

## Workflow

Copy this checklist and track progress:

```
- [ ] Step 0: Verify prerequisites
- [ ] Step 1: Resolve the board, find the card, read everything, parse Implementation
- [ ] Step 2: Load context (INDEX, constraints, every domain, the spec)
- [ ] Step 3: Confirm the ship plan with the user — WAIT for approval
- [ ] Step 4: Make sure the feature worktree exists and is up to date
- [ ] Step 5: Verify the branch in its worktree (builds, optional tests)
- [ ] Step 6: Build the feature digest (card + spec + code on the branch)
- [ ] Step 7: Update the knowledge base in the worktree; show the diff; commit + push
- [ ] Step 8: Merge feature/<slug> → main from the main checkout; push
- [ ] Step 9: Move the card to Shipped
- [ ] Step 10: Remove the worktree and delete the merged branch
- [ ] Step 11: Report
```

### Step 1: Resolve the board, find the card, read everything, parse Implementation

1. `list_boards` → `{{TRELLO_BOARD_NAME}}`; `set_active_board`. `get_lists` → ids of
   **`Implemented`** and **`Shipped`**.
2. Find the card by number (`idShort`) or name. **It must be in `Implemented`.** If it's elsewhere,
   stop and say where it is and which skill comes next.
3. Read everything: `get_card` (`includeMarkdown: true`), `get_card_comments`,
   `get_acceptance_criteria`.
4. Parse `## Implementation`: **`branch`** from `**Branch:**`, **`worktree`** from `**Worktree:**`
   (default `.worktrees/<slug>`), **`specPath`** from `**Spec:**`.
5. Check that `branch` exists on origin (`git ls-remote --heads origin <branch>`) and that
   `specPath` exists on that branch. If either is missing, stop.

### Step 2: Load context

1. `Meta/INDEX.md`, `Meta/constraints.md`.
2. **Every** `Meta/domains/*.md`: what the knowledge base claims **today**.
3. The spec (`Meta/specs/<slug>.md` on the branch): design intent, decisions, values, assets.

### Step 3: Confirm the ship plan — WAIT for approval

Before any commit or merge, present:

> Ship `feature/<slug>` → `main`. Card: <title + link>. Spec: `<specPath>`.
> Knowledge base files to update on the branch first: <list of domain files, INDEX if a domain is
> added, the spec's status banner>. Then merge to `main`, move the card to **Shipped**, and remove
> the worktree and the branch.

**Wait for explicit approval.** Merging isn't something this skill undoes. The approval also covers
the knowledge-base commit in Step 7 (its diff is shown, but it doesn't need a separate review).

### Step 4: Make sure the feature worktree exists and is up to date

```bash
git fetch origin
git worktree list                                    # is <worktree> there?
```

- **Present:** `git -C <worktree> status` must be clean (if not, stop and ask), then
  `git -C <worktree> pull --ff-only origin <branch>`.
- **Missing:** recreate it from the pushed branch, then run the worktree setup from
  `Meta/constraints.md` §4:

  ```bash
  git worktree add <worktree> <branch>
  # if the local branch is absent:
  # git worktree add <worktree> -b <branch> origin/<branch>
  ```

### Step 5: Verify the branch in its worktree

From `<worktree>`:

1. **Builds:** run the build check from `Meta/constraints.md` §4 (`{{BUILD_COMMAND}}`).
2. **Tests, if the project has any:** run them (`{{TEST_COMMAND}}`).

If anything fails, **stop**. Don't write docs that describe behavior the build contradicts, and
don't merge.

### Step 6: Build the feature digest

| Source | What to take from it |
| --- | --- |
| Trello card | Problem, decisions, edge cases, out of scope, acceptance criteria |
| Spec | Design intent, behavior, values, assets |
| Code on the branch | What actually runs: modules, configuration, data, entry points |
| Existing `Meta/domains/*.md` | What's already described; reconcile, don't duplicate |

**Code on the branch wins** for what runs; the spec for intent; the card for decisions. Don't treat
an acceptance criterion as shipped unless the code confirms it. Flag drift in the report. Walk
`references/knowledge-update-checklist.md`.

### Step 7: Update the knowledge base in the worktree; show the diff; commit + push

Edit the files **in `<worktree>`** (on the feature branch), never in the main checkout. Follow
`Meta/INDEX.md` → "Keeping the knowledge base current":

| Document | What to do |
| --- | --- |
| `Meta/domains/<domain>.md` (each one the change touches) | **Rewrite** to describe the current behavior, using `references/domain-doc-template.md`. Remove what's no longer true. No changelog, no "previously…". Keep it concise: it's a summary of how the domain works, not a list of features. |
| New `Meta/domains/<domain>.md` | Only if the change created a whole new area of the project. Add its row to the Domains table in `Meta/INDEX.md`. |
| `Meta/INDEX.md` | Domain table status (e.g. stub → current) or a new row. Nothing else. |
| `Meta/constraints.md` | Only if the change introduced a new non-negotiable rule that agents must know. |
| The spec | Change the banner to **Status: Shipped** (date, branch). Don't rewrite the body; add a short "Post-ship notes" section only if the code differs from it. |

Show the user a short summary of the diff (files changed + the rule changes in bullets), then commit
and push on the feature branch:

```bash
git -C <worktree> add Meta/
git -C <worktree> commit -m "Update knowledge base for <card title>"
git -C <worktree> push origin <branch>
```

### Step 8: Merge feature/<slug> → main from the main checkout; push

The main checkout is already on `main` (it never switches branch):

```bash
git fetch origin
git pull --ff-only origin main
git merge --no-ff origin/<branch> -m "Merge <branch>: <card title>"
git push origin main
```

**On a merge conflict, stop.** Report the conflicting files. Don't resolve by guessing; ask the user
how to proceed (resolve together, or `git merge --abort`). Never force-push, `reset --hard`, or skip
hooks.

### Step 9: Move the card to Shipped

1. `move_card` → **`Shipped`**.
2. `update_card_details`: set `## Status` to `Shipped (YYYY-MM-DD).`
3. `add_comment`: `🚀 Shipped (YYYY-MM-DD): feature/<slug> merged to main; knowledge base updated; worktree removed.`

Don't rename the card.

### Step 10: Remove the worktree and delete the merged branch

From the main checkout (never from inside the worktree you're removing):

```bash
git worktree remove <worktree>         # refuses if the worktree has uncommitted changes
git worktree prune
git branch -d <branch>                 # -d only: refuses if not fully merged
git push origin --delete <branch>
```

- If `worktree remove` refuses (uncommitted changes), **stop** and ask; never `--force`.
- If `branch -d` refuses, the branch isn't fully merged: **stop** and report; never `-D`.
- Delete the remote branch only after the merge to `main` is pushed.

### Step 11: Report

1. **Card:** name + link; moved to **Shipped**.
2. **Verified:** build / test result on the branch before merging.
3. **Knowledge base:** files changed and the headline behavior changes; committed on the branch
   before the merge.
4. **Merged:** `<branch>` → `main`, pushed (merge commit SHA).
5. **Drift:** any mismatch between code, spec and card worth a follow-up.
6. **Cleanup:** worktree removed, branch deleted locally and on origin (or what was left and why).

## Trello tools used (`trello` server)

`list_boards`, `set_active_board`, `get_lists`, `get_cards_by_list_id`, `get_card`,
`get_card_comments`, `get_acceptance_criteria`, `update_card_details`, `move_card`, `add_comment`.

## Hard rules

- **Card must be in `Implemented`.** Branch, worktree and spec come from its `## Implementation`
  block.
- **Approve before merge.** Never merge without the user's explicit approval of the Step 3 plan.
- **Knowledge base first, then merge**, so the docs land on `main` with the code.
- **Rewrite domain docs to the current state**; they describe the project now, not its history.
- **Worktree-first.** Knowledge-base edits happen in the feature worktree; the main checkout stays
  on `main` and only fetches, fast-forwards and merges.
- **Stop on a failed build, a failed test or a merge conflict.** A half-shipped change is worse than
  stopping.
- `git branch -d` only, never `-D`; never `worktree remove --force`; never force-push or reset;
  never stash or discard the user's changes.

## References

- `../implement-feature/references/card-metadata.md`: the `## Implementation` block format.
- `references/knowledge-update-checklist.md`: what to update and where to take it from.
- `references/domain-doc-template.md`: structure of a `Meta/domains/<domain>.md` file.
