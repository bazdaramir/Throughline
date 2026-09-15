---
type: hub
status: active
project: global
created: 2026-08-13
updated: 2026-08-13
source: human
---

# VERIFY — the hand-check

Bases dashboards cannot be rendered headlessly, so this is the one part of the system that needs
your eyes. **Ten minutes.** Everything else is covered by `sh tools/tl-validate`, which runs in the
repository root.

Re-run this checklist after any change to a `.base` file or to the notes a dashboard counts.

## Open the vault

Obsidian → **Open folder as vault** → `vault/Throughline` inside the repository

Obsidian may ask you to trust the folder. Say yes — the vault ships with `.obsidian/` pre-configured so Bases, Templates, Properties, Graph, and Backlinks are already on and **no community plugins are involved at all**.

## The checklist

### 1 · Dashboards render — the main risk

Open each and confirm it shows rows rather than an error banner.

- [ ] `00 Command/Decisions.base` → **4 rows**, two views (*All decisions*, *Active only*)
- [ ] `00 Command/Gotchas.base` → **3 rows**, second view *Recurring* shows **1** (G-0001)
- [ ] `00 Command/Components.base` → **3 cards**
- [ ] `00 Command/Sessions.base` → **2 entries**; second view *With open threads* shows **2**
- [ ] `00 Command/Specs.base` → **1 row**
- [ ] `00 Command/Needs Review.base` → **non-empty** (expect ~4: G-0003 draft, C-0003 flagged, G-0002 expired, D-0001 stale). The count grows with the calendar — the queue compares dates against today, so D-0003 crosses the 90-day line on 2026-10-30 and D-0004 on 2026-11-10.
- [ ] `01 Projects/_Projects.base` → **1 card** (example-api)

**If a base shows an error**, record the exact message in [[Vault Changelog]]. The three things most likely to be wrong, in order:

1. the `sort:` block — shape inferred from `groupBy`, not documented
2. `note.` prefixes in `order:` — the docs are inconsistent about bare vs. prefixed
3. date math — `created < now() - duration('7d')` in Needs Review

### 2 · Home renders its embedded dashboards

`Home.md` embeds the real `.base` files with `![[File.base#View name]]` — it does **not** restate their filters.

- [ ] **Active projects** → `![[_Projects.base#Projects]]` renders as a card, not as a link or literal text
- [ ] **Needs your attention** → `![[Needs Review.base#Needs review]]` renders as a table with ~4 rows
- [ ] **Recent sessions** → `![[Sessions.base#Recent sessions]]` renders as a list of 2
- [ ] `Today.md` → `![[Sessions.base#With open threads]]` renders as a table of 2
- [ ] The dashboard links at the bottom of Home (`[[Decisions.base|Decisions]]` etc.) resolve rather than showing as unresolved

**If an embed renders as plain text or an unresolved link**, the `#View name` subpath is the thing to suspect — note which one and what it showed. Obsidian autocompletes these: type `![[Needs Review.base#` in a scratch note and it should offer *Needs review* and *Drafts awaiting promotion*. If it offers nothing, the subpath form is wrong.

### 3 · No broken links

- [ ] Open the **Unresolved links** panel. It should be **empty**.
  *(Settings → Appearance won't show it; use the command palette → "Open outgoing links" or check the graph for orphan halos.)*

### 4 · Templates work

Settings → Templates should already point at `90 Templates`.

- [ ] Create a new note, run **Insert template**, pick each of the 7 in turn
- [ ] `{{date:YYYY-MM-DD}}` expands to today's date in `created`/`updated`
- [ ] Frontmatter parses — the Properties panel shows fields, not raw YAML

### 5 · The graph has shape

- [ ] Open Graph view. You should see **hubs and spokes, not a hairball**: `C-0002 Session store` is visibly the densest node, with decisions, gotchas, a spec, and both sessions hanging off it.

### 6 · The system reads as real

- [ ] Open `G-0001 Redis TTL silently resets on SET` and read it. It should read as genuine engineering — if the worked example feels like filler, the product's best teaching asset is broken and needs rewriting.

## Already checked, automatically

`sh tools/tl-validate` covers these on every run, so none of them need your attention:

- every note has frontmatter with the six universal properties, a known `type`, a `status` valid
  for that type, and a `source` of `human`, `agent`, or `backfill`
- every knowledge note sits in the folder its `type` belongs in, under the project it names, with
  the properties its type requires
- every `affects` names a Component in the same project, and every completed supersession is
  recorded on both notes
- every `[[wikilink]]` target exists, and every `[[File.base#View name]]` names a view that file
  defines
- no `tl/` tag outside the eleven — a misspelt `tl/draft` quarantines nothing
- the SessionStart brief, run against a disposable copy of this vault, prints exactly the example in
  the README, keeps a draft out however it is tagged, and reads Obsidian-style lists, CRLF line
  endings, and byte-order marks identically
- one SessionEnd spawns exactly one distiller, and the distiller has no shell

Verified once at build time rather than on every run: all 7 note types appear in at least one
dashboard, and the example's note counts (3 components, 4 decisions, 3 gotchas, 2 sessions, 1 spec,
2 terms, 1 practice).

---

See also: [[Vault Changelog]] for the six places this vault knowingly departs from the blueprint.
