---
name: debrief
description: After shipping a change, review the diff against its spec and propose notes for what deserves keeping — especially decisions made during implementation that deviated from the plan. Use when the user runs /throughline:debrief, has just merged or finished a piece of work, or wants to capture what a change taught them.
argument-hint: "[PR-url]"
---

# Throughline · debrief

Turns finished work into retained knowledge, without a documentation ritual.

**The highest-value target is deviation from the spec.** Plans get written down. Deviations from
plans almost never do — and the deviation is usually where the real reasoning lives.

## Preconditions

**Resolve the vault** — `$THROUGHLINE_VAULT` → `vault:` in `./.throughline` → `~/Throughline`.
**Resolve the project slug** from `./.throughline`; if absent, say so and stop.

## Steps

1. **Establish the change.** With a PR URL, read that PR's diff, commits, and discussion. Without
   one, use the merge-base diff for the current branch, or the last few commits if the branch is
   already merged. State which you used.

2. **Find the related Spec**, if any, by matching branch name, commit messages, and touched paths
   against `Specs/*.md` and their `affects`.

3. **Compare implementation to intent. This is the core of the skill.**
   - What did the implementation do that the spec did not call for?
   - What did the spec call for that was not done, or was done differently?
   - **Every such deviation is a candidate Decision** — a real choice was made under pressure and
     the reasoning for it exists nowhere else.

4. **Scan for the other three signals:**
   - **Surprises** — something that behaved unexpectedly → a Gotcha, titled by symptom.
   - **New subsystems** — a directory that no Component covers → a Component.
   - **New invariants** — something that must now stay true → belongs in an existing Component's
     `## Invariants`. **Propose this in your output; do not edit the Component yourself.**

5. **Apply the significance bar — actively.** Most changes produce nothing worth keeping. A refactor
   with no trade-off, a dependency bump, a formatting pass, a straightforward feature built exactly
   as specced: **all of these correctly produce zero notes.**

   **Silence is a correct and common outcome.** If you find nothing that meets the bar, say so and
   write nothing. Do not manufacture significance to justify the invocation — an auto-capture system
   that finds profundity in every change is producing noise, and noise is what kills these products.

6. **Write the notes that survive the bar**, each following its type's contract exactly — the same
   rules as `decide`, `gotcha`, and `map`, including symptom-titling for Gotchas, the mandatory
   alternatives table for Decisions, and precise `paths` for Components.

## Output contract

Zero to three notes. More than three from one change almost always means the bar slipped.

Every note: correct template, correct section order, `source: agent`, `tags: [tl/draft]`, resolved
`affects`, and **never** `last_verified`.

Report as a short list — for each: the type, the ID, the path, and one line on why it met the bar.
If a deviation from the spec drove it, say which. Then the quarantine notice.

If nothing met the bar, say exactly that in one sentence and stop.

## Constraints

- **Never modify the Spec.** Do not set it to `done`, do not edit its sections. Whether the work is
  finished is a human judgement.
- **Never edit a Component to add an invariant.** Propose it in the output; the user applies it.
- Never remove a `tl/draft` tag.
- Never write more than one note about the same fact — check existing notes for duplicates first,
  and for a repeated Gotcha symptom increment `recurrence` rather than duplicating.
- This is the **manually invoked** sibling of the SessionEnd distillation (`bin/tl-session-end`). Do
  not attempt any automatic, background, or session-lifecycle behaviour.
