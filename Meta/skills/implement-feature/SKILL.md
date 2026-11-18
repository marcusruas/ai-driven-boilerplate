---
name: implement-feature
description: Spec and implement a refined task end to end. Use when the user names a Trello card in Refined (by number or name) and wants it built. Creates the feature/<slug> branch in its own git worktree, moves the card to In Development, writes the spec in Meta/specs/, plans with the Superpowers writing-plans skill, implements it, verifies it builds, commits and pushes, hands the change to the user to test, and moves the card to Implemented only after the user confirms. Needs the Trello MCP and Superpowers.
disable-model-invocation: true
---

# Implement Feature

Take a **Refined** Trello card, write its spec, build it on `feature/<slug>` in its own worktree
(`.worktrees/<slug>/`), hand it to the user to test, and move the card to **Implemented** once they confirm. This skill writes the spec, the code
and the assets; it does **not** merge to `main` (that's `ship-feature`).

Board: `{{TRELLO_BOARD_URL}}` (name `{{TRELLO_BOARD_NAME}}`).

## Pipeline position

```
Backlog → Refined → In Development → Implemented → Shipped
           ↑ refine-task   ↑ THIS SKILL (start)  ↑ THIS SKILL (user confirms)   ↑ ship-feature
```

## Input (required)

The card **number** (`12` / `#12`, Trello's `idShort`) or **name** (exact title or unambiguous
substring):

> `/implement-feature 12`

Show candidates and ask if nothing or more than one card matches.

## Prerequisites (verify FIRST — stop if missing)

1. **Trello MCP** (`trello` server). Verify with `list_boards`. If missing, **stop**:
   > "The Trello MCP is not available. Install/enable the `trello` MCP server, then re-run
   > `/implement-feature`."
2. **Superpowers plugin**: the `writing-plans` skill must be available. If not, **stop** and tell
   the user to install Superpowers (https://github.com/obra/superpowers).
3. **Clean main checkout**: the main checkout is on `main` and `git status` shows no uncommitted
   changes. It's the permanent hub; all feature work happens in a worktree
   (`Meta/constraints.md` → "Git"). If it's dirty or on another branch, **stop** and ask. Never
   stash, discard or reset changes to get past this.

## Workflow

Copy this checklist and track progress:

```
- [ ] Step 0: Verify prerequisites
- [ ] Step 1: Resolve the board, find the card, read everything
- [ ] Step 2: Validate the card is ready (Refined, or resuming from In Development)
- [ ] Step 3: Load context (INDEX, constraints, every domain, conversation)
- [ ] Step 4: Create the feature worktree + branch and push it
- [ ] Step 5: Move the card to In Development + write the Implementation block
- [ ] Step 6: Explore the code and write the spec (Meta/specs/<slug>.md); commit + push
- [ ] Step 7: Write the implementation plan (Superpowers writing-plans)
- [ ] Step 8: Implement
- [ ] Step 9: Verify (build, optional tests)
- [ ] Step 10: Commit + push; build the GitHub compare link
- [ ] Step 11: Hand off to the user to test — WAIT for confirmation
- [ ] Step 12: Move the card to Implemented
- [ ] Step 13: Report
```

### Step 1: Resolve the board, find the card, read everything

1. `list_boards` → `{{TRELLO_BOARD_NAME}}`; `set_active_board`. `get_lists` → ids of **`Refined`**,
   **`In Development`** and **`Implemented`**.
2. Find the card by number (`idShort`) or name across those lists (`get_cards_by_list_id`).
   Capture `cardId`, `cardUrl`, current list.
3. Read everything: `get_card` (`includeMarkdown: true`), `get_card_comments`,
   `get_acceptance_criteria`, and note attachments (mockups, references).

### Step 2: Validate the card is ready

- **In `Refined`:** normal start. The description must have the refined sections (`Problem`,
  `Decisions (from refinement)`, `Domains affected`; see
  `../refine-task/references/card-template.md`). If it's still thin or ambiguous, **stop** and send
  the user to `/refine-task`.
- **In `In Development`** with an `## Implementation` block: this is a **resume**. Read the branch,
  worktree and spec from the block. If the worktree is gone, recreate it from the pushed branch
  (`git worktree add .worktrees/<slug> feature/<slug>`, or
  `git worktree add .worktrees/<slug> -b feature/<slug> origin/feature/<slug>` if the local branch
  is missing) and run the worktree setup. Continue from the first unfinished step.
- **In `Backlog`:** stop, send the user to `/refine-task`.
- **In `Implemented` / `Shipped`:** stop and tell the user; next step is `/ship-feature` (or
  nothing).

### Step 3: Load context

Follow `Meta/INDEX.md` → "Implementing from a spec":

1. `Meta/INDEX.md`, `Meta/constraints.md`.
2. **Every** file in `Meta/domains/`, so you know how the rest of the project works. Look actively
   for cross-domain interactions.
3. **Conversation context:** if refinement happened earlier in this chat, its confirmed decisions
   are authoritative alongside the card. Don't re-ask what's already answered.

If the card conflicts with existing behavior documented elsewhere and doesn't resolve it, **stop and
ask the user**. Don't guess.

### Step 4: Create the feature worktree + branch and push it

Derive the **slug** from the card title: lowercase kebab-case (`Password reset` →
`password-reset`). Branch: `feature/<slug>`. Worktree: `.worktrees/<slug>/`.

Collision pre-flight (against the main checkout), then create:

```bash
git fetch origin
git status --short --branch                          # clean, on main
git worktree list                                    # .worktrees/<slug> must not exist
git branch --list "feature/<slug>"                   # must be empty
git ls-remote --heads origin "feature/<slug>"        # must be empty
git worktree add .worktrees/<slug> -b feature/<slug> origin/main
git -C .worktrees/<slug> push -u origin feature/<slug>
```

If the worktree or branch already exists, stop and ask: resume it, or choose a different slug. If a
git lock error appears (another agent is mid-`worktree add`), wait briefly and retry; never force.

Then run the **worktree setup** from `Meta/constraints.md` §4 (`{{SETUP_COMMAND}}`) inside
`.worktrees/<slug>/`. **From here on, every edit, build, test and commit happens inside the
worktree**, never in the main checkout.

### Step 5: Move the card to In Development + write the Implementation block

1. `move_card` → **`In Development`**.
2. `update_card_details`: merge an `## Implementation` block into the description (keep everything
   else) per `references/card-metadata.md`, and set `## Status` to `In development.`
3. `add_comment`: `🛠️ In development (YYYY-MM-DD): branch feature/<slug>.`

### Step 6: Explore the code and write the spec

1. Walk `references/code-exploration-checklist.md` for every domain the card touches. The spec must
   name **real** files and modules, not placeholders. Note anything in the code that would block the
   new behavior today and write down the required fix.
2. Write `Meta/specs/<slug>.md` from `references/spec-template.md`. Requirements:
   - Status banner, card link, purpose paragraph.
   - Decisions table from the card's `Decisions` + conversation.
   - Behavior, configuration values, assets, file checklist, acceptance criteria (copied 1:1 from
     the Trello checklist), manual test steps.
   - No TBDs, no contradictions, scope matches the card's `Out of scope`.
3. Commit and push the spec on the feature branch, from the worktree:

   ```bash
   git -C .worktrees/<slug> add Meta/specs/<slug>.md
   git -C .worktrees/<slug> commit -m "Add spec for <card title>"
   git -C .worktrees/<slug> push origin feature/<slug>
   ```

Show the user a short summary of the spec. If writing it surfaced a real gap (not covered by the
card), ask before continuing.

### Step 7: Write the implementation plan

**REQUIRED SUB-SKILL:** use `superpowers:writing-plans`. Feed it the spec (authoritative design), the
card (intent, acceptance criteria, out of scope), the conversation decisions and the code touchpoints
from Step 6. Keep the plan aligned with the spec. If planning surfaces a gap or conflict, stop and
ask; don't invent behavior.

### Step 8: Implement

Work through the plan task by task, inside the worktree (`.worktrees/<slug>/{{CODE_DIR}}/`). Follow
`references/implementation-checklist.md` and `Meta/constraints.md` (stack hygiene, mock secrets, no
upgrades without asking). When a step needs something only the user can do (an editor action, an
account setting), list exact instructions for them rather than guessing.

**Automated tests are optional.** Add them when they're worth it, for pure logic; don't force tests
onto things a manual test covers better.

### Step 9: Verify

Run these from the worktree (`.worktrees/<slug>/`).

1. **Build:** run the build check from `Meta/constraints.md` §4 (`{{BUILD_COMMAND}}`). Fix every
   error before handing off.
2. **Tests (if any exist for this area):** run them (`{{TEST_COMMAND}}`). Fix failures before
   handing off. Record which tests were added or changed.

### Step 10: Commit + push; build the compare link

1. Review `git status`. Commit only this feature's work, and check that any files an editor rewrote
   are intended changes.

   ```bash
   git -C .worktrees/<slug> add -A
   git -C .worktrees/<slug> commit -m "Implement <card title>"
   git -C .worktrees/<slug> push origin feature/<slug>
   ```

2. Confirm the push landed: `git -C .worktrees/<slug> rev-parse HEAD` must equal the SHA from
   `git ls-remote --heads origin feature/<slug>`.
3. Build the compare link from the remote (`git remote get-url origin`, strip `.git`):
   `https://github.com/<owner>/<repo>/compare/main...feature/<slug>`, with a size summary from
   `git diff --stat origin/main...feature/<slug>` and `git log --oneline origin/main..feature/<slug>`.

Never commit on `main`, force-push, skip hooks, or amend a pushed commit. Follow-up fixes are new
commits.

### Step 11: Hand off to the user to test — WAIT for confirmation

Tell the user it's implemented, building, committed and pushed, then ask them to test it:

- **How to test:** {{RUN_INSTRUCTIONS}}, from the worktree folder `.worktrees/<slug>/` (that's
  where `feature/<slug>` is checked out; the main checkout stays on `main`).
- **Manual test steps** from the spec, and the acceptance criteria they map to.
- **Steps they need to do themselves** (if any), with exact instructions.
- **The GitHub compare link** with its commit/file/line counts.
- What changed, in short bullets.

**Do not move the card until the user explicitly confirms it works.** If they report problems, go
back to Step 8, fix, verify (Step 9), make a **follow-up** commit + push (Step 10), and hand off
again. If the behavior changes in a way that contradicts the spec, update the spec in the same
follow-up commit.

### Step 12: Move the card to Implemented

After the user's confirmation:

1. `move_card` → **`Implemented`**.
2. `update_card_details`: set `## Status` to `Implemented: ready to ship.`
3. `add_comment`: `✅ Implemented (YYYY-MM-DD): tested and confirmed by the user. Branch feature/<slug>.`

### Step 13: Report

1. **Card:** name + link; moved to **Implemented**.
2. **Spec:** `Meta/specs/<slug>.md`, one-line summary.
3. **Plan:** where writing-plans saved it.
4. **Built:** what was implemented; acceptance criteria covered.
5. **Tests:** which automated tests were added/changed, or "none" and why.
6. **Branch:** `feature/<slug>` pushed, with the compare link and its size.
7. **Open items:** anything deferred.
8. **Next:** `/ship-feature <card>` to update the knowledge base and merge to `main`.

## Trello tools used (`trello` server)

`list_boards`, `set_active_board`, `get_lists`, `get_cards_by_list_id`, `get_card`,
`get_card_comments`, `get_acceptance_criteria`, `update_card_details`, `move_card`, `add_comment`.

## Hard rules

- **Card must start in `Refined`** (or be resumed from `In Development`). Move it to
  `In Development` when work starts, and to `Implemented` **only after the user confirms** the test.
- **Spec before plan, plan before code.** The spec is committed on the branch before
  implementation starts.
- **Stop on conflict or gap** between the card/spec and existing behavior; ask, don't guess.
- **Worktree-first.** All work happens in `.worktrees/<slug>/`; the main checkout stays on `main`
  and its branch is never switched. Never stash, reset or discard the user's changes.
- **No merge to `main`.** That's `/ship-feature`.
- Honor `Meta/constraints.md`.

## References

- `references/card-metadata.md`: the `## Implementation` block written to the card (read by
  `ship-feature`).
- `references/spec-template.md`: spec outline and quality bar.
- `references/code-exploration-checklist.md`: what to read before writing the spec.
- `references/implementation-checklist.md`: quality bar while implementing.
- `../refine-task/references/card-template.md`: the refined card shape (input).
