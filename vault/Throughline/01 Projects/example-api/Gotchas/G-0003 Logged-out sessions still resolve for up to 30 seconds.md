---
type: gotcha
status: open
project: example-api
created: 2026-08-12
updated: 2026-08-12
source: agent
affects: ["[[C-0002 Session store]]"]
severity: high
cost: "2h"
external: "Redis 7.2"
tags:
  - tl/draft
  - tl/security
---

> [!question] `#tl/draft` — auto-captured, not yet confirmed by a human
> This note was written by an agent at the end of a session. **It is excluded from every retrieval path** — the SessionStart brief will not mention it, `/throughline:brief` will not include it, and no skill may promote it. A human removes the `tl/draft` tag after a ten-second read, or deletes the note.
>
> This quarantine is the mechanism that makes automatic capture safe. Without it, auto-capture poisons the vault within a month.

## Symptom

After a merchant clicks "log out", requests carrying the old session cookie keep succeeding — usually for a few seconds, observed once at 28 seconds. The dashboard shows them as logged out while the API still answers. No error anywhere; the session simply resolves.

## Cause

Revocation deletes the Redis key, but the API runs four instances and each keeps a 30-second in-process cache of resolved sessions to avoid a Redis round trip per request. The cache is not invalidated on revoke — there is no mechanism to invalidate it, because instances do not know about each other.

So `revoke` is synchronous with respect to Redis and eventually consistent with respect to the thing that actually answers requests. That violates the invariant [[C-0002 Session store]] states plainly: *"`revoke` is synchronous and total."*

## Workaround

None that is both correct and cheap. Three options, none adopted yet:

1. Drop the in-process cache. Correct immediately, costs a Redis round trip per request. Measured at +0.4ms p50 — probably acceptable, nobody has approved it.
2. Publish revocations on a Redis pub/sub channel and have instances evict locally. Correct in the normal case, silently wrong if an instance misses the message.
3. Ship [[S-0001 Migrate session storage to Postgres]], which removes the race structurally — revocation and the transaction that triggers it become the same transaction. This is the reasoning recorded in [[D-0004 Move session storage to Postgres]].

Until one of these lands, treat logout as "effective within 30 seconds" and say so to anyone who asks.

## Do not

**Do not shorten the cache TTL to one second and call it fixed.** It narrows the window without closing it, and it converts a reproducible 30-second bug into an intermittent 1-second bug — much harder to detect, equally exploitable, and it will be forgotten.
