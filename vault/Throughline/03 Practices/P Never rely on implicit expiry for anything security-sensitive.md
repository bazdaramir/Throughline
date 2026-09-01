---
type: practice
status: active
project: global
created: 2026-08-12
updated: 2026-08-12
source: human
scope: universal
applies_to: ["typescript", "redis", "postgres"]
origin: ["[[G-0001 Redis TTL silently resets on SET]]", "[[G-0003 Logged-out sessions still resolve for up to 30 seconds]]"]
enforcement: review
last_verified: 2026-08-12
---

> [!info] This is what a graduated gotcha looks like
> Two landmines, five hours between them, same underlying mistake. A rule with no scar tissue behind it is an opinion; a rule linked to the gotchas that produced it is a hard-won law. The `origin` property is that link — and it is why the agent can cite the pain, not just the rule.

## The rule

If a thing must stop being valid at a specific moment, write that expiry explicitly, in the same atomic operation that writes the thing — and verify expiry at read time, never trust the store to have enforced it.

## Why

Both of the gotchas in `origin` are the same mistake wearing different clothes:

- [[G-0001 Redis TTL silently resets on SET]] — expiry was written as a *second* command after the value. A process death between the two left sessions immortal. The store was trusted to hold expiry metadata that was set separately from the data.
- [[G-0003 Logged-out sessions still resolve for up to 30 seconds]] — expiry (revocation) was applied to the store, but the thing actually answering requests was an in-process cache that had never heard of it. The store enforced expiry correctly and it did not matter.

The pattern: **expiry that lives anywhere other than the data it governs will eventually diverge from it.** Under load, during a deploy, across a cache — the divergence window is small, which is exactly why it takes two incidents to find.

It costs three lines to do right and roughly three hours to debug when it is wrong. That ratio is the whole argument.

## How to comply

**Write value and expiry atomically.**

```ts
await redis.set(key, payload, { EX: ttlSeconds });     // one command
// never: set(...) then expire(...)
```

**Store expiry as data, not as store metadata**, wherever the store allows it:

```sql
expires_at timestamptz not null    -- not a TTL setting, a column
```

**Check expiry at read time regardless of what the store promises.**

```ts
const s = await sessions.read(id);
if (!s || s.expiresAt <= new Date()) return null;   // do not assume the store reaped it
```

**Any cache in front of an expiring thing inherits the expiry problem.** Either give the cache a TTL strictly shorter than the shortest thing it caches, or do not cache the thing.

## Exceptions

Non-security-sensitive caches where staleness is merely inefficient — a memoised config read, a rendered template. The test: *if this value were honoured one minute after it should have expired, would that be a security or correctness incident?* If yes, this rule applies. If no, cache freely.
