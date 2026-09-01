---
type: hub
status: active
project: global
created: 2026-08-13
updated: 2026-09-02
source: human
---

# Vault Changelog

Versioned record of what changed in the vault, and — more importantly — **every place the shipped vault departs from the blueprint specification, with the reason.** Nothing is dropped silently.

---

## v0.1.5 — 2026-09-02 — Repository polish; documentation reconciled with the built system

No change to the vault's structure, note types, dashboards, or retrieval semantics. This pass fixed
documentation that had fallen behind the code, and added the packaging and validation the repository
was missing.

**The core defect: the docs still described a Phase 1 vault.** Phases 2 and 3 shipped, but several
notes and skills were never updated and told the reader the opposite — that the plugin did not exist
yet. The worst instance was user-facing behaviour: `/throughline:init` closed by saying
`/throughline:backfill` "is a Phase 3 stub and will not write anything yet", which actively steered
users away from a skill that has been fully implemented, subagent and all, since v0.1.3.

Corrected in: `skills/init`, `skills/map`, `skills/brief`, `skills/debrief`,
`agents/throughline-backfiller`, `00 Command/Home.md`, `00 Command/Today.md`,
[[Workflow Cheat Sheet]], and the repository root.

`skills/map` additionally referenced a "Phase 3 Gotcha Guard" as though it were built. It is W10,
marked **V2** in [[Workflow Cheat Sheet]], and deliberately unbuilt — a third hook has to earn its
place against the two-hook constraint. The reference is gone rather than promoted.

**Build-phase vocabulary removed from shipped text.** "Phase 2" and "Phase 3" are facts about how
this was built, not about how it works. They now appear only here, in this changelog, where they
belong. [[VERIFY]] lost its build-conversation voice for the same reason and is now a standing
checklist rather than one-time scaffolding; the instruction to delete it before shipping is
withdrawn, because dashboard rendering will always need human eyes.

**Onboarding was broken.** The plugin had no marketplace manifest, so there was no working install
path — the documented next step for a new user did not exist. Added
`.claude-plugin/marketplace.json` at the repository root, verified with `claude plugin validate .`
and by installing from it. `claude plugin details throughline` now reports the inventory
independently: 12 skills, 2 agents, 2 hooks, 0 MCP servers, 0 LSP servers.

**`tools/tl-validate` added.** The claim in [[VERIFY]] that frontmatter and wikilinks "were verified
programmatically at build time" was true but unreproducible — nothing in the repository could check
it again. The script now does, and asserts the plugin's shape as well, so the two-hook and
twelve-skill constraints fail loudly if anyone drifts from them. Its failure path was exercised, not
assumed. It deliberately ignores wikilinks inside code spans: those are documentation about the
syntax, and treating them as links produced three false positives on the first run.

**A `README.md` was written**, which the repository had never had.

### Deliberately not changed

- No new skills, hooks, subagents, or MCP servers. The counts are identical and now asserted.
- No change to the SessionStart or SessionEnd contracts, the draft quarantine, `source:` semantics,
  the auditor's read-only guarantee, or the Session-notes-are-evidence rule from v0.1.4.
- The two cosmetic Bases gaps from v0.1.0 — alphabetical severity sort, and no per-project note
  count — remain open. Neither is worth the syntax risk yet.
- `plugin.json` still declares `license: Commercial` and the repository carries no `LICENSE` file.
  Flagged rather than resolved: licensing is a decision, not a cleanup task.

---

## v0.1.4 — 2026-08-17 — Session notes are evidence, not quarantined knowledge

Clarification, not a structural change. No vault file was altered.

Automatic SessionEnd capture writes two kinds of note, and they are provenanced differently:

| | `source:` | `#tl/draft` |
|---|---|---|
| **The Session log itself** | `agent` | **not required** |
| **Any additional Decision / Gotcha / Component** | `agent` | **mandatory** |

