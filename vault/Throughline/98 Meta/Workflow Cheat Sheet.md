---
type: hub
status: active
project: global
created: 2026-08-13
updated: 2026-08-13
source: human
---

# Workflow Cheat Sheet

All 26 workflows, one line each.

**Legend** — `[CHAT]` works in any AI chat by copy-paste · `[CC]` needs Claude Code · `[HOOK]` runs automatically · `[SUB]` runs in an isolated subagent · ⭐ load-bearing · **V1** ships as a skill in the plugin.

> **Six commands cover ninety percent of use:** `/throughline:brief`, `/throughline:why`, `/throughline:decide`, `/throughline:gotcha`, `/throughline:map`, `/throughline:audit`. Do not try to learn the rest. Discover them from `/throughline:help` when you need them.

## CAPTURE — getting knowledge in without asking the human to write

| # | Workflow | Where | V1 |
|---|---|---|---|
| W1 | **Session Distillation** — a finished session becomes a Session note plus 0–3 draft notes | `[CC][HOOK]` | SessionEnd hook |
| W2 | **Decision Capture** — a real choice becomes a Decision with its alternatives table | `[CHAT][CC]` | `/decide` |
| W3 | **Gotcha Capture** — a landmine becomes a searchable, symptom-titled Gotcha | `[CHAT][CC]` | `/gotcha` |
| W4 | **Component Mapping** — a subsystem becomes a Component with `paths` and invariants | `[CC]` | `/map` |
| W5 | Domain Term Extraction — business vocabulary becomes Term notes | `[CC]` | prompt |
| W6 | **Retro-Documentation from Git History** ⭐ — a year of commits becomes 10–20 real notes | `[CC][SUB]` | `/backfill` |

## RETRIEVE — getting the right knowledge into context, and nothing else

| # | Workflow | Where | V1 |
|---|---|---|---|
| W7 | **Session Brief** ⭐ — the agent opens already knowing. ≤400 tokens, no model call | `[CC][HOOK]` | SessionStart hook |
| W8 | **Targeted Brief** — "before I touch auth, tell me what I need to know". 1500 tokens | `[CC]` | `/brief` |
| W9 | **Decision Lookup** — "why is it like this?", walking the full supersession chain | `[CHAT][CC]` | `/why` |
| W10 | Gotcha Guard — warns before you edit a mined file | `[CC][HOOK]` | **V2** |
| W11 | Area Onboarding — get oriented in an unfamiliar subsystem | `[CC][SUB]` | prompt |

## MAINTAIN — fighting decay, which is where every other system fails

| # | Workflow | Where | V1 |
|---|---|---|---|
| W12 | **Vault Audit** ⭐ — eight checks against the real codebase. **Flags, never modifies** | `[CC][SUB]` | `/audit` |
| W13 | Staleness Sweep — surface what nobody has re-verified | `[CC]` | prompt |
| W14 | Decision Reconciliation — find decisions the code has quietly contradicted | `[CC][SUB]` | prompt |
| W15 | Link Weaving — connect notes that should reference each other and don't | `[CC]` | prompt |
| W16 | Deduplication & Merge — collapse near-duplicate notes | `[CC]` | prompt |

## PLAN — using accumulated knowledge to shape future work

| # | Workflow | Where | V1 |
|---|---|---|---|
| W17 | **Grounded Spec Drafting** ⭐ — a spec with your real constraints and landmines pre-loaded | `[CC]` | `/spec` |
| W18 | Pre-Flight Conflict Check — does this plan contradict a decision? | `[CC]` | prompt |
| W19 | Risk Surface Analysis — what does this change put at risk? | `[CC]` | prompt |
| W20 | Agent-Sized Work Breakdown — split work into pieces an agent can finish | `[CC]` | prompt |

## REVIEW — turning finished work into retained knowledge

| # | Workflow | Where | V1 |
|---|---|---|---|
| W21 | **Change Debrief** — what deviated from the spec, and why | `[CC]` | `/debrief` |
| W22 | Grounded PR Description — a PR body that cites the decisions it implements | `[CC]` | prompt |
| W23 | Post-Incident Capture — incident → timeline → gotchas | `[CC]` | Incident module, **V2** |
| W24 | **Weekly Review** — the week, plus the git-vs-notes gap | `[CC]` | `/week` |

## LEARN — the vault getting smarter, not just bigger

| # | Workflow | Where | V1 |
|---|---|---|---|
| W25 | Pattern Extraction — recurring gotchas graduate into Practices | `[CC][SUB]` | prompt |
| W26 | Knowledge Gap Analysis — what does the vault not know that it should? | `[CC][SUB]` | prompt |

---

## The twelve V1 skills

`init` · `help` · `decide` · `gotcha` · `map` · `brief` · `why` · `spec` · `debrief` · `week` · `audit` · `backfill`

The other fourteen workflows ship as **documented prompts** in the Starter library — usable immediately by copy-paste, promoted to skills in V2 based on which ones people actually run. That is deliberate: V2's roadmap gets written by observing real usage instead of guessing.

> **The plugin is optional.** Everything marked **V1** is built and installable, but the vault also
> works entirely by hand — open `90 Templates/` and write a note. That is the whole system; the
> plugin only removes the typing. See [[Vault Changelog]].

---

See also: [[How Throughline Works]] · [[Note Types Reference]] · [[Home]]
