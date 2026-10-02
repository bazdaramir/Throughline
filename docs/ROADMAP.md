# Throughline roadmap

This is a living document. It is revised after each meaningful stage, and every item says *why* it
sits where it does. The ordering rule is **user value × reliability × trust × differentiation**,
not feature count.

**The product, in one sentence.** Throughline captures project knowledge automatically, preserves
provenance, and retrieves the right context when it matters — without ever letting unverified
information into retrieval. Anything that weakens the second half of that sentence is rejected, no
matter how useful it looks.

**Where it stands.** A serious prototype. The trust model is real and tested. The weakest part is
not correctness but *adoption*: the loop capture → quarantine → human review → retrieval only pays
off once a person has reviewed something, and until now reviewing meant five hand edits.

## Competitive position

Checked 2026-10-02 against the alternatives a developer already has.

| | What they give you | Where Throughline differs |
|---|---|---|
| `CLAUDE.md` / Claude Code memory | Always-loaded, hand-written instructions | Retrieval is selective (≤400 tokens), capture is automatic, and decay is audited |
| claude-mem (a Bun worker, SQLite and a Chroma vector DB, MCP search tools, a web viewer, cloud sync) | Automatic capture of every observation, with semantic search | Its README describes **no review step** — observations are stored and searchable as captured. Heavy: a daemon, a database, a vector store. Throughline is `sh` and Markdown, and nothing unreviewed reaches the model |
| A `docs/` folder or an ADR repo | Durable, reviewable, in git | Nothing captures or retrieves for you |
| Obsidian on its own | Excellent human browsing | No capture, no retrieval into the agent |

**Where Throughline is weak.** Review is manual work (being fixed — see NOW). There is no search
(`/throughline:why` and `/throughline:brief` are model-driven walks over notes, not an index).
Retrieval is exact path matching, so a decision recorded against a since-renamed directory is
missed. There is no web UI and no team story. CI now runs the gate on Linux, macOS and Windows,
but no real-world installs exist beyond the author's machine.

**Where the genuine opportunity is.** *Trust as the product.* Memory tools compete on how much they
remember; the unaddressed problem is that agents act on memories nobody checked. A memory with
provenance, a quarantine, and an auditable human promotion step is something a team can adopt and an
organisation can approve. That is the thing to build on.

---

## NOW

**Stage 3 — make the review loop visible and trustworthy.** *(this release — changelog v0.1.9)*
Capture → quarantine → human review → retrieval only feels natural if each step can be seen.
- The brief **counts** the unreviewed drafts that would reach it, never showing one, so a quiet
  brief is no longer mistaken for an empty vault.
- `tl-promote` says what is **trusted**, what is **waiting**, and what was **recently promoted**.
- Previewing is now **truly read-only**: no temporary file is written into the vault.
- The one channel into a future session that no human reviews — the distiller's `open_threads` —
  is **labelled, capped and cleaned**.
- Two notes sharing an id now **fail the vault contracts**.
- **CI** runs the gate on Linux, macOS and Windows, so portability is measured rather than assumed.
  Its first run found a real bug — the SessionEnd hook did not parse under macOS's `/bin/sh` — and
  it is fixed; all three platforms then passed 72/72.

Stage 2 (`tl-promote`) shipped in v0.1.8: lists drafts, previews the exact edit, applies only with a
person at a terminal, walks the queue, completes supersessions, and logs each promotion.

## NEXT

1. **Widen CI where it is thin.** The three `*-latest` runners ship gawk (Ubuntu, Windows) and BSD
   awk (macOS). Not covered: **`mawk`**, the default awk on many Debian and Ubuntu installs;
   busybox awk and sed (Alpine); `zsh` as `sh`; and an older macOS than the runner's. A job that
   installs `mawk`, and one in an `alpine` container, would close most of that. The first run
   already earned its place by finding a real bug.
2. **Verify `tl-promote` at a real terminal on Windows.** The apply path has only been exercised
   through its test seam. Whether the `-t 0`/`-t 1` check behaves in mintty, PowerShell and Windows
   Terminal is untested, and no CI runner has a terminal.
3. **Share the draft detector.** The rule for "is this a draft" exists in both `tl-brief` and
   `tl-promote`. The validator promotes a note and checks the brief then retrieves it, for several
   tag shapes, which catches drift but is no substitute for one implementation.
