---
name: tasks
description: SDD phase 3 — break an approved plan into a numbered, testable task list. Produces tasks.md alongside plan.md.
---

# Tasks (SDD Phase 3)

Decomposes the approved `plan.md` into discrete, ordered tasks. Each task should be small enough to land as a single coherent commit and have a clear "done" condition.

## Inputs

- Existing `spec.md` and `plan.md` in the feature directory.

## Output: `tasks.md`

Use this template:

```markdown
# Tasks: <Feature Name>

> Status legend: `[ ]` todo · `[~]` in-progress · `[x]` done · `[!]` blocked

## Task 1: <Short imperative title>
- **Goal:** <one sentence>
- **Files:** `path/to/file.py`, `path/to/other.py`
- **Done when:**
  - [ ] <observable condition>
  - [ ] <test passes / endpoint returns / etc.>
- **Depends on:** —

## Task 2: <...>
- **Goal:** ...
- **Files:** ...
- **Done when:** ...
- **Depends on:** Task 1

...

## Task N: Tests & Verification
- **Goal:** Ensure feature meets spec acceptance criteria.
- **Done when:** all spec acceptance criteria check off.

---

## Groups

> Groups bundle tasks into end-to-end units of work. `implement` executes
> one group at a time and runs code review on each group before moving on.
> List groups in execution order; each task appears in exactly one group.

- **Group A — <name>:** Tasks 1, 2
  - Theme: <one-line "what does this group accomplish on its own">
- **Group B — <name>:** Tasks 3, 4, 5
  - Theme: ...
- **Group C — Verification:** Task N
  - Theme: Final spec acceptance check.
```

## Rules

- **Each task is independently reviewable.** A reviewer should be able to look at one task's diff and judge it without needing the others.
- **Order by dependency.** Earlier tasks should not depend on later ones.
- **Right-size:** ~30 min to ~2 hrs of work each. If a task is bigger, split it. If smaller, merge it.
- **Tie back to the plan.** Every file in `plan.md`'s "Key Files" should appear in at least one task. Every spec acceptance criterion should be covered by at least one task's "Done when".
- **Tests are tasks too** — either folded into the task that adds the code, or as a dedicated final verification task.
- **Don't pre-commit to implementation details** that weren't in the plan; if you find new ones while breaking down, flag them rather than silently adding.
- **Group tasks into end-to-end units.** A group should land as a coherent
  slice of the feature — typically 1–4 tasks that share a theme and can be
  reviewed together. Define groups in a `## Groups` section at the END of
  `tasks.md`. Every task must belong to exactly one group, and groups must
  be listed in execution order.

## After writing

1. Print the file path. **Do not paste the file contents into chat.**
2. Inline in chat, give just: total task count, and (if any) a one-line note on plan gaps the breakdown surfaced.
3. Say: "Tasks written. Review them in your editor and tell me when you're ready to proceed to `implement`."
4. **Stop. Do not start `implement` until the user explicitly says to proceed.** If they ask for changes, edit `tasks.md` in place.
