---
type: spec
status: ready
project: example-api
created: 2026-08-12
updated: 2026-08-12
source: human
affects: ["[[C-0002 Session store]]"]
depends_on: ["[[D-0004 Move session storage to Postgres]]", "[[D-0001 Route all database access through the repository layer]]"]
risk: medium
estimate: "6h"
last_verified: 2026-08-12
---

## Objective

Postgres is the authoritative session store in production, the Redis adapter is deleted, and `revoke` is synchronous and total.

## Constraints
<!-- auto-populated from linked Decisions and Practices -->

- All Postgres access goes through `src/db/repositories/` — [[D-0001 Route all database access through the repository layer]]. The session adapter is **not** exempt; this rule has already been broken twice in this exact file.
- Expiry is written explicitly on every write, never inherited — [[P Never rely on implicit expiry for anything security-sensitive]].
- Session ids are never reused, including across the migration. Do not renumber.

## Known landmines
<!-- auto-populated from Gotchas on affected Components -->

- [[G-0003 Logged-out sessions still resolve for up to 30 seconds]] — the in-process cache is the actual bug. Moving stores does **not** fix it unless the cache is dropped in the same change. Shipping the migration and leaving the cache in place would close nothing while appearing to.
- [[G-0001 Redis TTL silently resets on SET]] — not inherited by Postgres, but the reaper introduces the mirror-image risk: expiry that never runs rather than expiry that resets.

## Approach

1. Add `session` table plus repository methods in `src/db/repositories/session.ts`. Unique index on `id`; index on `(merchant_id, expires_at)` for the "active sessions" query [[D-0002 Store sessions in Redis with a sliding TTL]] foreclosed.
2. Make the Postgres adapter authoritative behind the `SESSION_STORE` env var. Keep Redis shadow-reading and log disagreements — do not fail on them.
3. Run seven consecutive days with zero disagreements. This is the gate in [[D-0004 Move session storage to Postgres]] and it is not negotiable for a shorter window.
4. Remove the in-process session cache. Measure p50/p99 before and after; the budget is 5ms p99.
5. Delete `src/auth/store/redis.ts` and `src/auth/store/index.ts`. Flip the interface to call the repository directly.
6. Ship the reaper as a scheduled job with an alert if it falls more than one hour behind.

## Out of scope

- **Rate limiting stays on Redis.** Redis is not being removed from the stack, only from session storage.
- **No change to `src/auth/session.ts`'s public interface.** `read`/`write`/`revoke` keep their signatures. If this migration requires an interface change, stop and reopen [[D-0004 Move session storage to Postgres]].
- **No session data backfill.** Existing Redis sessions expire naturally within 30 minutes of cutover. Do not write a migration script for them.
- **Do not touch `legacy/billing/`** for any reason, including unrelated lint — [[D-0003 Freeze the legacy billing module]].

## Done when

- [ ] `SESSION_STORE=postgres` in production for 7 days with zero shadow-read disagreements
- [ ] p99 keyed session lookup under 5ms measured on production traffic, not staging
- [ ] In-process session cache removed; logout is total and observable in an integration test
- [ ] `src/auth/store/` contains no Redis code
- [ ] Reaper deployed with a lag alert
- [ ] [[C-0002 Session store]] updated: `stability: stable`, `external_deps` no longer lists Redis
