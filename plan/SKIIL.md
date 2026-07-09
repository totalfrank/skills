---
name: plan
description: SDD phase 2 — write a technical plan based on an approved spec. Produces plan.md alongside spec.md.
---

# Plan (SDD Phase 2)

Translates the approved `spec.md` into a concrete technical approach. This is where architectural decisions, affected code, and trade-offs get pinned down — **before** any code is written.

## Inputs

- An existing `spec.md` in the feature directory.
- The current state of the codebase (read it; don't guess).

## Output: `plan.md`

Use this template:

```markdown
# Plan: <Feature Name>

## Approach
<2-4 sentences describing the chosen design at a high level.>

## Affected Components
<For each module/service touched, list it with a one-line role. Cite paths.>
- `src/backend/...` — <role>
- `src/engine/...` — <role>

## Data Model Changes
<New tables/columns, migrations, schema changes. None? Say so.>

## API / Interface Changes
<New endpoints, RPCs, message types, function signatures. Include request/response shapes.>

## Key Files & Functions
<Specific files that will be created or modified, with file:line where relevant.>
- `src/backend/foo.py:120` — extend `handle_request` to ...
- `src/backend/bar.py` (new) — ...

## Dependencies
<New packages, version bumps, internal services.>

## Risks & Mitigations
- **Risk:** <thing that could go wrong>
  **Mitigation:** <how we handle it>

## Alternatives Considered
<Other approaches and why we didn't pick them. Keep terse.>

## Rollout
<Feature flag? Migration order? Backwards-compat concerns?>

## Test Strategy
<Unit / integration / manual. What are the must-have tests?>
```

## Rules

- **Cite file:line** for any code you reference. No hand-waving.
- **Read the code before claiming it works a certain way.** Use the graph tools (semantic_search_nodes, query_graph) for cross-references — they're faster than grep.
- **Don't smuggle in scope creep.** If the plan grows beyond the spec, flag it: either trim, or go back and update the spec.
- **State trade-offs explicitly.** Every non-trivial decision should have an "Alternatives Considered" entry.
- Keep it tight — bullets over prose.

## After writing

1. Print the file path. **Do not paste the file contents into chat.**
2. Inline in chat, briefly note (1-3 bullets max): any non-obvious risks, surprising decisions, or spec gaps the plan surfaced.
3. Say: "Plan written. Review it in your editor and tell me when you're ready to proceed to `tasks`."
4. **Stop. Do not run `tasks` until the user explicitly says to proceed.** If they ask for changes, edit `plan.md` in place.
