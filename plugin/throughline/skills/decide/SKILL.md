---
name: decide
description: Record an architectural decision and the alternatives that were rejected, detecting whether it supersedes an earlier decision. Use when the user runs /throughline:decide, or says they have decided / chosen / settled on an approach worth recording.
argument-hint: "[what was decided]"
---

# Throughline · decide

Captures a real architectural choice while the reasoning is still fresh. **The
`Alternatives considered` table is the point of the note** — it is what separates a Decision from a
changelog entry, and it is the thing that stops the same debate being had again in four months.

## Preconditions

**Resolve the vault** — `$THROUGHLINE_VAULT` → `vault:` in `./.throughline` → `~/Throughline`.
Never guess. **Resolve the project slug** from `./.throughline`; if absent, tell the user to run
`/throughline:init` and stop.

**At least one Component must exist.** `affects` requires one and the auditor enforces it. If
`Components/` is empty, stop and say:
> No components mapped yet, so this decision has nothing to attach to. Run `/throughline:map <dir>` first.

## What counts as a decision

Write a note when a choice **constrains future work** — it forecloses options, or a future
maintainer would be surprised by it and change it back.

Do **not** write one for: a bug fix, a routine refactor, a naming choice, a library version bump
with no trade-off, or anything you would not defend in review. If the user invoked this skill for
something that is not a decision, say so plainly and offer `/throughline:gotcha` instead. Not
everything deserves a note, and a vault full of trivia is worse than a small one.

## Steps

1. **Reconstruct the decision** from the conversation and the argument. What was chosen, what
   constraint forced it, what was rejected and why.

2. **Resolve `affects`** — read `Components/*.md` and match the decision against their `paths` and
   subject matter. At least one, as many as genuinely apply. Never invent a Component.

3. **Check for duplication and supersession.** Read every existing Decision whose `affects`
   intersects yours.
   - **Same choice already recorded and still current** → do not write a duplicate. Report the
     existing note and stop.
   - **This reverses or replaces an earlier decision** → this is a supersession. Set `supersedes` on
     the new note, and on the predecessor set `status: superseded` and
     `superseded_by: "[[D-NNNN <its title>]]"`. **Change nothing else on the predecessor** — not its
     body, not its `updated`, not its reasoning. It is a historical record.
   - **Two or more plausible predecessors** → this is the one time you may ask a question. Ask which.

4. **Allocate the ID** — highest `D-NNNN` in `Decisions/` plus one, zero-padded to four. Never
   reuse. Filename `D-NNNN <Imperative title>.md`; the title is imperative and human —
   `D-0012 Freeze the legacy billing module`.

5. **Write the note** from `90 Templates/Decision.md`.

## Output contract

```yaml
---
type: decision
status: active
project: <slug>
created: <today YYYY-MM-DD>
updated: <today YYYY-MM-DD>
source: agent
affects: ["[[C-0002 Session store]]"]
confidence: high | medium | low
tags:
  - tl/draft
---
```

`status: active` because the user has just made this decision. Optional, when true: `supersedes`,
`reversal_cost`, `evidence` (`["commit:a3f9c21"]` or a quoted wikilink to a Session).
**Never set `last_verified`.**

Set **`confidence` honestly** — `medium` or `low` when the reasoning rests on numbers you have not
measured or assumptions you have not tested, and say which in `Context`.

Set **`reversal_cost` honestly**. It is what makes the note a guardrail rather than an archive
entry: an agent about to contradict an `irreversible` decision should stop and ask, one contradicting
a `low` decision should just mention it.

Sections, in this exact order:

- `## Decision` — one sentence, imperative.
- `## Context` — what made this necessary. The constraint, not the history.
- `## Alternatives considered` — **mandatory table.** Reconstruct the rejected options from the
  conversation; they are usually visible in it. If genuinely no alternative was weighed, write one
  row saying so — **never fabricate plausible-sounding alternatives.**
- `## Consequences` — what this makes easy, what it makes hard, what it forecloses.
- `## Revisit when` — **a checkable trigger, never a date.** "When we exceed 10k concurrent
  sessions" is checkable by an agent reading metrics. "In six months" is not.

Report: the ID, the path, **the alternatives table inline** so the user can correct it in thirty
seconds, any supersession you applied, and — if `reversal_cost` is `irreversible` — say so loudly.
Close with the quarantine notice: the note is `#tl/draft` and `/throughline:why` will not surface it
until the tag is removed by hand.

## Constraints

- **The only existing note you may touch is one predecessor Decision**, and only its `status` and
  `superseded_by` fields.
- Never delete, never rename, never overwrite. Supersede instead — renames break wikilinks and
  destroy the audit trail.
- Never remove a `tl/draft` tag.
- Never write a Decision with an empty `affects`.
- Never invent alternatives, and never turn `Revisit when` into a date.
- One Decision per invocation.
