---
name: sdd
description: Run the full Spec-Driven Development flow (specify → plan → tasks → implement) for a feature or refactor. Use this for any non-trivial change.
---

# Spec-Driven Development (SDD)

Orchestrates the full 4-phase SDD flow. Each phase produces a reviewable artifact; the user approves before moving to the next phase.

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
4. **Run `specify`** → produce `spec.md` → print path → **STOP**.
5. Wait for the user to say "proceed" (or equivalent: "go", "next", "looks good", etc.). Then **run `plan`** → produce `plan.md` → print path → **STOP**.
6. Wait for "proceed". Then **run `tasks`** → produce `tasks.md` → print path → **STOP**.
7. Wait for "proceed". Then **run `implement`** → execute tasks, checking off as you go.

## Review model

The user reviews artifacts **in their editor**, not in chat. After writing each file:
- Print the path. Do not paste the file contents back into chat.
- Inline in chat, surface only what blocks the next phase (open questions, plan-revealed spec gaps, etc.) — keep this terse, a few bullets max.
- **Wait for explicit go-ahead** before advancing. Don't infer approval from silence or from a tangentially-related message.
- If the user asks to change something, **edit the file in place** (use Edit tool), don't rewrite. The user may also edit the file directly — re-read it before the next phase if they say they did.

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
