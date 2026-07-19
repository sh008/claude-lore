# Protocol block

Append this to the target project's `CLAUDE.md` during init (create the file if absent),
adjusting the project name and paths. Without it, a future session has no instruction to
read the lore — the files exist but nothing points at them.

Translate the prose into the language the project's docs use; keep the paths and the
field keys as-is.

---

```markdown
## Session protocol (mandatory)

`docs/lore/` is this project's knowledge base. It exists so a session starts from
recorded state instead of re-reading the repository.

**At session start:**
1. Read `docs/lore/STATE.md` and `docs/lore/ROADMAP.md` in full.
2. Before touching an area, grep it for known traps:
   `grep -i -A6 "<area>" docs/lore/LEARNINGS.md`
3. Open only the files the task needs. Do not rebuild context by reading the codebase —
   that is what these files replace.

**Before changing anything architectural:** check `docs/lore/DECISIONS.md` for why the
current shape was chosen. If you are about to reverse a recorded decision, say so
explicitly to the user first.

**At the end of each task, before reporting it done:**
1. Update `docs/lore/STATE.md` — status, its date, and move finished items out.
2. Append a dated entry to `docs/lore/JOURNAL.md`.
3. If a choice was made between real alternatives → `docs/lore/DECISIONS.md`
   (include what was rejected and why).
4. If time was lost to a trap or a failed approach → `docs/lore/LEARNINGS.md`
   (include a `Trigger:` line so it can be found by grep).
5. If a phase advanced → tick the box in `docs/lore/ROADMAP.md`.

Record outcomes, not plans. Use absolute dates. Do not record what the code or
`git log` already says. If a recorded fact turns out to be wrong, correct or delete
it — a stale record is worse than none, because it gets trusted.
```
