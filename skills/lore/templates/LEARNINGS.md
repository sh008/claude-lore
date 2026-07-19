# LEARNINGS — <project name>

> Traps, failed approaches, and dead ends. The most valuable file here: it is the
> only one that stops a fresh session from re-paying a cost that was already paid.
>
> Every entry needs a `Trigger:` line naming the file, module, or tool it fires on —
> future sessions find these by grep, not by reading top to bottom:
>
>     grep -i -A6 "<keyword>" docs/lore/LEARNINGS.md
>
> Write the negative result plainly. "This did not work" is the whole point.

---

## <YYYY-MM-DD> — <the lesson, stated as a rule>

**Trigger:** <filenames / module / library / command this applies to — be generous
with keywords, this is the search index>

**What happened:** <the symptom, concretely — the error, the wrong output, the hang>

**Cause:** <the real cause, once found>

**Do instead:** <the working approach>

**Don't retry:** <the specific thing that looks reasonable but is already known to
fail — name it explicitly so it isn't attempted again>

---
