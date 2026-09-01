---
name: init
description: Make the current repository Throughline-aware — resolve the vault, write the .throughline pointer file, and create the project partition. Use once per repo, or when a Throughline command reports the repo is not initialised.
argument-hint: "[project-slug]"
---

# Throughline · init

Makes this repo Throughline-aware. **Ask at most one question. Print exactly one next step.**
Target: under sixty seconds.

## Preconditions

**Resolve the vault**, in this order — never guess a path:

1. `$THROUGHLINE_VAULT` (`%THROUGHLINE_VAULT%` on Windows)
2. the `vault:` line in `./.throughline`, if the file already exists
3. `~/Throughline`

A directory is a valid vault only if it contains `00 Command/` and `90 Templates/`. If none
resolve, say this and stop:

> No Throughline vault found. Point `THROUGHLINE_VAULT` at your vault directory (the folder
> containing `00 Command/`), then run `/throughline:init` again.

**Do not create a vault.** The vault ships with the product; inventing one produces a broken
half-vault with no templates or dashboards.

## Steps

1. **If `./.throughline` already exists** — read it, confirm the project and vault it names still
   resolve, print the confirmation, and stop. This skill is idempotent; running it twice changes
   nothing.

2. **Determine the project slug**, in order:
   - the argument, if given
   - the basename of `git remote get-url origin`, minus `.git`
   - the current directory name

   Lowercase it and hyphenate it. If the derived slug is ambiguous or unusable, **this is the one
   question you may ask.** Otherwise do not ask — state the slug you chose in the output.

3. **Check for a partition collision.** If `<vault>/01 Projects/<slug>/` already exists and is
   non-empty, stop and ask. Never merge into an existing project's partition.

4. **Create the partition**: `<vault>/01 Projects/<slug>/` with subfolders `Components/`,
   `Decisions/`, `Gotchas/`, `Specs/`, `Sessions/`.

5. **Write `<vault>/01 Projects/<slug>/_Project.md`.** There is no template for this note — it is a
   hub, not one of the seven knowledge types. Match this shape exactly:

   ```markdown
   ---
   type: hub
   status: active
   project: <slug>
   created: <YYYY-MM-DD today>
   updated: <YYYY-MM-DD today>
   source: human
   stack: []
   repo: <git remote url, or omit the line entirely if there is no remote>
   ---

   # <slug>

   ## Charter
   *[One paragraph: what this codebase is for. Write it yourself — it is the one thing an agent cannot infer.]*

   ## Stack
   *[Fill `stack:` above and summarise here.]*

   ## Map
   *No components mapped yet. Run `/throughline:map <dir>` to add one.*

   ## Live work
   *Nothing yet.*

   ## Standing constraints
   *Nothing yet. Decisions with `reversal_cost: irreversible` belong here once you have them.*
   ```

6. **Write `./.throughline` in the repository root** — not in the vault. Exactly:

   ```yaml
   # .throughline — commit this
   project: <slug>
   vault: <absolute path to the vault>
   ```

   Always write the `vault:` line explicitly. The `~/Throughline` fallback is frequently wrong —
   the vault often lives elsewhere — and a wrong fallback fails silently later.

7. **Offer the `CLAUDE.md` conversion. Never apply it.** If `./CLAUDE.md` exists and is longer than
   ~50 lines, show the router it *could* become and ask whether to write it. If the user does not
   clearly agree, leave the file untouched.

   ```markdown
   Project knowledge lives in the Throughline vault at `<vault>/01 Projects/<slug>/`.

   **Do not read the whole vault.** Retrieve selectively:
   - Before changing anything: `/throughline:brief <area>`
   - To ask why something is the way it is: `/throughline:why <topic>`
   - Active decisions:  `01 Projects/<slug>/Decisions/`  (status: active)
   - Open gotchas:      `01 Projects/<slug>/Gotchas/`    (status: open)
   - Component maps:    `01 Projects/<slug>/Components/`
   - Domain language:   `02 Domain/`
   - Conventions:       `03 Practices/`

   ## Hard rules (only things that must ALWAYS be in context)
   - [move only the genuinely always-true rules here]
   ```

8. **Print one next step.** It is `/throughline:map <your main source dir>` — mapping a Component
   first, because `decide`, `gotcha`, `spec`, and `brief` all resolve against Components and are
   less useful until at least one exists.

   `/throughline:backfill --since 90d` is the natural second step, but do not volunteer it here.
   Mapping one Component first gives the backfiller something to attach its output to.

## Output contract

Report, in four lines: the vault path used, the project slug, what was created, and the single next
step. Nothing else. If `CLAUDE.md` conversion was offered and declined, say so in one clause.

## Constraints

- **Max one question**, and only for an ambiguous slug.
- Never create a vault. Never guess a vault path.
- Never overwrite an existing `.throughline`, `_Project.md`, or partition.
- Never modify `CLAUDE.md` without explicit agreement in this conversation.
- Never invent a slug the user did not imply.
- Do not create `Home.md`, dashboards, or templates — those ship with the vault.