**Why the Session log is exempt.** §12.4 defines Session notes as *evidence, not knowledge*. They
are never promoted, never reviewed as knowledge, and `Needs Review.base` already excludes
`type: session` by design. Tagging them `#tl/draft` would mark them for a review queue that
deliberately ignores them.

**The quarantine is otherwise untouched.** Every knowledge note produced by automation still carries
`#tl/draft`, is excluded from all retrieval, and can only be promoted by a human. Nothing in the
capture/retrieval asymmetry changed.

Encoded in `plugin/throughline/bin/tl-session-end` only. Verified live across three runs.

---

## v0.1.3 — 2026-08-16 — `Audit Log.md` added for the Phase 3 auditor

Created `98 Meta/Audit Log.md` as the append-only target the auditor writes its findings to.

**Why it was missing.** The blueprint's folder tree (§11.2) lists this file, but its build
instruction (§26.2 1.6) specifies only four meta documents and omits it. Phase 1 correctly followed
the build instruction, which left W12 without the target it names. Phase 3 supplies it.

**One deliberate deviation from §26.4.** The blueprint has the auditor applying `#tl/needs-review`
tags directly to notes, and §11.5's tag table lists the auditor as the applier. That was overridden:
**the auditor is strictly read-only on knowledge notes.** It writes findings to this log and
recommends tags; a human applies them.

The consequence is worth stating plainly: the `file.hasTag("tl/needs-review")` clause in
`Needs Review.base` now only fires for tags **you** apply. Audit findings reach you through the
Audit Log instead of appearing in the dashboard automatically. This is a trade of convenience for
the guarantee that no automated process ever edits a knowledge note.

**This is the only structural change to the vault in Phase 3.** Everything else Phase 3 added lives
in `plugin/throughline/` — two hooks, two shell scripts, and two subagent definitions.

---

## v0.1.2 — 2026-08-16 — Markers for `/throughline:week`

Added `<!-- tl:week:start -->` / `<!-- tl:week:end -->` to `00 Command/Today.md`, wrapping a new
`## Weekly review` section.

W24 specifies that `/throughline:week` stores its output in `Today.md`, but Phase 1 had since given
that note a live Base embed (`![[Sessions.base#With open threads]]`), dashboard links, and a user
scratch list. A plain write would have destroyed all three. The skill now replaces **only** what sits
between the markers, and refuses to run if they are missing rather than guessing where to write.

**This is the only structural change to the vault in Phase 2.** Everything else Phase 2 added lives
in `plugin/throughline/`.

---

## v0.1.1 — 2026-08-13 — Dashboards embedded rather than duplicated

