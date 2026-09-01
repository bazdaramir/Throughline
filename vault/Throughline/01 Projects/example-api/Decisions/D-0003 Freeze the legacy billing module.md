---
type: decision
status: active
project: example-api
created: 2026-06-18
updated: 2026-08-01
source: human
affects: ["[[C-0003 Billing webhook receiver]]"]
confidence: high
reversal_cost: irreversible
last_verified: 2026-08-01
tags:
  - tl/inherited
---

> [!danger] `reversal_cost: irreversible`
> This is the property that turns a document into a guardrail. An agent about to modify `legacy/billing/` should **stop and ask**, not proceed and mention it. That distinction is the entire point of the field.

## Decision

`legacy/billing/` is frozen. No edits, no refactors, no lint fixes, no dependency bumps. New billing behaviour goes in `src/billing/`, and the two are bridged by a single adapter at `src/billing/legacy-bridge.ts`.

## Context

The legacy module computes historical settlements for merchants onboarded before March 2026. Its output has been reconciled against auditor-signed statements. We cannot re-derive those numbers — the inputs are gone. Any change to the module, including a formatting change that alters float evaluation order, risks producing a different number for a historical period we have already attested to.

Nobody currently at the company wrote it. That is recorded as `#tl/inherited`, not as an excuse.

## Alternatives considered

| Option | Why not |
|---|---|
| Rewrite it properly with tests | The tests would encode our *current* understanding, which is exactly what we cannot verify. Green tests would give false confidence. |
| Characterisation-test it, then refactor | Better, and genuinely tempting. Rejected because a characterisation suite only covers inputs we can still produce, and the historical inputs are precisely the ones we no longer have. |
| Delete it and recompute from the ledger | The ledger *starts* after the legacy period. There is nothing to recompute from. |
| Freeze it | Ugly, honest, and cheap. Chosen. |

## Consequences

**Easier:** a bright, greppable line. No debate about whether a given change is safe — no change is.

**Harder:** the module's dependencies are pinned forever, including one with a known CVE that is not reachable from our call path. That exception is documented in the security review, not worked around here.

**Forecloses:** repo-wide automated refactors and formatter runs. CI excludes the path explicitly; a formatter that reformats it will fail the build.

## Revisit when

The last merchant onboarded before March 2026 closes their account, at which point the historical settlements stop being legally relevant and the module can be deleted outright — not refactored.
