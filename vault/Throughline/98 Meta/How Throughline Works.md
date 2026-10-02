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

Two things in the brief are not knowledge, and it says so:

- **A count of waiting drafts.** If unreviewed drafts would reach the brief once promoted — a draft Component that owns a changed file, or a draft Decision or Gotcha that governs one — the brief says how many, by kind: *"2 unreviewed draft(s) touch these files: 1 component, 1 gotcha — not shown, not trusted."* It never prints a title, an id, or a word of a draft. A count says that something waits; it says nothing about whether to believe it. Without it, a brief that is quiet because everything relevant is still a draft looks exactly like an empty vault.
- **The last session's open threads.** The one channel into a future session that no human reviews: the distiller writes them from a transcript that may contain web pages or tool output. So they are labelled as unreviewed, capped at three, cut at 140 characters, and stripped of control characters.

---

## The draft quarantine

Every automatically-generated knowledge note is tagged `#tl/draft` and carries `source: agent` (or `source: backfill`).

**Drafts are excluded from every retrieval path.** The brief skips them. `/brief`, `/why`, and `/spec` skip them. No skill, hook, or subagent may ever remove that tag — promotion is a human action, always, without exception.

This one rule is what makes automatic capture safe. Without it, auto-capture poisons the vault within a month, and a memory you cannot trust is worse than no memory at all, because it is confidently wrong.

**What counts as a draft** is decided the way Obsidian decides it: `tl/draft` in the frontmatter `tags` in any form — `- tl/draft`, `- "tl/draft"`, `[tl/draft]` — or `#tl/draft` written inline in the body, outside code. The Needs Review queue and the SessionStart brief apply the same test, so they cannot disagree about a note.

### Promoting a draft

Read it first, and fix what is wrong — once promoted it is yours, so correct the title, `affects`,
`paths`, or the reasoning before you accept it. Then promote it with the command, or by hand.

**With `tl-promote`** (a shell command that ships in the plugin's `bin/`; `/throughline:help` prints
its exact path). Run it from the repository root, in your own terminal:

- `tl-promote` lists the drafts, Components first, and flags any that depend on a draft Component.
- `tl-promote D-0004` shows **exactly** what promoting that note would change — a few lines — and
  asks. Nothing is written until you answer `y` at a terminal, so an agent can preview a promotion
  but cannot make one.
- `tl-promote --review` walks the whole queue the same way: Components first, one prompt each.

Each promotion is one line in the [[Promotion Log]]: when, which note, who, and what the draft
claimed to be when you accepted it.

**By hand, in Obsidian** — the command does exactly these steps, no more:

1. **Remove `tl/draft`** from `tags`.
2. **Set `last_verified`** to today. Without it the note stays in [[Needs Review.base|Needs Review]], which keeps every unverified note older than seven days.
3. **Promote what it hangs off.** A Decision or Gotcha reaches the brief only *through* a promoted Component in its `affects` — if that Component is still a draft, promote it first.
4. **If it has `supersedes`,** open the Decision it names and set `status: superseded` and `superseded_by` to this note. Change nothing else on it. Skills deliberately leave this to you: until you promote the replacement, the old decision is still the one in force.

Leave `source` as it is. Provenance records who *wrote* the note, not who approved it.

**Dropping a draft** is deleting the note. The command never deletes.

The command refuses, and changes nothing, when it cannot be exact: a `#tl/draft` written in the body
(Obsidian counts it as a tag, so removing the frontmatter tag would not promote the note), a tags
list spread over several lines, or any edit that would still leave the note a draft. Those need a
hand edit.

Obsidian's Properties panel rewrites list properties as one item per line when you edit them. That is fine — retrieval reads both shapes.

---

## `source` is the trust property

Every note records where it came from: `human`, `agent`, or `backfill`. You can always see, at a glance, whether a claim came from a person or was inferred by a model. Systems that blur this line lose the user eventually.

---

## The one ritual

Open [[Needs Review.base|Needs Review]] once a week and clear it. Promote the drafts worth keeping, delete the rest, re-verify what the auditor flagged. With the plugin, run `/throughline:week` first — it gives every draft a keep-or-drop call and points at the auditor's findings, which live in [[Audit Log]] rather than in the queue, because the auditor never tags a note.

That is the entire maintenance burden. One ritual is sustainable. Five are not.

---

## What the human layer is for

Be clear-eyed about this: **the machine side of Throughline would work identically against a plain folder of Markdown files.** Claude Code does not need Obsidian to read this vault.

Obsidian's contribution is entirely on the human side — browsing, linking, dashboards, graph, review, curation. That is not a weakness, it is the correct division of labour:

> **Claude Code writes and retrieves. Obsidian is how you read, trust, and correct.**

Open [[C-0002 Session store]] and look at its backlinks pane. That pane *is* the answer to "everything we know about the session store" — every decision, gotcha, spec, and session that touched it, assembled without anyone writing a query. That is the argument for Obsidian over a `docs/` folder, in one gesture.

---

See also: [[Note Types Reference]] · [[Workflow Cheat Sheet]] · [[Vault Changelog]] · [[Home]]
