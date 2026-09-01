---
type: term
status: active
project: global
created: 2026-06-10
updated: 2026-06-10
source: human
aliases: ["billing period", "invoice period"]
code_refs: ["src/billing/cycle.ts"]
not_to_be_confused_with: "[[T Settlement window]]"
authority: "Merchant Agreement §3.1"
last_verified: 2026-06-10
---

## Definition

The calendar interval over which we aggregate a merchant's activity to produce one invoice. Fixed length, fixed boundaries, set per merchant at onboarding and never changed thereafter.

## In code

`billing_cycle` is `(merchant_id, starts_on, ends_on)` on the `invoice` table — plain `date`, not `timestamptz`, because cycles are calendar facts and have no time zone of their own.

```ts
type BillingCycle = { startsOn: LocalDate; endsOn: LocalDate };   // inclusive both ends
```

Note `endsOn` is **inclusive**. This is inconsistent with [[T Settlement window]]'s half-open range and it is a real, deliberate wart: invoices are legal documents that name a last day, and off-by-one on that day is a customer complaint.

## Not

- **Not [[T Settlement window]].** The distinction that causes real bugs: a payment can fall inside billing cycle N while its settlement window closes during cycle N+1. The invoice for cycle N includes the payment; the funds arrive during N+1. Merchants ask about this constantly and the answer is "these are two different clocks."
- **Not adjustable.** Changing a merchant's cycle after onboarding would make two invoices overlap or leave a gap. There is no code path to do it, on purpose.
- **Not a subscription period.** We do not bill subscriptions. If that changes, this term needs rewriting rather than extending.

## Examples

- Merchant onboarded 14 March → cycles run the 14th to the 13th, forever.
- Cycle `2026-07-14 … 2026-08-13` contains a payment captured 2026-08-13 whose settlement window closes 2026-08-16, inside the *next* cycle. Both are correct. This is the example to give anyone who is confused.
