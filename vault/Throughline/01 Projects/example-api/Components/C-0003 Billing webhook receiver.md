---
type: component
status: active
project: example-api
created: 2026-06-09
updated: 2026-07-02
source: human
paths: ["src/billing/webhooks/", "src/billing/settlement.ts"]
external_deps: ["Stripe API 2026-04-01"]
stability: changing
owner: amir
last_verified: 2026-07-02
tags:
  - tl/needs-review
  - tl/external
---

> [!warning] Flagged by the auditor
> `paths` still lists `src/billing/settlement.ts`, but settlement logic was split across three files in July and this note has not been updated. It is tagged `#tl/needs-review` so it shows up in the hygiene queue. **This is deliberate — it is what a real flagged note looks like on first open.**

## What it does

Receives Stripe webhook events, verifies their signature, and turns them into settlement records. It is the only part of the system that accepts unauthenticated inbound traffic, which makes it the riskiest surface in the codebase.

## Key files

- `src/billing/webhooks/handler.ts` — signature verification and dispatch.
- `src/billing/webhooks/events/` — one module per event type we handle. Unknown events are logged and dropped, never errored.
- `src/billing/settlement.ts` — maps a verified event onto a [[T Settlement window]].

## How it fits

**Upstream:** Stripe, directly. Does *not* pass through [[C-0001 Auth subsystem]] — it authenticates by signature instead, which is why the invariant below matters so much.

**Downstream:** the settlement repository, per [[D-0001 Route all database access through the repository layer]]. Deliberately does **not** touch `legacy/billing/` — see [[D-0003 Freeze the legacy billing module]].

## Invariants

- **Every handler is idempotent, keyed on the Stripe event id.** Stripe retries. This is not defensive programming, it is a hard requirement — see [[G-0002 Stripe webhook retries deliver duplicate events after a 500]].
- **Signature verification happens before the body is parsed.** Parsing first and verifying second is a well-known way to accept a forged event.
- **A settlement record is never mutated, only superseded.** The settlement ledger is append-only for audit reasons.
- **Unknown event types are dropped, never errored.** Returning a 500 for an event we do not handle causes Stripe to retry it forever.

## Known gotchas
<!-- populated from backlinks — do not edit -->

## Decisions governing this
<!-- populated from backlinks — do not edit -->
