# Domain doc template (`Meta/domains/<domain>.md`)

A concise summary of how one area of the project works **right now**, written for an AI agent. It is
rewritten (not appended to) whenever a shipped change alters the domain. Detail lives in the specs
and the code.

```markdown
# <Domain>

> Status: **current** (last updated YYYY-MM-DD by <card title>).

## Purpose

<Two or three sentences: what this part of the project is and why it exists.>

## Current behavior

<Present-tense rules as they work today. Bullets, each one checkable against the project.>
- ...

## Where it lives

<Real paths under `{{CODE_DIR}}/`: modules, configuration, data, assets.>
- ...

## Interactions

<Which other domains it depends on or affects, and how.>
- ...

## Gotchas

<Non-obvious constraints an agent would get wrong without being told.>
- ...
```

## Writing rules

- **Present tense:** "Sessions expire after 30 minutes", not "We added session expiry".
- **Current state only:** delete rules that no longer hold. No changelog, no "previously".
- **Summarize, don't list features:** if the file starts reading like a feature log, condense it.
- **Real paths only:** every path in "Where it lives" must exist on `main` after the merge.
- **Link specs sparingly:** point to `../specs/<slug>.md` only where the deep design matters.