4. **Find `.throughline` from a subdirectory.** Both hooks and `tl-promote` look only in the current
   directory, so Claude Code started in `src/` gets no brief and no distillation. Walk up to the git
   root. Decide, and document, whether `resume` and `compact` sessions should get a brief.
5. **Validate any vault, not just the one in this repo.** The contract checks (status per type,
   folder per type, `affects` names a Component, supersession recorded on both notes, unique ids)
   would catch a hand edit that silently takes a note out of retrieval. Ship them as a command a
   user can run on their own vault.
6. **Per-project weekly review.** `/throughline:week` writes into one global `Today.md`, so a
   multi-project vault would overwrite itself. Needs a contract decision before code.
7. **A tool-restricted auditor.** The auditor's "flags, never modifies" guarantee rests on a prompt
   while it holds Bash. Pre-collect the git facts it needs, as the distiller now does, and drop Bash.
8. **Stop two writers allocating the same id.** The validator now detects a duplicate; nothing yet
   prevents one. `decide`, `gotcha`, `map` and the distiller each take "highest + 1".

## LATER

- **Knowledge-health report.** Counts that matter to a team lead: drafts waiting and for how long,
  promoted-but-unverified notes past 90 days, Components with no decisions, recurring gotchas that
  should become Practices. Mostly queries over data the vault already has.
- **Behavioural evals for the prompt contracts** (`claude plugin eval`): `decide` leaves the
  predecessor alone, `gotcha` dedupes, `why` never quotes a draft, the auditor writes only its log.
  This is the largest unproven area. Expensive and non-deterministic, so it needs a plan first.
- **Import and export.** A portable archive of a project's promoted knowledge, and a way to move a
  project partition between vaults. Needed before any team use.
- **Team vaults.** One vault shared through git, with promotion attributed to a person. The
  Promotion Log already records who; the open questions are merge behaviour and permissions.
- **Upgrade safety.** A version stamp in the vault and a check that tells a user when their plugin
  and their vault disagree.
- **The remaining 14 workflows** as copy-paste prompts in `starter/`, promoted to skills only where
  real usage shows demand.
- **W10 Gotcha Guard** — a warning before editing a file with an open gotcha on it. Needs a third
  hook; the two-hook constraint stays until there is evidence a third earns its place.
- **W23 Post-Incident Capture** — an incident timeline distilled into gotchas.

## EXPERIMENTAL

- **Semantic retrieval as an opt-in second pass.** Exact path matching is why the brief is
  trustworthy, and also why it misses renamed paths. Embeddings would improve recall at the cost of
  "cannot invent a decision you never made". If built, it sits *beside* the deterministic brief,
  clearly labelled, never replacing it, and never reading drafts.
- **Cross-project practice extraction.** Recurring gotchas graduating into shared Practices
  automatically. Today it is a human judgement; automating it is only worth it if the false-positive
  rate can be measured.
- **Suggested promotions.** A deterministic score (confidence, evidence, age, recurrence) to order
  the review queue. Only worth building once real queues exist to learn from.

## REJECTED

- **A 13th skill for promotion.** The twelve-skill constraint is asserted by the validator, and a
  model-driven skill cannot be human-only. A shell command that needs a terminal can.
- **Promotion from inside Obsidian** (a community plugin). Breaks "zero community plugins", adds a
  second implementation of the draft rule, and the rule must live in one place.
- **Auto-promotion of high-confidence drafts.** Directly contradicts the product. Confidence is a
  model's opinion of itself; promotion is a human's decision. Not negotiable.
- **A database, a daemon, or an MCP server.** The product's advantage over heavier tools is that it
  is local, inspectable Markdown and shell. Nothing on this roadmap needs more.
- **A drafts hint that shows draft content in the brief.** Showing even a title crosses the
  quarantine. Counts only — and that is what shipped.
- **Ids or titles in the brief's draft line.** Considered, because it would help the user find them.
  Rejected: the line goes into a model's context, and the model can read any file it is pointed at.
  The user finds them with `tl-promote`, which is not in the model's context.
- **Dropping `open_threads` from the brief.** It is the product's best continuity feature ("last
  time you left the migration untested"). The risk is that nobody reviews it, so it is bounded and
  labelled rather than removed.
- **Fixing every vault problem on promotion** (renaming, retitling, merging duplicates). A
  promotion that does more than it says is a promotion nobody can trust. It edits the tag, the
  verification date, and a supersession, and it says so.
