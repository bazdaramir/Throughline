---
type: component
status: active
project: example-api
created: 2026-06-04
updated: 2026-08-12
source: human
paths: ["src/auth/session.ts", "src/auth/store/"]
parent: "[[C-0001 Auth subsystem]]"
external_deps: ["Redis 7.2", "Postgres 16"]
stability: volatile
owner: amir
last_verified: 2026-08-12
---

## What it does

Holds session state — merchant id, scopes, issued-at, expiry — and answers lookups by session id. It is currently being moved from Redis to Postgres, so both adapters exist and the Redis one is authoritative until [[S-0001 Migrate session storage to Postgres]] lands.

## Key files

- `src/auth/session.ts` — the public interface. `read`, `write`, `revoke`. Everything else is an implementation detail.
- `src/auth/store/redis.ts` — current authoritative adapter.
- `src/auth/store/postgres.ts` — the replacement. Written, shadow-reading in staging, not yet authoritative.
- `src/auth/store/index.ts` — adapter selection by env var. Delete this once the migration completes.

## How it fits

**Upstream:** [[C-0001 Auth subsystem]] middleware, on every authenticated request.

**Downstream:** Redis today, Postgres tomorrow. All Postgres access goes through the repository layer per [[D-0001 Route all database access through the repository layer]] — the Postgres adapter is not exempt, and reviewers have let that slide twice.

## Invariants

- **Session IDs are never reused.** Not after revocation, not after expiry, not after a wipe. Downstream audit logging assumes an id maps to exactly one session for all time.
- **`revoke` is synchronous and total.** When it returns, no subsequent `read` may resolve that session. This invariant is currently *violated* under Redis — see [[G-0003 Logged-out sessions still resolve for up to 30 seconds]].
- **Expiry is written explicitly on every write.** Never inherited, never implicit. See [[P Never rely on implicit expiry for anything security-sensitive]].
- **The two adapters must agree.** While both are live, a session written by one must be readable by the other. This is what the shadow-read in staging is testing.

## Known gotchas
<!-- populated from backlinks — do not edit -->

## Decisions governing this
<!-- populated from backlinks — do not edit -->
