---
name: help
description: List the Throughline commands, grouped by lifecycle stage, with the six that cover most use and the one weekly ritual. Use when the user asks what Throughline can do, which command to run, or types /throughline:help.
argument-hint: "[skill-name]"
---

# Throughline · help

**This skill does no reasoning and reads no files.** Print the reference below and stop. Do not
inspect the vault, do not resolve the project, do not personalise the output. It must be instant.

## Steps

1. If an argument was given and it names one of the twelve skills, print only that skill's row plus
   its one-line detail from the table below.
2. If an argument was given and it does not name a skill, say so and print the full list.
3. Otherwise print everything below the line.

---

## Output contract — print verbatim

**Throughline — engineering memory for AI-assisted development.**

**Start with these six. They cover ninety percent of use.**

| Command | What it does |
|---|---|
| `/throughline:brief <area>` | Load what you need to know before touching an area of code |
| `/throughline:why <question>` | "Why is it like this?" — walks the full decision history |
| `/throughline:decide [what]` | Record an architectural choice and its rejected alternatives |
| `/throughline:gotcha [symptom]` | Record a landmine so it never costs you the same hours twice |
| `/throughline:map <dir>` | Map a subsystem into a Component note |
| `/throughline:audit` | Check the vault still matches reality |

**Everything, by lifecycle stage**

*Setup*
- `/throughline:init [slug]` — make this repo Throughline-aware. Run once per repo.
- `/throughline:backfill [--since 90d]` — build the vault from your git history.

*Capture — getting knowledge in*
- `/throughline:decide [what]` — a Decision note. The `Alternatives considered` table is the point.
- `/throughline:gotcha [symptom]` — a Gotcha note, titled by what you *observe*.
- `/throughline:map <dir>` — a Component note. Sets `paths`, which everything else joins on.
- `/throughline:spec <objective>` — a Spec with your real constraints and landmines pre-loaded.

*Retrieval — getting the right knowledge back*
- `/throughline:brief <area>` — a small targeted slice. Never a vault dump.
- `/throughline:why <question>` — decision lookup, including whether it has been superseded.

*Review and hygiene*
- `/throughline:debrief [PR-url]` — after shipping, capture what deviated from the plan.
- `/throughline:week` — the weekly review, including work that produced no knowledge at all.
- `/throughline:audit` — eight checks of the vault against the codebase. Flags, never modifies.

**The one ritual:** open `00 Command/Needs Review.base` in Obsidian once a week and clear it.
Everything else is automatic. One ritual is sustainable; five are not.

**Everything a skill writes is quarantined** with `#tl/draft` and is invisible to `/throughline:brief`
and `/throughline:why` until *you* promote it by deleting that tag. No skill can promote its own
output. That is what makes automatic capture safe.

**Two things happen without you asking.**

- **At session start**, a brief is assembled from the files you have been changing — the decisions
  and open landmines that govern them. It is built by path matching, not by a model, so it is
  instant and *cannot invent a decision you never wrote*.
- **At session end**, if the session was substantial, the log is distilled in the background. Your
  session ends immediately; the note appears a minute later. Most sessions correctly produce
  nothing but the log.

Both are silent when there is nothing to say, and neither can promote anything.

Full workflow reference: `98 Meta/Workflow Cheat Sheet.md` in the vault.

## Constraints

- No file reads. No vault access. No model reasoning. If this skill is slow, it is wrong.
- Do not invent commands. There are exactly twelve.
- Do not describe the automatic layer as doing more than it does. Two hooks, both silent by default,
  neither able to promote a note.
