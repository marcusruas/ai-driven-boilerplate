# Card metadata: the `## Implementation` block

`implement-feature` writes this block into the card description when it starts work (Step 5).
`ship-feature` reads the branch, worktree and spec from it, so **don't rename the labels**. Keep the
refined sections (Problem, Decisions, …) above it.

```markdown
## Implementation
- **Branch:** `feature/<slug>`
- **Worktree:** `.worktrees/<slug>`
- **Spec:** `Meta/specs/<slug>.md`

## Status
In development.
```

`## Status` moves through:

| Stage | Status line | Set by |
| --- | --- | --- |
| Refined | `Refined: ready for implementation.` | `refine-task` |
| In Development | `In development.` | `implement-feature` (Step 5) |
| Implemented | `Implemented: ready to ship.` | `implement-feature` (Step 12) |
| Shipped | `Shipped (YYYY-MM-DD).` | `ship-feature` |

## Rules

| Field | Format |
| --- | --- |
| **Branch** | Required. `feature/<slug>` in backticks (the merge/PR ref). |
| **Worktree** | Required. Where the feature is checked out, `.worktrees/<slug>`, in backticks. |
| **Spec** | Required. Path from the repo root, `Meta/specs/<slug>.md`, in backticks. |

The slug in all three must match (`password-reset` → `feature/password-reset` +
`.worktrees/password-reset` + `Meta/specs/password-reset.md`).

## Parsing

- `**Branch:**` → the value inside the backticks.
- `**Worktree:**` → the path inside the backticks.
- `**Spec:**` → the path inside the backticks.

If the block or the `**Branch:**` line is missing, stop and tell the user to run
`/implement-feature` (or add the block by hand). If `**Worktree:**` is missing, use
`.worktrees/<slug>`.
