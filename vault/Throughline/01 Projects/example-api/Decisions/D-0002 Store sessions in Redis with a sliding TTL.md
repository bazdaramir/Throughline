---
type: decision
status: superseded
project: example-api
created: 2026-06-06
updated: 2026-08-11
source: human
affects: ["[[C-0002 Session store]]"]
confidence: high
reversal_cost: medium
superseded_by: "[[D-0004 Move session storage to Postgres]]"
last_verified: 2026-08-11
evidence: ["commit:1f7ab93"]
---

> [!info] Superseded — kept, not deleted
> This decision was replaced by [[D-0004 Move session storage to Postgres]]. It stays in the vault because the *reasoning* is still load-bearing: it explains why the Redis adapter is shaped the way it is, and the migration has to preserve two of the properties argued for here.

## Decision

Store session state in Redis, keyed by session id, with a 30-minute TTL refreshed on every authenticated request.

## Context

We needed session lookup on every request with a p99 under 5ms, and at the time Postgres was already the bottleneck on the merchant dashboard's dashboard-load query. Redis was already in the stack for rate limiting, so this added no new dependency.

## Alternatives considered

| Option | Why not |
|---|---|
| Signed stateless JWTs | Revocation is the whole problem. A stateless token cannot be revoked without a revocation list, which is a session store wearing a disguise. |
| Postgres table | Rejected *at the time* on latency grounds. This is exactly what [[D-0004 Move session storage to Postgres]] later reversed, once the dashboard query was fixed and the real numbers came in. |
| In-memory per instance | Breaks the moment we run more than one instance, which we already did. |

## Consequences

**Easier:** sub-millisecond lookups. TTL expiry is free — no reaper job.

**Harder:** sessions are now in a store with no transactional relationship to the merchant rows they reference. Revocation is eventually consistent, which produced [[G-0003 Logged-out sessions still resolve for up to 30 seconds]]. The sliding-TTL refresh produced [[G-0001 Redis TTL silently resets on SET]].

**Forecloses:** any "show me all active sessions for this merchant" query, which the dashboard team asked for twice and we declined twice.

## Revisit when

Session lookup stops being on the hot path, or Postgres p99 for a keyed lookup drops under 5ms.

*(It was revisited on exactly this trigger. That is what the `Revisit when` field is for — see [[D-0004 Move session storage to Postgres]].)*
