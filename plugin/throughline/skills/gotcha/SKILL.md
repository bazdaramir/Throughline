---
name: gotcha
description: Record a landmine — something surprising, non-obvious, or costly — titled by its symptom so it can be found again, including the tempting wrong fix. Use when the user runs /throughline:gotcha, or has just solved something painful, surprising, or hard to debug.
argument-hint: "[symptom]"
---

# Throughline · gotcha

The highest value-per-word note type in the system and the cheapest to create. Run it immediately
after solving something painful, while the failed attempts are still in the conversation.

## Preconditions

**Resolve the vault** — `$THROUGHLINE_VAULT` → `vault:` in `./.throughline` → `~/Throughline`.
Never guess. **Resolve the project slug** from `./.throughline`; if absent, tell the user to run
`/throughline:init` and stop. **At least one Component must exist** for `affects`; if none, say so
and point at `/throughline:map`.

## The titling rule — the thing this skill most often gets wrong

**Title by the SYMPTOM, never by the cause.**

Future-you searches for what they *observe*, not for what they later learned was happening.

- ✅ `G-0021 Redis TTL silently resets on SET`
- ✅ `G-0034 ECONNRESET from the pool after exactly 30 seconds idle`
- ❌ `G-0021 Connection pool misconfiguration`
- ❌ `G-0034 Incorrect TTL handling`

Titling by cause is the single most common way gotcha logs become unsearchable, and the damage is
invisible for months — until the day someone greps for the error string and finds nothing.

**Before writing, check your own title:** would someone hitting this for the first time — who does
not yet know the cause — search for these words? If not, retitle it.

## Steps

1. **Dedupe against the symptom.** Read every `Gotchas/*.md`. If one describes the same *observable
   symptom* — even if you now understand the cause differently — **do not create a second note.**
   Instead, on that existing note:
   - increment `recurrence` (absent → set it to `2`; this is its second occurrence)
   - set `updated` to today
   - change nothing else

   Then report the increment and stop. A recurring landmine is far more informative than two notes
   about one landmine.

2. **Resolve `affects`** by matching the failure against Component `paths`. Never invent one.

3. **Estimate `cost`** — how long it cost **the first time**, in hours, from the length of the
   conversation and the number of failed attempts. Say in your output that it is an estimate. This
   field is the emotional anchor and the number the ROI view sums; a rough honest figure beats none.

4. **Set `severity`** by blast radius, not by annoyance: `critical` for data loss, security, or
   silent wrongness; `high` for a broken workflow with no obvious cause; `medium` and `low` below.

5. **Allocate the ID** — highest `G-NNNN` plus one, zero-padded. Never reuse.

6. **Write the note** from `90 Templates/Gotcha.md`.

## Output contract

```yaml
---
type: gotcha
status: open
project: <slug>
created: <today YYYY-MM-DD>
updated: <today YYYY-MM-DD>
source: agent
affects: ["[[C-0002 Session store]]"]
severity: critical | high | medium | low
cost: "3h"
tags:
  - tl/draft
---
```

Optional, when true: `expires` (when the upstream fix is expected — only with a real basis),
`resolved_by`, `external: "Redis 7.2"`, `recurrence`. **Never set `last_verified`.**

Sections, in this exact order:

- `## Symptom` — **what you actually observe**, written to be searched. Exact error strings, exact
  timings, exact log lines. This section is an index entry, not prose.
- `## Cause` — why it happens.
- `## Workaround` — what to do. Copy-pasteable if possible.
- `## Do not` — **mandatory. The tempting wrong fix.** Often more valuable than the right fix,
  because the wrong fix is the first thing both a human and an agent will try. If the conversation
  contains an approach that looked right and failed, it belongs here.

Report the ID, the path, the estimated `cost` flagged as an estimate, and the quarantine notice.

**If `recurrence` is now ≥ 2**, add: this has bitten more than once and is a candidate for promotion
to a Practice — a landmine you have stepped on twice is a missing convention. **Note it; do not
create the Practice.** That is a separate workflow and a human judgement.

## Constraints

- **The only existing note you may touch is a duplicate Gotcha**, and only `recurrence` and `updated`.
- Never title by cause. Self-check before writing.
- Never omit `## Do not`.
- Never invent a Component for `affects`.
- Never remove a `tl/draft` tag. Never create a Practice note.
- One Gotcha per invocation.
