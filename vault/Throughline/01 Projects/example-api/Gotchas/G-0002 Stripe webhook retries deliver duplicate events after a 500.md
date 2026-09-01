---
type: gotcha
status: open
project: example-api
created: 2026-07-02
updated: 2026-07-02
source: human
affects: ["[[C-0003 Billing webhook receiver]]"]
severity: critical
cost: "5h"
external: "Stripe API 2026-04-01"
expires: 2026-07-01
last_verified: 2026-07-02
tags:
  - tl/painful
  - tl/external
---

> [!warning] `expires` is in the past
> This note claimed the upstream fix would land by 2026-07-01. It did not. Throughline surfaces expired notes in the hygiene queue rather than letting a stale prediction sit in the vault looking authoritative. **Either move the date or drop the field — but decide.**

## Symptom

A merchant is credited twice for the same payment. In the settlement ledger: two records, different ids, identical `stripe_event_id`, timestamps roughly 60 seconds apart. Always follows a deploy or a database blip — anything that made one handler return a 500.

## Cause

Stripe retries any webhook that does not return a 2xx, with exponential backoff, for up to three days. That is documented and correct behaviour on their side. Our handler was idempotent on *payment intent id*, not on *event id* — and Stripe can legitimately emit more than one event for the same payment intent. So the second delivery of `charge.succeeded` looked like a new fact rather than a redelivery, and produced a second settlement record.

The five hours were spent looking at Stripe's retry logs, which were correct, before looking at our idempotency key, which was not.

## Workaround

Key idempotency on `event.id`, which Stripe guarantees is stable across retries:

```ts
// src/billing/webhooks/handler.ts
const seen = await settlements.findByEventId(event.id);
if (seen) return res.status(200).end();   // already processed; ack and stop
```

The unique index is what actually enforces this — the check above is an optimisation:

```sql
CREATE UNIQUE INDEX settlement_stripe_event_id_uniq
  ON settlement (stripe_event_id);
```

## Do not

**Do not dedupe on payment intent id.** It is the obvious key, it is wrong, and it will silently drop legitimate second events for the same intent — a partial refund following a charge, for instance. This is worse than the bug it fixes, because a dropped settlement is invisible and a duplicated one is not.

**Do not return a 500 for events we do not handle.** Stripe will retry them for three days. Log and return 200 — see the invariants on [[C-0003 Billing webhook receiver]].
