---
type: hub
status: active
project: example-api
created: 2026-06-02
updated: 2026-08-13
source: human
stack: ["typescript", "node 22", "express", "postgres 16", "redis 7.2"]
repo: git@github.com:example/example-api.git
---

# example-api

> [!info] This is the worked example that ships with Throughline
> It is a fictional but realistic Node/TypeScript billing API. Read three or four of its notes and you will understand the whole system faster than the manual can explain it. **Delete this folder once your own projects are in the vault** — or keep it as a reference.

## Charter

A billing API for a marketplace. Takes payment events from Stripe, resolves them against our own settlement schedule, and exposes session-authenticated endpoints to the merchant dashboard.

## Stack

TypeScript on Node 22, Express, Postgres 16 as the system of record, Redis 7.2 for sessions (being retired — see [[D-0004 Move session storage to Postgres]]).

## Map

| Component | What it is |
|---|---|
| [[C-0001 Auth subsystem]] | Everything that answers "who is this request" |
| [[C-0002 Session store]] | Where session state actually lives. Currently mid-migration. |
| [[C-0003 Billing webhook receiver]] | Ingests Stripe events. The riskiest surface in the codebase. |

## Live work

- [[S-0001 Migrate session storage to Postgres]] — `ready`, blocked on nothing, ~6h
- Open landmines: [[G-0001 Redis TTL silently resets on SET]], [[G-0002 Stripe webhook retries deliver duplicate events after a 500]]

## Standing constraints

- **Never touch `legacy/billing/`** — [[D-0003 Freeze the legacy billing module]] is `irreversible`.
- All database access goes through the repository layer — [[D-0001 Route all database access through the repository layer]].
- Domain vocabulary that is easy to get wrong: [[T Settlement window]], [[T Billing cycle]].
