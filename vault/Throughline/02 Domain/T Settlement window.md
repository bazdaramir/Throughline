---
type: term
status: active
project: global
created: 2026-06-10
updated: 2026-07-15
source: human
aliases: ["settlement period", "clearing window"]
code_refs: ["src/billing/settlement.ts"]
not_to_be_confused_with: "[[T Billing cycle]]"
authority: "Merchant Agreement §4.2"
last_verified: 2026-07-15
---

## Definition

The interval between a payment being captured and the funds becoming withdrawable by the merchant. It opens at capture and closes when the acquirer confirms clearing — **not** on a fixed schedule.

## In code

`settlement_window` is a Postgres `tstzrange` on the `settlement` table. Open windows have an unbounded upper bound. `src/billing/settlement.ts` computes the lower bound from the Stripe event; the upper bound is written later by the clearing reconciliation job.

```ts
type SettlementWindow = { opensAt: Date; closesAt: Date | null };  // null = still open
```

## Not

- **Not a fixed 48-hour or 72-hour period.** Duration varies by acquirer and by merchant risk band. Code that assumes a constant is wrong, and has been written twice.
- **Not the same as [[T Billing cycle]].** A billing cycle is ours and calendar-driven; a settlement window is the acquirer's and event-driven. They overlap arbitrarily. One billing cycle can contain many settlement windows, and a settlement window can straddle two billing cycles.
- **Not closed when the merchant sees "settled" in the dashboard.** The dashboard shows *expected* close, which is an estimate.

## Examples

- Capture Tue 09:00, acquirer confirms Thu 14:20 → window `[Tue 09:00, Thu 14:20)`. Duration 53h20m.
- Capture Fri 17:00, high-risk merchant, confirms following Wed → duration 5 days. Same code path, same table, no special case.
- A disputed charge → window never closes; upper bound stays null until the dispute resolves. Any query that assumes `closesAt` is eventually non-null will hang on this row forever.
