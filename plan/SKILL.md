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

Use this template. Anywhere the plan touches concrete code — schemas,
signatures, payloads, edits, config — **write it as a fenced code block**, not
as a sentence describing it. A reader should be able to skim the blocks and see
the shape of the change. The examples below are illustrative; use the project's
actual language, dialect, and file layout.

````markdown
# Plan: <Feature Name>

## Approach
<2-4 sentences describing the chosen design at a high level.>

## Affected Components
<For each module/service touched, list it with a one-line role. Cite paths.>
- `src/backend/...` — <role>
- `src/engine/...` — <role>

## Data Model Changes
<Show the actual DDL / migration / model definition. None? Write "None." and move on.>

```sql
-- migrations/0042_widget_owner.sql
ALTER TABLE widget
  ADD COLUMN owner_id  UUID        NOT NULL REFERENCES account(id),
  ADD COLUMN archived_at TIMESTAMPTZ NULL;

CREATE INDEX widget_owner_idx ON widget (owner_id);
```

<Backfill/ordering notes go in prose under the block, one or two lines.>

## API / Interface Changes
<One block per endpoint / RPC / message type / exported function. Show the real
signature and the request/response shape. Mark anything breaking.>

```python
# src/backend/api/widgets.py  (new route)
@router.post("/widgets/{widget_id}/archive")
async def archive_widget(widget_id: UUID, user: User) -> ArchiveResponse: ...
```

```jsonc
// POST /widgets/{widget_id}/archive → 200
{ "widget_id": "…", "archived_at": "2026-01-01T00:00:00Z" }
// 403 when the caller is neither owner nor admin
```

```diff
# BREAKING — src/engine/client.py
- def list_widgets(self) -> list[Widget]:
+ def list_widgets(self, include_archived: bool = False) -> list[Widget]:
```

## Key Files & Functions
<Every file created or modified, with file:line. Sketch each change as code —
a diff block for edits, a signature stub for new code. Sketches, not
implementations: signatures and the load-bearing lines only.>

```diff
# src/backend/foo.py:120 — handle_request
-    if widget.owner_id != user.id:
+    if widget.owner_id != user.id and not user.is_admin:
         raise Forbidden()
```

```python
# src/backend/bar.py (new)
def archive_widget(session: Session, widget_id: UUID) -> Widget: ...
```

## Dependencies
<New packages, version bumps, internal services. Show the manifest edit.>

```diff
# pyproject.toml
  dependencies = [
+   "pydantic>=2.7",
  ]
```

## Risks & Mitigations
- **Risk:** <thing that could go wrong>
  **Mitigation:** <how we handle it>

## Alternatives Considered
<Other approaches and why we didn't pick them. Keep terse.>

## Rollout
<Feature flag? Migration order? Backwards-compat concerns? Show the flag or the
command sequence when there is one.>

```bash
# migration must land before code that reads owner_id
alembic upgrade head && deploy api
```

## Test Strategy
<Unit / integration / manual. Name the actual test files and cases.>

```python
# tests/api/test_widgets.py
def test_archive_widget_requires_owner_or_admin(): ...
def test_list_widgets_excludes_archived_by_default(): ...
```
````

## Rules

- **Show the code, don't describe it.** Schemas, signatures, payloads, edits,
  and config changes go in fenced code blocks. "Add an `owner_id` column and an
  index" is worse than four lines of DDL — the block is unambiguous and faster
  to read. Prose is for *why*; code blocks are for *what*.
- **Sketches, not implementations.** Keep each block to roughly 15 lines:
  signatures, type definitions, and the load-bearing lines. Function bodies,
  error handling, and boilerplate belong in the implementation, not the plan.
- **Tag every fence** with a real language (`sql`, `python`, `ts`, `jsonc`,
  `diff`, `bash`, …) and start it with a comment naming the file — ideally
  `# path/to/file.py:120` — so the block is self-locating.
- **Use `diff` for modifications, plain language fences for new code.** A diff
  makes the before/after obvious; a diff of a whole new file is just noise.
- **Match the project.** Mirror the codebase's real language, schema dialect,
  naming, and file layout. The template's Python/SQL examples are illustrative
  only.
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
