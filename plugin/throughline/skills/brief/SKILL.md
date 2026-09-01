---
name: brief
description: Assemble a small targeted context brief about one area of the codebase — its invariants, the decisions that govern it, and the open landmines in it — before modifying that code. Use when the user runs /throughline:brief, or is about to work on an area and wants to know what they need to know first.
argument-hint: "<area>"
---

# Throughline · brief

Answers "before I touch auth, tell me what I need to know."

**This is a targeted slice, not a vault dump.** An unbounded brief is context rot wearing the
product's own logo — it recreates the exact problem Throughline exists to solve. The budget below is
a correctness requirement, not a style preference.

## Preconditions

**Resolve the vault** — `$THROUGHLINE_VAULT` → `vault:` in `./.throughline` → `~/Throughline`.
**Resolve the project slug** from `./.throughline`; if absent, say so and stop.

An area query is required — a Component name, a path, or a concept.

## Retrieval rules — all binding

- **Budget: 1500 tokens. Hard ceiling.** Over budget → drop to titles only and tell the user to
  narrow the query. Never exceed it by "just a bit because this is all relevant."
- **Order by damage-if-unknown.** Not by recency, not alphabetically, not grouped by note type. What
  would hurt most if this agent did not know it goes first. An `irreversible` decision outranks an
  interesting one.
- **Exclude every note tagged `#tl/draft`.** Unreviewed agent output must never enter a working
  context as established fact.
- **Only `status: active` Decisions and `status: open` Gotchas.** Superseded and resolved notes are
  history; they belong to `/throughline:why`, not here.
- **Never include Session note bodies.** Read only `open_threads` from the most recent Session on
  this branch. Session logs in context is precisely how context rot gets rebuilt inside the tool
  designed to prevent it.
- **Write for an agent about to modify this code, not a human browsing.** No preamble, no narration,
  no "this document covers". Facts, constraints, hazards.

## Steps

1. **Resolve the query to Components.** Match against `paths` first (a path query is exact), then
   note titles, then subject matter. If it resolves to nothing, **say so and stop** — never widen
   the search to loosely related material.

2. **Traverse outward from those Components** — this is a link walk, not a search:
   - the Components' `## Invariants` and `## What it does`
   - Decisions whose `affects` names them, `status: active`
   - Gotchas whose `affects` names them, `status: open`
   - Practices in `03 Practices/` whose `applies_to` matches the stack
   - Domain Terms referenced by any of the above — these prevent the subtlest errors
   - `open_threads` from the newest Session touching this area

3. **Rank by damage-if-unknown**, then cut to budget from the bottom.

4. **Write the brief**, then **save a copy** to `<vault>/04 Briefs/<YYYY-MM-DD>-<slug>.md` with
   minimal frontmatter:

   ```yaml
   ---
   type: brief
   project: <slug>
   created: <today YYYY-MM-DD>
   source: agent
   ---
   ```

   `04 Briefs/` is disposable and gitignored by design. A brief is a derived artefact, and letting
   derived summaries accumulate in permanent knowledge is how memory systems poison themselves. It
   is not one of the seven note types and never becomes one.

## Output contract

Cite every source note as a wikilink so the user can jump to it. Roughly this shape:

```
◆ <area> · <project>

  INVARIANTS — must stay true
    · Session IDs are never reused. [[C-0002 Session store]]

  ACTIVE DECISIONS
    [[D-0003 Freeze the legacy billing module]]  ⚠ irreversible — do not touch legacy/
    [[D-0001 Route all database access through the repository layer]]  reversal_cost: high

  OPEN LANDMINES
    [[G-0001 Redis TTL silently resets on SET]]  cost: 3h
      → write value and expiry in one command; never SET then EXPIRE

  UNFINISHED
    ⋯ "migration script untested against prod schema"

  Saved to 04 Briefs/2026-08-16-session-store.md
```

Include the one-line workaround for each Gotcha — that is the part that actually prevents the
repeat. Omit any section that is empty rather than printing "None".

## Constraints

- **1500 tokens, hard.**
- Never include a `#tl/draft` note.
- Never include a Session note body.
- Never include superseded Decisions or resolved Gotchas.
- Never modify a knowledge note. The only file this skill writes is the disposable brief.
- Never widen a query that resolved to nothing.
- This is **not** the automatic SessionStart brief. That is a separate, deterministic, 400-token
  mechanism (`bin/tl-brief`, no model call). Do not replicate or pre-empt it.
