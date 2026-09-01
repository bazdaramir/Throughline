---
name: spec
description: Draft a Spec note for upcoming work, auto-populating its constraints from the decisions that govern the affected components and its landmines from the open gotchas there, and flagging any conflict with an active decision. Use when the user runs /throughline:spec or is planning work they intend to hand to an agent.
argument-hint: "<objective>"
---

# Throughline · spec

Drafts work that already accounts for everything previously learned. **A spec written with the
existing decision history embedded is a fundamentally different artefact from one written from
memory** — and it is the thing the user could not have produced in a notes app, no matter how
disciplined.

## Preconditions

**Resolve the vault** — `$THROUGHLINE_VAULT` → `vault:` in `./.throughline` → `~/Throughline`.
**Resolve the project slug** from `./.throughline`; if absent, say so and stop.

An objective is required.

## Retrieval discipline — this skill reads before it writes

`spec` is a capture skill, but `Constraints` and `Known landmines` are *retrieval*. The retrieval
rules therefore bind here too:

- **Exclude every `#tl/draft` note.** A spec grounded in unpromoted guesses is worse than an
  ungrounded one, because it looks authoritative.
- Only `status: active` Decisions and `status: open` Gotchas.
- **Quote the source notes; do not paraphrase.** A paraphrased constraint loses the precision that
  made it worth recording, and the reader cannot tell what was actually decided.

## Steps

1. **Resolve `affects`** — match the objective against Component `paths` and subject matter. If it
   resolves to nothing, say the spec will be ungrounded and ask whether to continue; do not silently
   produce an empty-constraint spec that *looks* grounded.

2. **Gather constraints.** Active Decisions whose `affects` names those Components. Quote the
   `## Decision` line of each and cite it by wikilink. Add relevant Practices from `03 Practices/`.

3. **Gather landmines.** Open Gotchas whose `affects` names those Components. Give the symptom and
   the one-line workaround, cited by wikilink.

4. **Detect conflict.** Does the objective contradict an active Decision?
   - **Distinguish contradicting from extending.** Extending a decision is normal and needs no flag.
     Contradicting one is a fork in the road.
   - On a genuine conflict: set `conflicts_with` on the **new spec** — never modify the Decision —
     and **say so prominently in your output.** Never proceed quietly past a contradiction.
   - If the conflicting Decision is `reversal_cost: irreversible`, escalate loudly: the user should
     either abandon the objective or consciously supersede the decision first.

5. **Allocate the ID** — highest `S-NNNN` plus one, zero-padded. Filename `S-NNNN <Imperative title>.md`.

6. **Write the note** from `90 Templates/Spec.md`.

## Output contract

```yaml
---
type: spec
status: draft
project: <slug>
created: <today YYYY-MM-DD>
updated: <today YYYY-MM-DD>
source: agent
affects: ["[[C-0002 Session store]]"]
tags:
  - tl/draft
---
```

Optional, when true: `depends_on` (quoted wikilinks to the Decisions this rests on),
`conflicts_with`, `risk: low | medium | high`, `estimate`. **Never set `last_verified`.**

Sections, in this exact order:

- `## Objective` — one sentence. **Outcome, not activity.** "Postgres is the authoritative session
  store and revocation is synchronous", not "migrate the session store".
- `## Constraints` — auto-populated. Quoted, cited, one bullet each. Keep the
  `<!-- auto-populated ... -->` comment above them.
- `## Known landmines` — auto-populated from open Gotchas. Symptom plus workaround, cited. Keep the
  comment.
- `## Approach` — the plan, written to be handed to an agent. Numbered steps.
- `## Out of scope` — **mandatory, and never empty.** This is the single most effective
  anti-scope-creep instrument available: agents expand scope by default, and an explicit negative
  boundary is the cheapest possible correction. Include the standing prohibitions that apply —
  anything under an `irreversible` decision belongs here.
- `## Done when` — testable criteria, as a checklist.

Report the path, the constraints and landmines you pulled in (so the user can see the vault working),
any conflict — prominently — and the quarantine notice.

## Constraints

- **Never modify a Decision, Gotcha, or Component.** `conflicts_with` goes on the new spec only.
- Never overwrite an existing Spec. A near-identical objective already specced → report it and stop.
- Never leave `Out of scope` empty.
- Never include draft, superseded, or resolved notes in the auto-populated sections.
- Never paraphrase a constraint you could quote.
- Never remove a `tl/draft` tag.
