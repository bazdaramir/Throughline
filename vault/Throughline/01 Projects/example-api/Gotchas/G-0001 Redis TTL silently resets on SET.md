---
type: gotcha
status: open
project: example-api
created: 2026-06-21
updated: 2026-08-05
source: human
affects: ["[[C-0002 Session store]]"]
severity: high
cost: "3h"
recurrence: 2
external: "Redis 7.2"
last_verified: 2026-08-05
tags:
  - tl/painful
  - tl/external
---

> [!warning] `recurrence: 2` — this has bitten twice
> A landmine you have stepped on twice is not a landmine, it is a missing convention. This gotcha graduated into [[P Never rely on implicit expiry for anything security-sensitive]].

## Symptom

Sessions that should expire 30 minutes after the last request survive for hours. In logs: a session id with `issued_at` six hours old still resolving successfully. No error, no warning, nothing in the Redis slowlog. Reproduces only under real traffic, never in tests.

## Cause

`SET key value` **clears any TTL previously associated with the key.** Redis treats a plain `SET` as a full replacement of the value *and* its expiry metadata. Our `touch()` path wrote the refreshed session with `SET` and then called `EXPIRE` as a second command. Whenever the process was killed between the two — deploy, OOM, scale-in — the key was left immortal.

The window is small, which is why it took two incidents to find: it only matters if you die between two commands that are microseconds apart.

## Workaround

Write value and expiry in a single atomic command:

```ts
// src/auth/store/redis.ts
await redis.set(key, payload, { EX: ttlSeconds });   // one command, one round trip
```

Never this:

```ts
await redis.set(key, payload);
await redis.expire(key, ttlSeconds);                  // ← the process can die here
```

To find keys already leaked by this bug: `redis-cli --scan --pattern 'sess:*'` then `TTL` each; anything returning `-1` has no expiry and should be deleted.

## Do not

**Do not "fix" this with a background sweeper that scans for TTL-less session keys and expires them.** That was the first fix proposed, it looks reasonable, and it is wrong: it converts a small correctness bug into a permanent operational dependency, and it leaves the race intact — a session can still be read during the window before the sweeper reaches it. Fix the write, not the aftermath.

**Do not** assume `SETEX` is safer. It is the same atomicity as `SET ... EX`; the problem was never which command, it was that there were two of them.
