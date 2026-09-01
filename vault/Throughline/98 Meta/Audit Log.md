---
type: hub
status: active
project: global
created: 2026-08-16
updated: 2026-08-16
source: human
---

# Audit Log

Append-only record of `/throughline:audit` runs.

**The auditor flags; it never modifies a knowledge note.** It reads the vault against your real
codebase, writes its findings here, and stops. Nothing is tagged, edited, promoted, renamed, or
deleted on your behalf — you decide what each finding means.

Work the findings through [[Needs Review.base|Needs Review]].

> [!info] Why this file is append-only
> An audit is a dated observation about a moment in time. Rewriting it would destroy the record of
> what the vault claimed and when — which is the only way to see knowledge decaying rather than just
> being wrong today. Entries are added below; nothing above is ever edited.

---

*No audits yet. Run `/throughline:audit` in a repository that has been through `/throughline:init`.*
