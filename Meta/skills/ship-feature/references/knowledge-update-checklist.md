# Knowledge base update checklist (before the merge)

Run this on the feature branch (ship-feature Step 6–7). Work top to bottom; mark a section "N/A" in
the report when it doesn't apply.

> **Source priority:** code on the branch > spec > Trello card > existing knowledge base. If they
> disagree, document what **runs** and flag the drift.

## 1. Feature digest (drives every edit)

- [ ] One sentence: what changed for the user.
- [ ] Current rules after the change (values, states, conditions).
- [ ] Decisions from the card that shaped the behavior.
- [ ] Edge cases that actually shipped (check the code, not only the card).
- [ ] Code and asset touchpoints: modules, configuration, data, assets.
- [ ] Out of scope: don't document it as current behavior.

## 2. Domain files (`Meta/domains/`)

- [ ] Identify every domain the change touched.
- [ ] Rewrite each to the current state with `domain-doc-template.md`; delete what's no longer
      true.
- [ ] Keep each file concise; condense if it's turning into a feature list.
- [ ] "Where it lives" paths exist on the branch.
- [ ] If the change created a genuinely new area of the project, create its domain file.

## 3. `Meta/INDEX.md`

- [ ] Domain table: update status (stub → current) or add a row for a new domain.
- [ ] Nothing else changes in INDEX for a normal ship.

## 4. `Meta/constraints.md`

- [ ] Only add a rule if the change introduced a non-negotiable agents must follow everywhere.

## 5. Spec (`Meta/specs/<slug>.md`)

- [ ] Banner → **Status: Shipped** + date + branch.
- [ ] "Post-ship notes" only if the code diverged from the spec.

## 6. Consistency pass

- [ ] Domain files agree with each other on names, values and behavior.
- [ ] No domain file still describes removed behavior.

## 7. Exit bar

- [ ] A new agent reading `INDEX.md` → the relevant domain files understands how the project behaves
      now.
- [ ] The user has seen a summary of the diff before the commit.