**Defect found on inspection.** `Home.md` and `Today.md` shipped with their dashboard views written as inline ` ```base ` fences that *restated* filter definitions already living in the `.base` files. Two problems: the views did not render as dashboards, and `Home.md` carried its own copy of Needs Review's entire six-clause filter — free to drift from the real one the moment either was edited.

**Fixed.** Both notes now embed the real `.base` files:

| Note | Was | Now |
|---|---|---|
| `Home.md` — Active projects | inline copy of the `_Projects` filter | `![[_Projects.base#Projects]]` |
| `Home.md` — Needs your attention | inline copy of the 6-clause filter | `![[Needs Review.base#Needs review]]` |
| `Home.md` — Recent sessions | inline session filter | `![[Sessions.base#Recent sessions]]` |
| `Today.md` — Open threads | inline session + open_threads filter | `![[Sessions.base#With open threads]]` |
| `Today.md` — Touched recently | inline unfiltered view | links to the five dashboards |

**Syntax source.** Not guessed. Obsidian's help pages for embedding are missing (`/help/bases/embed` 404s), so the syntax was read out of the installed `obsidian.asar` (1.13.7): the embed resolver checks `extension === "base"`, loads the `bases` internal plugin, parses the file's `views:` array, and matches the link subpath against each view's `name:`, registering a `bases-view` embed with `subpath: "#" + viewName`. **`![[File.base#View name]]` is the supported form, and the view is addressed by its `name:` field.**

**No dashboard logic was changed.** The `.base` files are byte-identical to v0.1.0. The two `limit:` values that existed only in Home's inline copies (15 and 10) are gone rather than pushed into the shared `.base` files, since that would have altered the standalone dashboards.

**Open gap:** `Today.md`'s "touched in the last seven days" cross-type view has no backing `.base` file. It was replaced with links rather than inventing a dashboard the blueprint does not specify. Making it a real view requires a new `.base` file; it is listed as an open gap rather than guessed at.

---

## v0.1.0 — 2026-08-13 — Stage 1: the vault

First build. Folder scaffold, 7 templates, 7 Bases dashboards, 2 hub notes, the `example-api` worked example, 2 Terms, 1 Practice, 4 meta documents.

**Not yet built:** the Claude Code plugin (12 skills, 2 hooks, 2 subagents) and the product documentation set. Every `/throughline:*` command referenced in this vault arrives in Phase 2 and Phase 3. Until then the vault is operated by hand from `90 Templates/`.

### Departures from the blueprint, and why

**1 · Bases filter syntax rewritten.**
The blueprint (§11.7, §26.3) writes dashboard filters as pseudocode:
`status == "draft" OR tags.contains("tl/needs-review")`.
Real Bases syntax is structured YAML with `and`/`or`/`not` keys, and tags are matched with `file.hasTag("...")`. Every `.base` file was translated. No filter intent was lost. The blueprint anticipated this and instructed verification against the Bases docs first — done.

**2 · `verified` → `last_verified`.**
`Needs Review.base` in the blueprint filters on `verified == null`, but `verified` is defined nowhere in §12 — it appears in no note type. `last_verified` **is** defined (Decision and Component, optional). Resolved by using `last_verified` and promoting it to an optional *universal* property, documented in [[Note Types Reference]]. No new property was invented.

**3 · The draft clause is two clauses, not one.**
`status == "draft"` is a valid status **only for Spec**. For every other type, draft-ness is the tag `#tl/draft` (§11.5). The hygiene filter therefore carries both `status == "draft"` and `file.hasTag("tl/draft")`. This is not a contradiction in the blueprint — the two clauses cover genuinely different states.

**4 · Sessions and hubs excluded from the "unverified and older than 7 days" clause.**
As literally specified, that clause would pull **every Session note** into the hygiene queue permanently — sessions have no `last_verified` and are never reviewed, because they are evidence rather than knowledge (§12.4). Hub notes (`Home`, `Today`, `_Project`) would drift in for the same reason. Both are excluded with `type != "session"` and `type != "hub"`. Without this the queue is unusable within a week, which would kill the one ritual the product asks for.

**5 · `_Projects.base` ships without the "note count" column.**
The blueprint's §26.3 table asks for a per-project note count. Bases evaluates properties per file and has no folder-aggregate count function. The view ships with folder, status, stack, and updated instead. **Open gap** — revisit if Bases adds aggregation.

**6 · Gotcha severity sorts alphabetically.**
`Gotchas.base` sorts by `severity` ascending as specified, which orders `critical, high, low, medium` — not the semantic order. A ranking formula was considered and rejected as syntax risk for a cosmetic gain; `critical` and `high` still land at the top, which is where attention needs to go. **Open gap** — revisit with a formula once Bases formula syntax is confirmed by hand.

### Needs hand-verification in Obsidian

Bases cannot be rendered headlessly. See [[VERIFY]] for the checklist. The specific uncertainties:

- the `sort:` key shape (`- property: … / direction: ASC`) is inferred from the documented `groupBy` shape; the help pages do not document `sort` explicitly
- `note.` prefixes in `order:` lists versus bare property names
- date arithmetic — `created < now() - duration('7d')`
- negation — `!open_threads.isEmpty()` in `Sessions.base` and `Today.md`
- wikilinks to `.base` files (`[[Needs Review.base|Needs Review]]`) resolving in Obsidian

Anything that fails gets fixed and recorded here in v0.1.1.
