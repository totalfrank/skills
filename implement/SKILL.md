---
name: implement
description: SDD phase 4 — execute tasks from an approved tasks.md, one at a time, marking progress as you go.
---

# Implement (SDD Phase 4)

Executes the task list. Works through tasks in order, updating `tasks.md` status markers as you go. Stops to check in at meaningful boundaries.

## Inputs

- Existing `spec.md`, `plan.md`, `tasks.md` in the feature directory.

## Flow

1. **Read all three artifacts.** Spec for acceptance criteria, plan for
   design, tasks for what to do. Locate the `## Groups` section at the
   end of `tasks.md` — that's the execution unit. If `tasks.md` has no
   groups (legacy format), fall back to one-task-per-group.
2. **For each group, in order:**
   1. Announce the group and the tasks it contains.
   2. **For each task in the group, in order:**
      1. Mark it `[~]` in-progress in `tasks.md`.
      2. Do the work — edit code, add tests, run tests.
      3. Verify the task's "Done when" conditions actually hold.
      4. Mark it `[x]` done in `tasks.md` **immediately** — don't batch.
      5. **Commit the task.** One commit per task. The bookkeeping
         update to `tasks.md` belongs in the same commit as the work
         it describes. Use the task's title as the commit subject.
      6. Briefly report: "Task N done — <what changed in 1 line>."
   3. **Group review loop** (skip if the group has no code changes —
      e.g. pure docs):
      1. Run the `code-reviewer` subagent against the group's diff
         (the commits that landed for this group's tasks).
      2. If the review surfaces real bugs, fix them in-tree. Land the
         fixes as additional commits inside this group (mention the
         task they patch in the commit message), then re-run review.
      3. Repeat until a round comes back with no new bugs flagged.
         Cosmetic / style nits that aren't bugs don't block — note them
         but don't loop forever.
      4. Report the final review verdict in one line.
   4. **Pause for user check-in** before starting the next group if:
      - The group changed public APIs or data models.
      - You couldn't fully verify something.
      - The review loop surfaced design-level concerns (not just bugs).
      - The plan needs to change.
3. **Final verification group:** walk through every spec acceptance
   criterion and confirm it holds. Report any that don't.

## Rules

- **Don't silently re-plan.** If a task can't be done as written, mark it `[!]` blocked, explain why, and ask the user — don't improvise a new design.
- **Don't batch task completions.** Update `tasks.md` immediately after each task finishes. Lets the user pick up where you left off if interrupted.
- **One commit per task.** Each task lands as its own commit, including
  the `tasks.md` bookkeeping line for that task. Code-review fixes are
  additional commits inside the same group.
- **Run tests after each task** when feasible; don't save all testing for the end.
- **If you find a bug or rough edge unrelated to the current task,** note it but don't fix it inline — that's scope creep. Surface it to the user.

## When the spec/plan is wrong

If implementation reveals that spec or plan is incorrect:
1. Stop.
2. Tell the user what you found and why the existing artifacts don't fit.
3. Suggest going back to update `spec.md` / `plan.md` / `tasks.md` before continuing.

Do not patch around a wrong plan with code-level workarounds.
