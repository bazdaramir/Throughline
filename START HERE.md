# Throughline — start here

If you are reading this on GitHub, read [README.md](README.md) instead. It covers the whole system.

This file is for someone who already has the folder open locally and wants the shortest path in.

## 1 · Open the vault

Obsidian → *Open folder as vault* → `vault/Throughline`

Say yes if it asks you to trust the folder. The vault ships with `.obsidian/` pre-configured, so
Bases, Templates, Properties, Graph, and Backlinks are already on. No community plugins are
involved.

## 2 · Read one note

Start at `00 Command/Home.md`, then open
`01 Projects/example-api/Gotchas/G-0001 Redis TTL silently resets on SET.md`.

That one note teaches the system faster than any manual. Then look at its backlinks pane — every
decision, spec, and session that touched the same component, assembled without anyone writing a
query. That is the argument for a vault over a `docs/` folder, in one gesture.

## 3 · Install the plugin

From this directory:

```bash
claude plugin marketplace add ./
```

```bash
claude plugin install throughline@throughline
```

Then point Throughline at the vault — resolution is `$THROUGHLINE_VAULT` → the `vault:` line in a
repo's `.throughline` → `~/Throughline`, and the vault lives inside this repository rather than at
the fallback:

```bash
export THROUGHLINE_VAULT="/absolute/path/to/Throughline/vault/Throughline"
```

Now run `/throughline:init` inside any repository you want tracked.

The vault works without the plugin — open `90 Templates/` and write a note by hand. The plugin only
removes the typing.

## Where things are

| | |
|---|---|
| How the system is designed | `vault/Throughline/98 Meta/How Throughline Works.md` |
| What each note type is for | `vault/Throughline/98 Meta/Note Types Reference.md` |
| All 26 workflows, one line each | `vault/Throughline/98 Meta/Workflow Cheat Sheet.md` |
| What changed, and every deviation from spec | `vault/Throughline/98 Meta/Vault Changelog.md` |
| The hand-check Obsidian needs | `vault/Throughline/98 Meta/VERIFY.md` |
| Run the static checks | `sh tools/tl-validate` |
