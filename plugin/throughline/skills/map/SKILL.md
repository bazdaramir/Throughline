---
name: map
description: Read a directory and produce a Component note mapping the subsystem — what it does, its key files, and above all its invariants. Use when the user runs /throughline:map, or asks to map, document, or capture the structure of a subsystem.
argument-hint: "<directory-or-glob>"
---

# Throughline · map

Produces one Component note. **`paths` is the join key the entire retrieval layer depends on** —
the SessionStart brief and `/throughline:brief` both map changed files to notes through it. Get
`paths` wrong and retrieval silently degrades from targeted to noise.

## Preconditions

**Resolve the vault** — `$THROUGHLINE_VAULT` → `vault:` in `./.throughline` → `~/Throughline`.
Never guess. If unresolvable, say so and stop.

**Resolve the project slug** from `project:` in `./.throughline`. If absent:
> This repo is not initialised. Run `/throughline:init` first.
Never invent a slug.

**A target is required.** With no argument, ask which directory to map. Do not map the whole repo.

## Steps

1. **Check for overlap first.** Read every `<vault>/01 Projects/<slug>/Components/*.md` and collect
   their `paths`. If an existing Component already covers the target, **stop** — report which one,
   and offer to refine that note by hand. Never write a second Component over the same ground, and
   never edit the existing one.

2. **Read the target.** Directory tree, then the files themselves — entry points, exports, imports,
   types, tests. Follow imports outward far enough to describe what calls this and what it calls.

3. **Set `paths` precisely.** This is the highest-consequence field in the note.
   - List the specific files and subdirectories this component actually owns.
   - **Never** set a path so broad it captures unrelated code. `paths: ["src/"]` makes every file
     match every Component and destroys retrieval for the whole project.
   - Before writing, check: *would a change to any file under these paths genuinely be a change to
     this component?* If not, narrow it.
   - Directories end with `/`. Files carry their extension.

4. **Place it in the hierarchy.** If an existing Component's `paths` is a strict prefix of this
   one's, set `parent` to it. Otherwise omit `parent`.

5. **Allocate the ID** — scan `Components/` for the highest `C-NNNN`, add one, zero-pad to four.
   Never reuse an ID, even if a note was deleted. Filename: `C-NNNN <Noun phrase>.md`. The title is
   a human noun phrase (`C-0003 Session store`), not a slug.

6. **Write the note** using `90 Templates/Component.md` — the five `##` headings in template order.

7. **Large directories:** if the target contains several genuinely independent subsystems, write the
   parent Component and *propose* children in your output. Do not write more than one note per run.

## Output contract

```yaml
---
type: component
status: active
project: <slug>
created: <today YYYY-MM-DD>
updated: <today YYYY-MM-DD>
source: agent
paths: ["src/auth/session.ts", "src/auth/store/"]
tags:
  - tl/draft
---
```

Optional, only when genuinely known: `parent` (quoted wikilink), `owner`, `external_deps`,
`stability: stable | changing | volatile`. **Never set `last_verified`** — that records human
verification, and setting it hides the note from the hygiene queue.

Sections, in this exact order:

- `## What it does` — **two sentences maximum.** Ruthlessly.
- `## Key files` — paths with a one-line purpose each.
- `## How it fits` — upstream and downstream. What calls it, what it calls.
- `## Invariants` — **the critical section.** Things that must remain true and are *not* enforced by
  the type system: "session IDs are never reused", "scope checks never happen in route handlers".
  These are exactly what an agent violates cheerfully and a human catches three weeks later in
  production. If you cannot infer any honestly, write fewer — never pad.
- `## Known gotchas` — leave **empty**, carrying only `<!-- populated from backlinks — do not edit -->`
- `## Decisions governing this` — leave **empty**, same comment.

Report: the ID, the path, the `paths` value you chose, and the invariants — surfaced explicitly for
correction, because **the user's correction of the invariants is the value being captured.** Close
by noting the note is `#tl/draft` and invisible to retrieval until promoted.

## Constraints

- One note per invocation.
- **Never populate the two backlink sections.** Anything maintained by hand goes stale; anything
  derived from links cannot.
- Never edit or overwrite an existing Component.
- Never write `last_verified`.
- Never remove a `tl/draft` tag — promotion is a human action only.
- Do not describe *how* the code works line by line. This is a map, not documentation. Two sentences
  of purpose, the files, the shape, and the invariants.
