# Refinement checklist

Use this to drive the refinement questions. Work top to bottom and ask about anything the card
doesn't already answer. It's a prompt for questions, not a substitute for the `Meta/domains/` files,
which describe the current truth of the project.

Every question: concrete and scenario-shaped, with the realistic options (including keeping the
current behavior where one exists) and your recommendation.

> Add stack- or product-specific questions to the sections below when you set up a new project.

## 1. Scope & intent

- [ ] What does this card add or fix, in one sentence? Why does it matter to the user?
- [ ] What is explicitly **out of scope**?

## 2. User experience

- [ ] What triggers it (input, action, event, schedule)?
- [ ] What does the user see and get? What feedback confirms it worked or failed?

## 3. Domain impact (ask which apply)

- [ ] For each domain in `Meta/INDEX.md`: does this card change it, depend on it, or break an
      assumption it relies on?

## 4. Data & configuration

- [ ] Starting values for every number or limit involved.
- [ ] Where they live: config, constants, data files, admin settings.
- [ ] Does anything need migrating or backfilling?

## 5. Assets / content

- [ ] New art, copy, media or data, or reuse of existing ones?
- [ ] Placeholder acceptable for now, or final content required?

## 6. Edge cases

- [ ] Interrupted midway? Fails partway? Retried or triggered twice?
- [ ] Happening at the same time as something else?
- [ ] Invalid, missing or extreme input?

## 7. Platforms & performance

- [ ] Which platforms, devices or environments? Any differences between them?
- [ ] Anything heavy that needs a budget?

## 8. Exit bar

- [ ] Every "how should it behave when…" question has an agreed answer, written to
      `## Decisions (from refinement)`.
- [ ] Acceptance criteria are testable (Given / When / Then).
- [ ] An implementer could write the spec and build it **without** asking anything.
