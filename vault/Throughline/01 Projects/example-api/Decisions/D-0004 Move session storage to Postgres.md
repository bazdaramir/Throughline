---
type: decision
status: active
project: example-api
created: 2026-08-11
updated: 2026-08-12
source: human
affects: ["[[C-0002 Session store]]"]
confidence: medium
reversal_cost: medium
supersedes: "[[D-0002 Store sessions in Redis with a sliding TTL]]"
last_verified: 2026-08-12
evidence: ["[[2026-08-11-0915]]"]
---

## Decision

Move session storage from Redis to Postgres, behind the existing `src/auth/session.ts` interface, and retire the Redis adapter once shadow reads agree for seven consecutive days.

## Context

The trigger written into [[D-0002 Store sessions in Redis with a sliding TTL]] fired: the dashboard-load query was fixed in July, and keyed session lookup in Postgres now measures p99 3.1ms against a 5ms budget. Meanwhile Redis has cost us two production incidents that Postgres structurally cannot reproduce — [[G-0001 Redis TTL silently resets on SET]] and [[G-0003 Logged-out sessions still resolve for up to 30 seconds]] — because in Postgres, revocation and the transaction that triggers it are the same transaction.

`confidence: medium` and not `high`: the p99 figure is from staging under synthetic load. Production traffic has a different key distribution and we have not measured it.

## Alternatives considered

| Option | Why not |
|---|---|
| Keep Redis, fix the two gotchas | Both are workaround-able but neither is fixable. The TTL semantics are Redis's, and synchronous revocation across two stores is a distributed transaction. |
| Keep Redis as a read-through cache in front of Postgres | Retains the revocation race we are trying to eliminate, and adds an invalidation problem on top. Strictly worse than either endpoint. |
| Move to a managed session service | New vendor, new failure mode, new bill, and session data leaves our boundary. Not worth it for a keyed lookup. |

## Consequences

**Easier:** revocation becomes synchronous and total, which restores the invariant [[C-0002 Session store]] claims and currently violates. "All active sessions for this merchant" becomes a trivial query — the thing [[D-0002 Store sessions in Redis with a sliding TTL]] foreclosed and the dashboard team asked for twice.

**Harder:** expiry is no longer free. Postgres has no TTL, so we own a reaper job and the operational question of what happens when it stops running. That is a new failure mode we are choosing deliberately.

**Forecloses:** nothing we currently want. Redis stays in the stack for rate limiting.

## Revisit when

Production p99 for keyed session lookup exceeds 5ms for a sustained hour, or the reaper falls more than one hour behind on expired rows.
