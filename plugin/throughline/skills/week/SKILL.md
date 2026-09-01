---
name: week
description: Produce the weekly review — what was captured, what drafts await promotion with a keep or drop recommendation for each, what the auditor flagged, and which work happened that produced no knowledge at all. Use when the user runs /throughline:week or asks for their weekly Throughline review.
argument-hint: "[--since 7d]"
---

# Throughline · week

Ten minutes of hygiene that keeps the system trustworthy. **This is the one ritual the product asks
of the user** — everything else is automatic, so this needs to be worth the ten minutes.

## Preconditions

**Resolve the vault** — `$THROUGHLINE_VAULT` → `vault:` in `./.throughline` → `~/Throughline`.
**Resolve the project slug** from `./.throughline`; if absent, say so and stop.

**The markers must exist.** `<vault>/00 Command/Today.md` must contain both
`<!-- tl:week:start -->` and `<!-- tl:week:end -->`. If either is missing, **stop and report it.**
Do not guess where to write, and do not append to the end of the file — Today.md holds a live Bases
embed and the user's own scratch notes, and overwriting them is unrecoverable.

Window: last 7 days, or `--since`.

## Steps

1. **What was captured.** Notes with `created` in the window, grouped by type. Count and list.

2. **What awaits promotion.** Every note tagged `#tl/draft`, oldest first. **For each, give a
   one-line keep-or-drop recommendation** with a reason — that recommendation is what turns a list
   into a ten-second decision, and it is the difference between a queue that gets cleared and one
   that grows. Judge on: does this say something a future session would be worse off not knowing?

3. **What the auditor flagged.** Notes tagged `#tl/needs-review`, plus anything now past its
   `expires`, plus active Decisions whose `last_verified` is more than 90 days old.

4. **The git-vs-notes gap — the most useful part.** Compare git activity in the window against notes
   created:
   - files and areas with substantial commit activity that produced **no** notes at all
   - Components whose `paths` were touched repeatedly with nothing captured
   - branches merged with no Decision, Gotcha, or debrief

   State this as an observation, never an accusation. Most of it is legitimately unremarkable work.
   The purpose is to surface the case where something genuinely hard was solved and nothing was kept.
   If the repo has no git, skip this section and say why.

5. **Write it into Today.md, between the markers only.** Replace everything between
   `<!-- tl:week:start -->` and `<!-- tl:week:end -->`, keeping both marker lines in place.

   **Everything outside the markers must be byte-identical afterwards** — the frontmatter, the
   dashboard links, the `![[Sessions.base#With open threads]]` embed, and the user's scratch list.
   Do not touch `updated:` in the frontmatter; Today.md is a hub, not a knowledge note.

## Output contract

Written between the markers, and summarised in the conversation:

```markdown
### Week of <YYYY-MM-DD>

**Captured** — 3 decisions, 1 gotcha, 2 components

**Awaiting promotion** (4)
- [[G-0007 ...]] — keep. Cost 3h, will recur.
- [[D-0011 ...]] — drop. Restates [[D-0004 ...]]; no new constraint.

**Flagged**
- [[C-0003 ...]] — `paths` no longer matches the code
- [[G-0002 ...]] — expired 2026-07-01, still open

**Work that produced no knowledge**
- `src/billing/` — 14 commits, no notes. Worth a look.
```

Keep the whole thing **under a ten-minute read.** If a section is empty, say so in one line rather
than padding. A quiet week is a legitimate result and should read as one — do not manufacture
content to fill the template.

Close by pointing at `00 Command/Needs Review.base`, which is where the promotions actually happen.

## Constraints

- **Write only between the markers.** Never anywhere else in Today.md, never any other file.
- **Never promote a draft.** This skill recommends; the human decides and acts. Removing a
  `tl/draft` tag is never this skill's job.
- Never modify, tag, or delete any knowledge note. This skill is read-only outside its own section.
- Never create notes.
- Never manufacture findings for a quiet week.
