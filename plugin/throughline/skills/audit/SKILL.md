---
name: audit
description: Audit the vault against the real codebase and report knowledge that has decayed - dead paths, drifted decisions, expired gotchas, orphans, contradictions. Use when the user runs /throughline:audit or asks whether the vault still matches reality.
argument-hint: "[project-slug]"
---

# Throughline · audit

**This skill is a thin wrapper.** Check preconditions, then delegate to the `throughline-auditor`
subagent. Do not perform the audit inline.

**Why delegation is not optional:** the audit reads the entire vault plus large parts of the
codebase. Running that in the main session would consume the context window — which is the exact
problem this product exists to prevent. Doing it in an isolated subagent is that principle applied
to the product's own implementation.

## Preconditions

1. **Resolve the vault**: `$THROUGHLINE_VAULT` → `vault:` in `./.throughline` → `~/Throughline`.
   Unresolvable → say so plainly and stop. Never guess.
2. **Resolve the project slug** from `project:` in `./.throughline`, or from the argument. Absent →
   tell the user to run `/throughline:init` and stop.

## Steps

1. Resolve vault and project as above.
2. Tell the user, in one line, what is about to be audited and that it will take a few minutes.
3. **Delegate to the `throughline-auditor` subagent**, passing the resolved vault path and project
   slug.
4. Relay its report. Do not re-analyse, re-rank, or soften the findings.
5. Point the user at `00 Command/Needs Review.base` and `98 Meta/Audit Log.md`.

## Output contract

The auditor's eight checks in order, each finding with a specific file reference and a recommended
action, followed by its honest confidence summary. Then the two pointers above.

## Constraints

- **Never audit inline.** If the subagent is unavailable, say so and stop — do not partially
  implement the checks in the main session.
- **The auditor is read-only on knowledge notes and so are you.** It flags; it never modifies. It
  does not apply tags — not even `#tl/needs-review` — and neither do you. If a note warrants a tag,
  the recommendation goes to the human, who applies it.
- The only file written by this workflow is `98 Meta/Audit Log.md`, appended by the subagent.
- Never promote a draft. Never edit, rename, or delete a note.
