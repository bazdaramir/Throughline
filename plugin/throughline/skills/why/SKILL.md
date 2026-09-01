---
name: why
description: Answer "why is it like this?" from the Decision history, walking the full supersession chain to report what is actually current. Use when the user runs /throughline:why, or asks why something was built a certain way, whether a choice was already made, or what the reasoning behind an approach was.
argument-hint: "<question>"
---

# Throughline · why

Answers "have we already decided this, and why?" **This skill writes nothing, ever.**

Its defining behaviour is honesty about absence. A confident answer assembled from decisions that do
not actually address the question is worse than no answer, because the user will act on it.

## Preconditions

**Resolve the vault** — `$THROUGHLINE_VAULT` → `vault:` in `./.throughline` → `~/Throughline`.
**Resolve the project slug** from `./.throughline`; if absent, say so and stop.

A question is required. With no argument, ask what they want to know.

## Retrieval rules — binding

- **Read only promoted notes. Skip every note tagged `#tl/draft`.** Drafts are unreviewed agent
  output; surfacing one as established reasoning is exactly the failure the quarantine exists to
  prevent. If a draft *would* have answered the question, say that a draft exists and needs
  promoting — but do not quote its content as fact.
- Search `Decisions/` for the current project. Widen to other projects only if the question is
  explicitly cross-project, and label anything from another project clearly.
- Practices in `03 Practices/` are in scope when the question is about a convention rather than a
  one-off choice.

## Steps

1. **Find candidate Decisions** by matching the question against titles, `## Decision`, `## Context`,
   and `affects`.

2. **Walk the supersession chain to its end.** For every match, follow `superseded_by` forward until
   a note has none. **The last note in the chain is the current state, and that is what you report
   first.** Reporting a superseded decision as current is the worst thing this skill can do.

   - Chain loops (A superseded_by B, B superseded_by A) → report the cycle as a data error and stop
     following. Do not recurse.
   - `superseded_by` points at a note that does not exist → report the broken link; treat the note
     as current but say the chain is damaged.
   - **`superseded_by` points at a note that is still `#tl/draft`** → the chain's promoted end is the
     last *promoted* note. Report that one as current, then **disclose that an unpromoted draft
     claims to supersede it** — name it and say it needs review. Do not quote its content as fact.
     Both halves matter: saying a decision is current while an unreviewed reversal of it sits in the
     queue is misleading, and quoting the draft as settled would break the quarantine.

3. **Classify what you found, and say which:**
   - **Answered** — a decision directly addresses the question.
   - **Related but not answering** — say so explicitly. Name the decision, state what it *does*
     cover, and say plainly that it does not answer what was asked. **Do not extrapolate from it.**
   - **Nothing** — say nothing exists. Do not soften it, do not assemble an answer from adjacent
     material, do not reason from the code. Offer `/throughline:decide` to capture it now.

4. **Answer.**

## Output contract

For each relevant decision:

- **ID and title**, as a wikilink so the user can jump to it
- **Status**, and if superseded, the chain: `D-0002 → D-0004 (current)`
- **When** it was decided
- **The reasoning** — from `## Context`, not paraphrased into vagueness
- **The alternatives considered**, which is usually the most useful part
- **`reversal_cost`** when set — the user needs to know what contradicting it costs
- **`Revisit when`**, if its trigger now appears to be met, flag that

Keep it proportionate. One decision, answered clearly, beats five listed.

If several *active* decisions compete on the same question, **report all of them and say they
conflict.** Do not pick a winner — that is a finding, and it belongs in the user's hands.

## Constraints

- **Writes nothing. Creates nothing. Modifies nothing.** No exceptions.
- Never read `#tl/draft` notes as evidence.
- Never extrapolate. "Nothing exists" is a complete and correct answer.
- Never present a superseded decision without its successor.
- Never infer the reasoning from the code. This skill reports what was *written down*; if the vault
  does not know, the answer is that the vault does not know.
