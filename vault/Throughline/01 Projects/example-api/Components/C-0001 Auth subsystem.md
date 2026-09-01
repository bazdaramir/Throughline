---
type: component
status: active
project: example-api
created: 2026-06-04
updated: 2026-07-28
source: human
paths: ["src/auth/"]
stability: stable
owner: amir
last_verified: 2026-07-28
---

## What it does

Answers one question for every inbound request: which merchant is this, and are they still allowed to be here. Everything else about authentication — login UI, password reset, OAuth handshakes — lives in the dashboard app, not here.

## Key files

- `src/auth/middleware.ts` — the Express middleware every route mounts. The only legitimate entry point.
- `src/auth/session.ts` — session read/write. See [[C-0002 Session store]].
- `src/auth/scopes.ts` — merchant scope resolution. Pure functions, heavily tested.
- `src/auth/store/` — storage adapters. Currently two: Redis and Postgres.

## How it fits

**Upstream:** the merchant dashboard sends a session cookie; the webhook receiver does *not* pass through here (Stripe authenticates by signature instead — see [[C-0003 Billing webhook receiver]]).

**Downstream:** calls [[C-0002 Session store]] for state, and `src/db/repositories/merchant.ts` for scope lookup.

## Invariants

- **A request that reaches a route handler has a resolved merchant.** There is no "maybe authenticated" state. If resolution fails, the middleware throws before the handler runs.
- **Scope checks are never performed inside route handlers.** They happen in middleware, declaratively. A scope check in a handler is a bug, not a style choice.
- **`src/auth/scopes.ts` stays pure.** No I/O, no clock reads. It is the only part of auth that is exhaustively unit-tested, and that is only possible while it stays pure.

## Known gotchas
<!-- populated from backlinks — do not edit -->

## Decisions governing this
<!-- populated from backlinks — do not edit -->
