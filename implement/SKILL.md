---
name: implement
description: SDD phase 4 — execute an approved tasks.md group by group, running all the way through every task without stopping, then mark the PR ready for review.
---

# Implement (SDD Phase 4)

Executes the task list. Tasks are bundled into **groups** (defined at the end of
`tasks.md`); groups run in order, tasks within a group run in order, and `tasks.md`
status markers update as you go.

**Run to the end.** Once the user has approved `tasks.md`, this phase goes all the
way through the final group without stopping for check-ins. The approval at the
review gate covers the whole implementation, not one group of it. Only a genuine
blocker stops you (see *When to actually stop*) — observations, concerns, and nits
get noted and carried to the final report, not turned into a pause.

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
   4. **Move straight to the next group.** No check-in, no "want me to
      continue?", no waiting. Things worth the user's attention — a
      changed public API, something you couldn't fully verify, a
      design-level concern the review raised — go into a running
      **notes list** that you report at the end. They do not stop the run.
3. **Final verification group:** walk through every spec acceptance
   criterion and confirm it holds. Report any that don't.
4. **Ship it** — see *Finishing* below.

## Finishing

The moment the last group is done and the acceptance criteria are verified,
without waiting to be asked:

1. **Push** the branch — `git push -u origin <branch>`, retrying on network
   errors with exponential backoff (2s, 4s, 8s, 16s).
2. **Ensure a PR exists** for the branch. An already-open PR counts, draft or
   not — don't open a second one. If there is none, open one (as a draft),
   filling in the repo's PR template if it has one.
3. **Mark it ready for review immediately.** Use whichever mechanism this
   session actually has:
   - `mcp__github__update_pull_request` with `draft: false` — Claude Code on
     the web / remote. Load it via `ToolSearch` if it isn't already available.
   - `gh pr ready <number>` via Bash — local sessions with the `gh` CLI.
   - Neither available → say so explicitly in the final report and name the PR
     that needs a manual flip. The step never disappears silently.

   This is part of finishing, not a separate request: don't leave a completed
   implementation sitting in draft, and don't ask permission to flip it. A
   standing "open pull requests as drafts" convention — the web harness has
   one — governs how a PR is *created*, not whether a finished implementation
   stays draft. It does not override this step.
4. **Confirm the flip landed.** Re-read the PR (`mcp__github__pull_request_read`
   (`get`), or `gh pr view <number> --json isDraft`) and check that it is no
   longer a draft. If it still is, retry once; if it still is after that, report
   it as a blocker rather than assuming it worked.
5. **Report once**, briefly: groups completed, acceptance criteria status, the
   PR link, whether it is ready or draft, and the notes list you accumulated
   during the run.
6. Hand off to the `peerreview` skill to drive the review rounds. Marking ready
   for review is what opens round 1, so the handoff is immediate.

If the acceptance criteria *don't* all hold, still push and open the PR, but say
plainly which criteria fail and leave the PR in draft — ready-for-review means
the implementation is actually complete. Failing acceptance criteria are the
**only** reason this phase ends with a draft PR.

## Rules

- **Don't silently re-plan.** If a task can't be done as written, mark it `[!]` blocked, explain why, and ask the user — don't improvise a new design.
- **Don't batch task completions.** Update `tasks.md` immediately after each task finishes. Lets the user pick up where you left off if interrupted.
- **One commit per task.** Each task lands as its own commit, including
  the `tasks.md` bookkeeping line for that task. Code-review fixes are
  additional commits inside the same group.
- **Run tests after each task** when feasible; don't save all testing for the end.
- **If you find a bug or rough edge unrelated to the current task,** note it but don't fix it inline — that's scope creep. Add it to the notes list.
- **Don't ask for permission you already have.** The approved `tasks.md` authorizes
  every task in it, and finishing authorizes the push, the PR, and marking it ready.

## When to actually stop

Three things — and only these — interrupt the run:

- **A task can't be done as written.** Mark it `[!]` blocked, and if the rest of the
  group and later groups don't depend on it, keep going and report the blocker at
  the end. If work downstream genuinely can't proceed without it, stop there.
- **The spec or plan is wrong** (see below).
- **The user says stop.**

Everything else — a surprising API change, a partially-verified behavior, a
design-level concern from code review — is a note, not a pause.

## When the spec/plan is wrong

If implementation reveals that spec or plan is incorrect — not merely thin, but
actually contradicted by the code:
1. Stop.
2. Tell the user what you found and why the existing artifacts don't fit.
3. Suggest going back to update `spec.md` / `plan.md` / `tasks.md` before continuing.

Do not patch around a wrong plan with code-level workarounds.
