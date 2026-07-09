---
name: specify
description: SDD phase 1 — write a feature spec (WHAT and WHY, no tech). Produces spec.md in the feature's specs directory.
---

# Specify (SDD Phase 1)

Writes the feature specification. Spec is **product-level**: it captures what the feature does and why, not how it's built. A non-engineer should be able to read it.

## Inputs

- Feature description (from user; ask if unclear).
- Target specs directory (see `sdd` skill for resolution rules — read repo's `SDD.md` if present, else default to `specs/`).

## Output: `spec.md`

Use this template:

```markdown
# <Feature Name>

## Summary
<2-3 sentences: what this is and the user value.>

## Motivation
<Why now? What problem does it solve? Cite the trigger — incident, user request, strategic goal.>

## User Stories
- As a <role>, I want <action>, so that <outcome>.
- ...

## Acceptance Criteria
- [ ] <Observable, testable behavior>
- [ ] ...

## In Scope
- <bullet>

## Out of Scope
- <bullet — explicit non-goals>

## Open Questions
- <unknowns the user/PM needs to resolve before plan phase>
```

## Rules

- **No technology choices.** No file paths, no class names, no libraries, no DB schema. Save those for `plan.md`.
- **Be specific about behavior.** "Fast" → "p95 < 200ms". "Easy to use" → drop it or define it.
- **Surface ambiguity** as Open Questions rather than guessing. The user should resolve them before planning.
- Keep it short. A good spec for a medium feature is under 1 page.

## After writing

1. Print the file path. **Do not paste the file contents into chat** — the user will open it in their editor.
2. List Open Questions (if any) inline in chat — these are the only thing the user needs to see immediately, since they block the `plan` phase.
3. Say: "Spec written. Review it in your editor and tell me when you're ready to proceed to `plan` (or want to discuss changes)."
4. **Stop. Do not run `plan` until the user explicitly says to proceed.** The user may edit `spec.md` directly — that's expected. If they ask you to change something, edit the file in place; don't rewrite from scratch.
