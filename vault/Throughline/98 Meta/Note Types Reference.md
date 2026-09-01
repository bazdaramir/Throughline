---
type: hub
status: active
project: global
created: 2026-08-13
updated: 2026-08-13
source: human
---

# Note Types Reference

Seven types. The number was reached by subtraction — the first draft had fourteen. Every type that could not justify a distinct retrieval query was merged or cut.

## The universal frontmatter contract

Every Throughline note has these six properties. No type may omit them.

```yaml
type:        # decision | component | gotcha | session | spec | term | practice
status:      # type-specific lifecycle
project:     # project slug, or "global" for Domain and Practices
created:     # YYYY-MM-DD
updated:     # YYYY-MM-DD
source:      # human | agent | backfill   ← provenance. Non-negotiable.
```

One further optional property is universal: `last_verified: YYYY-MM-DD` — the last time a human confirmed this note still matches reality. The hygiene queue uses it.

## The seven types at a glance

| Type | Folder | Filename | Answers | Required beyond the contract |
|---|---|---|---|---|
| **decision** | `01 Projects/<p>/Decisions/` | `D-0012 {Imperative title}.md` | *Why is it like this?* | `affects` (≥1 component), `confidence` |
| **component** | `01 Projects/<p>/Components/` | `C-0003 {Noun phrase}.md` | *What is X and what must stay true?* | `paths` |
| **gotcha** | `01 Projects/<p>/Gotchas/` | `G-0021 {Symptom, not cause}.md` | *What will bite me here?* | `affects`, `severity`, `cost` |
| **session** | `01 Projects/<p>/Sessions/` | `2026-08-13-1430.md` | *What happened, and what did we already try?* | `duration`, `branch`, `touched` |
| **spec** | `01 Projects/<p>/Specs/` | `S-0007 {Imperative title}.md` | *What are we about to build?* | `affects` |
| **term** | `02 Domain/` | `T {Term}.md` | *What does this word mean here?* | — (`project: global`) |
| **practice** | `03 Practices/` | `P {Rule as imperative}.md` | *What rule did we earn?* | `scope` (`project: global`) |

`hub` also appears as a `type` on [[Home]], [[Today]], and each `_Project` note. It is navigation scaffolding, not a knowledge type, and is excluded from the hygiene queue.

## Status lifecycles

| Type | Valid `status` values |
|---|---|
| decision | `proposed` · `active` · `superseded` · `reversed` |
| component | `active` · `deprecated` · `planned` |
| gotcha | `open` · `resolved` · `wontfix` |
| session | `complete` · `abandoned` |
| spec | `draft` · `ready` · `building` · `done` · `abandoned` |
| term | `active` · `deprecated` |
| practice | `active` · `deprecated` |

## The properties that carry the most weight

- **`paths` (Component)** — the join key between the vault and the codebase. The SessionStart brief maps changed files to notes through it. Set it precisely or retrieval degrades from targeted to keyword-ish. No property in the system carries more architectural weight.
- **`reversal_cost` (Decision)** — what makes a decision a guardrail rather than an archive entry. An agent about to contradict an `irreversible` decision should stop and ask; one contradicting a `low`-cost decision should just mention it.
- **`cost` (Gotcha)** — how long it cost the *first* time. The emotional anchor, and the field an ROI dashboard sums.
- **`origin` (Practice)** — the gotchas that taught you the rule. Turns an opinion into a law.
- **`open_threads` (Session)** — feeds directly into the next session's brief. This is what produces *"last time you left the migration script untested against the prod schema."*

## Mandatory link directions

Enforced by templates, checked by the auditor. Each direction is a query something actually runs.

| From | To | Property |
|---|---|---|
| Decision | Component(s) it affects | `affects` |
| Gotcha | Component it lives in | `affects` |
| Spec | Decisions it depends on | `depends_on` |
| Session | Everything it touched | `touched` |
| Component | Parent component | `parent` |
| Decision | Decision it supersedes | `supersedes` |
| Any note | Domain terms used | inline wikilink, e.g. [[T Settlement window]] |

**Forbidden:** linking every note to everything (a dense graph carries no information); links purely for "relatedness"; and **links from permanent notes back to Sessions** — sessions are evidence, links flow *from* sessions *to* knowledge, never back.

## The eleven tags. There will not be more.

`#tl/needs-review` · `#tl/unverified` · `#tl/hot` · `#tl/security` · `#tl/performance` · `#tl/breaking` · `#tl/external` · `#tl/temporary` · `#tl/painful` · `#tl/inherited` · `#tl/draft`

Frontmatter properties do the heavy lifting. Tags exist only for cross-cutting dimensions a property cannot express. Tag proliferation is the classic way PKM systems die: every extra tag is a decision at capture time, and decisions at capture time are what kill capture.

**`#tl/draft` is the most important tag in the system** — it is the quarantine that makes automatic capture safe.

## Naming rules

- IDs are **monotonic per project**, never reused, never renumbered — they get cited in commit messages.
- Titles are **human sentences**, not slugs.
- **Gotcha titles describe the symptom, not the cause.** You search for what you observe.
- Decision and Spec titles are **imperative**.
- **Never rename a note.** Supersede it. Renames break wikilinks and destroy the audit trail.

## Types deliberately cut

Meeting note · Task/TODO · Bug report · Person · Daily note · Idea/inbox · Reference · Question · Metric · Runbook.

Each failed one test: *name a query, run by a skill or a hook, that this type uniquely answers.* If no automated part of the system needs it, it is a folder you maintain for free and abandon within a month.

---

See also: [[How Throughline Works]] · [[Workflow Cheat Sheet]] · [[Home]]
