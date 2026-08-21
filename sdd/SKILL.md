---
name: sdd
description: Run the full Spec-Driven Development flow (specify → plan → tasks → implement) for a feature or refactor. Use this for any non-trivial change.
---

# Spec-Driven Development (SDD)

Orchestrates the full 4-phase SDD flow. Phases 1–3 run **back-to-back in a single
pass** — spec, plan, and tasks are written without stopping between them. There is
exactly **one review gate**, after `tasks.md`. Phase 4 then runs to completion.

## Phases

1. **specify** — Write `spec.md` (WHAT and WHY, no tech)
2. **plan** — Write `plan.md` (HOW: architecture, files, APIs)
3. **tasks** — Write `tasks.md` (numbered, testable checklist)
4. **implement** — Execute tasks one at a time

## Where artifacts live

Look up the spec directory in this order:

1. **Project SDD config**: if the repo root contains an `SDD.md` file, read it for routing rules (e.g., per-component specs dirs).
2. **Default**: top-level `specs/` directory in the repo root.

Each feature gets a subdirectory named `YYYY-MM-DD-kebab-case-name/`, where the date is today's date in the user's local timezone (e.g., `2026-05-12-user-auth`, `2026-05-13-rate-limit`). If a directory with that exact name already exists (same date + same kebab name), append `-2`, `-3`, etc. to disambiguate.

## Flow

1. Confirm with the user the **one-line feature description** if not already clear from context.
2. Determine the target specs directory (read project `SDD.md` if present).
3. Allocate the `YYYY-MM-DD-feature-name/` directory (today's date).
4. **Run `specify` → `plan` → `tasks` in one continuous pass.** Each phase prints its
   path and rolls straight into the next. Do **not** stop for approval between them.
5. **STOP at the review gate.** With all three artifacts written, hand off: print the
   three paths, summarize in a few bullets (task/group count, live open questions,
   decisions you made unilaterally), and wait.
6. Wait for the user to say "proceed" (or equivalent: "go", "next", "looks good"). Then
   **run `implement`** → execute every group to completion, then push, ensure a PR
   exists, and **mark it ready for review**. That flip is part of `implement`'s
   *Finishing* step, not something to wait for a separate request on; the only reason
   the flow ends with a draft PR is a spec acceptance criterion that doesn't hold.

## Review model

The user reviews artifacts **in their editor**, not in chat.

- **One gate, after `tasks.md`.** Phases 1–3 are a single pass; the artifacts are
  cheap to regenerate and easier to judge together than one at a time.
- Print each path as you go. Do not paste file contents back into chat.
- **Wait for explicit go-ahead** before starting `implement`. Don't infer approval
  from silence or from a tangentially-related message.
- If the user asks to change something, **edit the file in place** (use Edit tool),
  don't rewrite. The user may also edit the file directly — re-read it before the
  next phase if they say they did. When a change to an upstream artifact invalidates
  a downstream one, propagate it forward and stop at the gate again.

**What may interrupt the pass.** The pass is not a promise never to ask a question —
it's a promise not to stop for *approval*. If a genuine fork appears (an ambiguity
where two readings produce materially different plans), ask it with
`AskUserQuestion`, fold the answer in, and continue the pass. Anything you can settle
with a defensible assumption, settle it and flag it at the gate.

If the user wants to skip a phase (e.g., spec is trivial), let them.

## When to use SDD vs. just doing it

Use SDD when the work is:
- A new feature touching multiple files / modules
- A refactor with non-obvious blast radius
- Anything where the user might want to review the approach before code is written

Skip SDD for:
- Single-file edits, typos, renames
- Bug fixes with an obvious root cause
- Tasks the user has already specified in detail

When in doubt, ask the user: "this looks non-trivial — want me to run SDD on it, or just go?"
