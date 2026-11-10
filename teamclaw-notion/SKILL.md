---
name: teamclaw-notion
description: The user's Teamclaw Notion workspace (Work / Projects / Teamclaw) is the source of truth for the BCN / Avernet BCS work — designs, decisions, open issues, active tasks, deep dives, onboarding notes. Use this whenever work touches BCN, BCS, the cluster executor plane, the gateway tier, executors, Redis keys/streams/Lua, buckets/leases, A2A dispatch, or the user's onboarding; whenever you need design context you don't have; whenever a discussion produces a decision, an open question, a task or a write-up worth keeping; and whenever the user says "check my comments", "the Notion page", "Teamclaw", or "update the docs".
---

# Teamclaw Notion workspace

The user tracks the BCN / gateway-tier work in Notion, not in PRs. Read it for
context; propose additions to it; keep it consistent. Requires the Notion MCP tools
(`notion-fetch`, `notion-search`, `notion-get-comments`, `notion-create-pages`,
`notion-update-page`, `notion-create-comment`, `notion-create-attachment`).

## 1. Map

Root page **Teamclaw**: `4149d01c2459801f952eeefe05de782a`

| Section | Data source (use as `data_source_id`) | Holds |
| --- | --- | --- |
| Onboarding Docs | `8ef9d01c-2459-8200-aee1-0762b3936dd5` | Project context & reading map; BCS onboarding notes (Rust for a Java dev, build/run) |
| Deep Dives | `1369d01c-2459-83bc-b15d-0701bb396678` | Designs and technical write-ups |
| Discussion Sessions | `b409d01c-2459-8311-8c44-87babaae9fa2` | Decision logs |
| Active Tasks | `c629d01c-2459-8315-a3f5-875eca3488d2` | Milestones / work items |
| Issues | `afa9d01c-2459-8210-8daf-077f4a5a439d` | Open decisions, risks, tech debt |

Key pages:

| Page | Id |
| --- | --- |
| Gateway tier — design | `4149d01c24598101a589e9a9a1d10644` |
| Redis key & operation design (gateway + executor) | `4149d01c245981109473cb856176dbef` |
| Gateway tier — implementation plan | `4149d01c245981fc903ddeee4421df82` |
| Cluster executor plane — decisions so far | `4149d01c24598116a0c4d6171d15f3ef` |
| Teamclaw / BCN — project context & reading map | `4149d01c24598140b7ddf190dfd1febd` |
| BCS onboarding notes | `4139d01c245981bf8f5cf7d9f0e8ae42` |

"Project Context & Architecture" (`3d09d01c245982ceb5bd019d9087baec`) holds leftover
content from another project; do not treat it as Teamclaw context.

Property values that fit (the databases were cloned from an RL template; leave a
field empty rather than pick an option that doesn't fit):
- Deep Dives: `Status` Draft / In Progress / Complete; `Topic` Architecture (usually).
- Discussion Sessions: `Status` Draft / Discussed / Action Items / Resolved; `Topic` Architecture; `Tags` ["Architecture"].
- Active Tasks: title prop is `Notes`; `Status` Not Started / WIP / Done / Blocked / Deprioritized; `Priority` P1–P3; `Type` WIP task / Feature / Bug Fix / Chore.
- Issues: `Status` Proposed / Backlog / Planning / Ready / Done; `Priority` Critical / High / Medium / Low; `Category` Architecture, Performance, Security, Testing, …
- Onboarding Docs: `Category` Getting Started / Code Development / Navigation; `Status` Published / Draft / Needs Update; `Order` number.

The ids above can go stale. If a fetch fails, `notion-search` for the title, use
what you find, and tell the user the skill needs updating.

## 2. Reading (do this freely, no need to ask)

- Before answering a design question or touching related code, fetch the relevant
  page. **Notion beats memory and beats PR #3609** (that spec is frozen and outdated).
- The decisions page is the quickest summary of what is settled; Issues are what is
  still open. Check both before proposing something that may already be decided.
- If the code or a newer user statement contradicts a page, say so and offer to fix
  the page.

## 3. Writing — propose first, then ask

Never create a page or change a page's substance on your own initiative. When
something worth keeping comes up (a decision, an open question, a design, a task, a
research result, a how-to), propose it in one short block:

> **Notion?** I'd add *<title>* to **<section>** (<props>), covering <one line>. Also
> update <page> §<n>. Want me to?

Then wait. Pick the section:
- settled decision → add a row to the decisions page (and close/update the matching Issue);
- open question or risk → Issues;
- unit of work → Active Tasks;
- design or analysis → Deep Dives;
- explanation for onboarding → Onboarding Docs.

Allowed without asking: replying to the user's comment threads when they ask you to
address comments, and edits they explicitly requested.

## 4. Addressing the user's comments

When asked to "check / address my comments":
1. `notion-get-comments` on every page in the map that might have new threads
   (design, Redis, plan, decisions at least). Unresolved only.
2. Reply in **each** thread (`notion-create-comment` with `discussion_id`) with the
   answer and where the page now says it. Never resolve threads — the user does.
3. Then edit the page. **Prefer `update_content` (targeted) over `replace_content`**:
   rewriting blocks detaches inline comment threads. Use a full replace only when the
   structure really changes, and warn the user that threads will detach.
4. Propagate: if a decision changed, update the decisions page, the Redis page, the
   implementation plan, the affected Issues and Active Tasks in the same pass.
5. A thread with no text context is anchored on a diagram; ask which section.

## 5. House style the user asked for

- Define before use: every page opens with a glossary. Colour legend on the design
  page — red = our terms, blue = A2A standard terms, orange = Redis keys/commands,
  purple = stream entry kinds, gray = example values. Every coloured term must be in
  the glossary.
- Say whether a term is A2A-standard, web-standard (e.g. SSE `Last-Event-ID`) or ours.
- No abbreviations in names (`session`, not `sess`). Redis items always show a key
  example **and** a value example.
- Describe only the current design. Don't mention superseded or never-adopted
  designs; the decisions page keeps history (struck through).
- Every data-flow section gets an SVG diagram; flows are drawn as numbered steps in
  the real order, with every Redis node named ("Redis (bucket inbox of b)").
- Link Issues and pages as Notion mentions, never plain text.
- MVP first: defer non-essential hardening (e.g. SSRF policy) to a "Later" section.

## 6. Diagrams

Write SVGs (light background, the palette above), render once with Playwright to
check for overlaps, then upload with `notion-create-attachment` passing the SVG as
`content` (direct uploads to api.notion.com are blocked from cloud sessions) and
place with `<image source="file-upload://…"/>`. Strip unused `<marker>` defs to keep
the payload small.

## 7. Current design in one paragraph (orientation only — the pages are authoritative)

Two tiers. Gateways hold PA connections (A2A over JSON-RPC+SSE and gRPC, no
WebSocket) and call supply agents; executors own buckets
(`xxh3(session_id) mod 4096`) and run orchestration. Everything between them is Redis:
bucket inbox `bcn:inbox:{b}` (gateway → owner), gateway inbox `bcn:gw:<id>`
(owner → gateway), executor inbox `bcn:exec:<id>` (hand-off), session stream
`bcn:session:{b}:<S>` (owner → gateways), lease `bcn:lease:{b}` with fencing token
and `inbox_cursor`. Every owner write is one fenced Lua script. Roster heartbeat every
0.5 s, TTL 1.5 s; failover < 3 s.
