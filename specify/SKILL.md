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
2. List Open Questions (if any) inline in chat, in one terse block.
3. **Continue straight into `plan`. Do not stop for review here.** Spec, plan, and
   tasks are written back-to-back in one pass; the single review gate is *after*
   `tasks.md` exists. Say one line — "Spec written → `<path>`. Moving on to plan."
   — and keep going.

The only thing that pauses this phase is an Open Question that is genuinely
**blocking** — one where two plausible answers produce materially different plans,
so writing `plan.md` without it would mean guessing at the design. In that case ask
via `AskUserQuestion` (not a free-form stop), get the answer, fold it into
`spec.md`, and continue. Questions that only affect details the plan can defer stay
listed as Open Questions and do not stop the pass.

If the user later asks for changes, edit `spec.md` in place; don't rewrite from
scratch. They may also edit it directly — re-read it before using it.
