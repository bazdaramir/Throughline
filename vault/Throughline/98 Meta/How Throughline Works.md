---
type: hub
status: active
project: global
created: 2026-08-13
updated: 2026-08-13
source: human
---

# How Throughline Works

Four layers. Each has one job. The boundaries between them are the whole design.

```
LAYER 4 · HYGIENE      What fights decay
                       Auditor subagent · staleness sweep · reconciliation
                       · supersession chains · confidence decay
                              ↓ corrects
LAYER 3 · RETRIEVAL    What gets into context
                       SessionStart brief · /brief · /why · CLAUDE.md as a thin router
                              ↓ reads from
LAYER 2 · STRUCTURE    Where knowledge lives          ← YOU ARE HERE (this vault)
                       7 note types · frontmatter · wikilinks · Bases · graph
                              ↑ written by
LAYER 1 · CAPTURE      How knowledge gets in
                       SessionEnd hook · /decide · /gotcha · /map · /backfill
```

**The design principle that governs all four:**

> The human should never be asked to write documentation, and the agent should never be given the whole vault.

Violate the first and the system dies of neglect in two weeks. Violate the second and you have reinvented context rot with extra steps.

---

## The three verbs, in order of how much they matter

**1. CAPTURE — automatically, from the work itself.**
Notes are a by-product of engineering, never a separate chore. A session ends; a draft note appears. You did nothing.

**2. RETRIEVE — selectively, at the moment of need.**
*This is where every competitor stops short.* At session start, a deterministic script maps the files you just touched to the notes that govern them and injects **at most 400 tokens**. Not the vault. Not a summary of the vault. The slice that matters, and nothing else.

**3. MAINTAIN — actively, because knowledge decays.**
*This is where nobody even starts.* An auditor reads the vault against the real codebase and flags decisions the code has quietly contradicted. It **flags and never modifies** — you decide.

---

## Why the brief is a script and not a model call

The SessionStart brief is assembled by deterministic path matching: `git` tells it which files changed, the `paths:` property on Component notes maps those files to notes, and `affects:` links fan out to the Decisions and Gotchas that govern them.

No model runs. That means the brief is instant, costs nothing, and — the part that matters — **cannot invent a decision you never made.** It can only tell you things you actually wrote down. A model-generated brief could not make that promise.

---

## The draft quarantine

Every automatically-generated note is tagged `#tl/draft` and carries `source: agent`.

**Drafts are excluded from every retrieval path.** The brief skips them. `/brief` skips them. No skill, hook, or subagent may ever remove that tag — promotion is a human action, always, without exception.

This one rule is what makes automatic capture safe. Without it, auto-capture poisons the vault within a month, and a memory you cannot trust is worse than no memory at all, because it is confidently wrong.

---

## `source` is the trust property

Every note records where it came from: `human`, `agent`, or `backfill`. You can always see, at a glance, whether a claim came from a person or was inferred by a model. Systems that blur this line lose the user eventually.

---

## The one ritual

Open [[Needs Review.base|Needs Review]] once a week and clear it. Promote the drafts worth keeping, delete the rest, re-verify what the auditor flagged.

That is the entire maintenance burden. One ritual is sustainable. Five are not.

---

## What the human layer is for

Be clear-eyed about this: **the machine side of Throughline would work identically against a plain folder of Markdown files.** Claude Code does not need Obsidian to read this vault.

Obsidian's contribution is entirely on the human side — browsing, linking, dashboards, graph, review, curation. That is not a weakness, it is the correct division of labour:

> **Claude Code writes and retrieves. Obsidian is how you read, trust, and correct.**

Open [[C-0002 Session store]] and look at its backlinks pane. That pane *is* the answer to "everything we know about the session store" — every decision, gotcha, spec, and session that touched it, assembled without anyone writing a query. That is the argument for Obsidian over a `docs/` folder, in one gesture.

---

See also: [[Note Types Reference]] · [[Workflow Cheat Sheet]] · [[Vault Changelog]] · [[Home]]
