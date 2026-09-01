---
type: decision
status: active
project: example-api
created: 2026-06-05
updated: 2026-06-05
source: human
affects: ["[[C-0002 Session store]]", "[[C-0003 Billing webhook receiver]]"]
confidence: high
reversal_cost: high
last_verified: 2026-02-10
evidence: ["commit:8c41d0e"]
---

> [!note] Why this note is in the hygiene queue
> `last_verified` is more than 90 days old while `status` is still `active`. Throughline surfaces active decisions nobody has re-checked in a quarter, because a decision everyone has stopped verifying is usually a decision the code has quietly stopped honouring.

## Decision

Every read and write to Postgres goes through a repository module in `src/db/repositories/`. No route handler, middleware, or service constructs SQL or touches the client directly.

## Context

Two weeks in, we had raw queries in four places, two of which disagreed about whether `merchant_id` was nullable. The disagreement was invisible until a null landed in production and a settlement went to nobody. The constraint is not "SQL is bad" — it is that we need exactly one place per table where the shape of a row is asserted.

## Alternatives considered

| Option | Why not |
|---|---|
| A full ORM (Prisma, TypeORM) | Owns the schema and the migration story. We already have migrations we like, and the settlement ledger's append-only constraints are awkward to express. |
| Query builder only (Knex) | Solves SQL ergonomics, not the duplication. You can still build the same wrong query in four places. |
| Raw SQL with a shared types file | Types drift from queries silently. Nothing forces the two to stay in sync. |
| Nothing — just review it | Tried for two weeks. Produced the null-merchant incident. |

## Consequences

**Easier:** one place to change when a column changes. Test doubles are trivial — swap the repository. Static analysis can assert that nothing outside `src/db/repositories/` imports the client.

**Harder:** simple one-off queries now need a repository method, which feels like ceremony and is the reason this rule gets quietly broken. It has been broken twice, both times in [[C-0002 Session store]]'s Postgres adapter.

**Forecloses:** any library that wants to own the connection, which is most ORMs.

## Revisit when

A repository method count passes ~40 for a single table, which would mean the repository has become a junk drawer rather than a boundary.
