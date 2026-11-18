# {{PROJECT_NAME}} — Constraints (non-negotiable)

> Read this **before** writing code, running commands, or touching git. These rules override
> convenience. When in doubt, stop and ask the user.

## 1. Secrets & configuration: only ever write MOCK values

- You **may** read, create and edit secrets/config files (e.g. `.env`, `secrets.json`, API keys,
  service or store credentials), **but only ever put MOCK / placeholder values in them.** Never write
  a real credential, key, token or secret into any file.
- Never print, echo, copy or commit a **real** secret. If you come across one, leave it untouched and
  do not surface it in chat, logs or commits.
- Use clearly fake placeholders (`MOCK_VALUE`, `your-key-here`) so nobody mistakes them for real ones.
- Never commit files that contain real secrets.

## 2. Git: one worktree per feature (the branch is the merge/PR ref)

Every feature lives in its own **git worktree**, so several features can be in flight at once and
agents never collide in a shared checkout.

- **The main checkout stays on `main`. Never switch its branch.** Against it, run only git that
  doesn't change its working tree: `git fetch`, `git worktree add|remove|list|prune`,
  `git branch --list`, `git ls-remote`, `git pull --ff-only origin main`, and the ship merge
  (`ship-feature`). Never `git checkout` / `git switch` / `git checkout -b` there.
- **All feature work happens in its worktree** at `.worktrees/<slug>/` (git-ignored, never
  committed), on branch `feature/<slug>`:

  ```bash
  git fetch origin
  git worktree add .worktrees/<slug> -b feature/<slug> origin/main
  git -C .worktrees/<slug> push -u origin feature/<slug>
  ```

  Edit, build, test and commit **inside** `.worktrees/<slug>/`, never in the main checkout.
- **Collision pre-flight** before creating a worktree: `git fetch origin`, `git worktree list`,
  `git branch --list feature/<slug>`, `git ls-remote --heads origin feature/<slug>`. If the worktree
  or branch already exists, or the main checkout isn't clean on `main`, stop and report: the feature
  may already be in flight. Never stash, discard or reset someone's changes to get past this.
- **Per-worktree setup.** A worktree is a fresh checkout: git-ignored files (dependencies, local
  config, build caches) aren't there. Run the project's setup in it before building (see §4).
- **Editors and IDEs.** The user tests a feature by opening its worktree folder
  (`.worktrees/<slug>/`). If their editor rewrites files there, review them before committing.
- Commits on `main` only happen through `ship-feature`'s merge, after the user approves the ship
  plan. Never force-push, never `reset --hard`, never skip hooks, never change git config. Never
  commit secrets (see §1).

## 3. Stack hygiene

> Replace these examples with the rules of your stack.

- **Never edit or commit generated folders** (build output, caches, dependency folders, generated
  project files). List them here and in `.gitignore`.
- **Files that travel together** (e.g. asset + metadata files, migrations + snapshots) are moved,
  renamed and deleted together.
- **Don't upgrade** the framework, engine, runtime or dependencies without asking.
- **Don't hand-edit generated or serialized files** unless the change is trivial and certain.

## 4. Build, run & verify

The skills use these to check a change before handing it to the user. Run them from the feature's
worktree (`.worktrees/<slug>/`), not the main checkout.

- **Worktree setup (fresh checkout):** {{SETUP_COMMAND}}
- **Build / compile:** {{BUILD_COMMAND}}
- **Automated tests:** {{TEST_COMMAND}} (optional: write tests when they're worth it, for pure
  logic; don't force them where a manual test covers it better).
- **How the user runs it to test a change:** {{RUN_INSTRUCTIONS}}

## 5. Code

- Prefer editing existing files over creating new ones; don't create docs or files unless asked.
- When you add tests, run them before you call the work done and report which tests changed or were
  added.

## 6. External services & data

- Do not run requests against production services that could expose user or sensitive data. Ask
  the user to run them and share only what is safe.

## 7. Scope & safety defaults

- For reversible, low-risk choices (naming, formatting, equivalent approaches), decide and note it.
  For scope changes or destructive actions, ask first.
