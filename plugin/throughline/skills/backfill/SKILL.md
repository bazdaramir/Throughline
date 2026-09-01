---
name: backfill
description: Reconstruct Decision, Gotcha, and Component notes from the repository's git history so the vault starts full instead of empty. Use when the user runs /throughline:backfill or wants to populate a new vault from existing project history.
argument-hint: "[--since 90d]"
---

# Throughline · backfill

**This skill is a thin wrapper.** Check preconditions, then delegate to the `throughline-backfiller`
subagent. Do not generate notes inline.

**Why delegation is not optional:** processing hundreds of commits and diffs would consume the main
context window entirely. This runs once, expensively, in an isolated subagent.

**This is the onboarding moment.** The user runs it, opens Obsidian, and the vault is about their
real code. Everything about how it behaves shapes whether they trust the system.

## Preconditions

1. **Resolve the vault**: `$THROUGHLINE_VAULT` → `vault:` in `./.throughline` → `~/Throughline`.
   Unresolvable → say so plainly and stop. Never guess.
2. **Resolve the project slug** from `project:` in `./.throughline`. Absent → tell the user to run
   `/throughline:init` and stop.
3. Confirm this is a git repository with history. If not, say so — there is nothing to read.

## Steps

1. Resolve vault, project, and git as above.
2. State the window (default `--since 90d`) and the commit count in scope, so the user knows the
   size of what they asked for.
3. **Delegate to the `throughline-backfiller` subagent**, passing vault path, project slug, and
   window.
4. Relay its progress and its closing confidence summary **verbatim** — especially which notes it
   is confident about and which it is inferring. Do not smooth that over.
5. Point the user at `00 Command/Needs Review.base`: everything produced is `#tl/draft` and needs a
   promotion pass.

## Output contract

Live progress while it runs, then the notes created grouped by type, then the honest confidence
summary. Target is **10–20 notes on a 100+ commit repo** — fewer means thresholds were too strict,
more than 25 means it generated noise.

Everything it writes carries:

```yaml
source: backfill
confidence: low        # on Decisions
tags:
  - tl/draft
```

`source: backfill` is deliberately distinct from `source: agent` — it marks knowledge inferred from
artefacts rather than distilled from an observed session, and the user must be able to tell those
apart at a glance.

## Constraints

- **Never backfill inline.** If the subagent is unavailable, say so and stop.
- Create only — never edit, rename, delete, or overwrite an existing note.
- Never remove a `tl/draft` tag. Never promote.
- Never write to the user's repository. Git is read.
- Never present inferred history as certain. The confidence summary is mandatory.
