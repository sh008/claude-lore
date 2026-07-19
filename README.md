# Lore

**The knowledge your codebase doesn't contain — decisions, dead ends, and state, recorded so the next session starts where the last one stopped.**

A Claude Code skill. Also works with any agent that reads `SKILL.md` files.

---

## The problem

You open a new session on a project you've been building for weeks. The agent has no memory of any of it, so it does the only thing it can: reads the repo. Ten minutes and a lot of context later, it knows *what the code is* — and still doesn't know:

- that the obvious-looking library was tried in week two and abandoned because it breaks on Windows paths
- that the odd-looking retry loop is odd on purpose, and removing it reintroduces a race
- that the thing you asked about last week is 80% done, and exactly where it stopped
- why the schema is denormalized

None of that is in the code. It was in a conversation that no longer exists. So the agent re-derives it — and sometimes re-derives it *wrong*, retrying an approach you already paid an hour to rule out.

Lore fixes this by writing that knowledge down as the work happens, in the repo, next to the code.

## What it does

Five files under `docs/lore/`, each with one job:

| File | Holds | Read when |
|---|---|---|
| `STATE.md` | Current status: what works, what's mid-flight, blockers, next steps. **The entry point.** | Every session start — fully |
| `ROADMAP.md` | Phased progress path with checkboxes and "done when" conditions | Session start; picking the next task |
| `DECISIONS.md` | What was chosen, why, and **what was rejected** | Before architectural changes; "why is it like this?" |
| `LEARNINGS.md` | Traps, failed approaches, dead ends — each with a `Trigger:` line | Grepped before touching an area |
| `JOURNAL.md` | Append-only dated work log | Rarely — to reconstruct a specific past session |

And it defines the discipline that keeps them from rotting: when to read each one, when to write, what's worth recording, and what isn't.

## Install

### Option 1 — Plugin marketplace (recommended)

```
/plugin marketplace add sh008/claude-lore
/plugin install lore@claude-lore
```

Updates come with `/plugin update`.

### Option 2 — Personal skill (available in every project)

```bash
git clone https://github.com/sh008/claude-lore.git
mkdir -p ~/.claude/skills
cp -r claude-lore/skills/lore ~/.claude/skills/
```

On Windows, `~/.claude/skills` is `C:\Users\<you>\.claude\skills`.

### Option 3 — Project skill (committed with one repo, shared with your team)

```bash
git clone https://github.com/sh008/claude-lore.git
mkdir -p .claude/skills
cp -r claude-lore/skills/lore .claude/skills/
```

No dependencies, no build step, no scripts — it's markdown all the way down.

## Usage

### Starting a project

```
/lore
```

or just *"set up lore for this project"*. It will:

1. Look for a knowledge file you already keep (`docs/STATUS.md`, `NOTES.md`, a `## Status` section in the README) and **migrate it rather than creating a competing second source of truth**
2. Scaffold `docs/lore/` from the templates, filled in from what's actually known
3. Build `ROADMAP.md` as phases, each with an observable "done when"
4. **Append a session protocol block to your `CLAUDE.md`**

Step 4 is the one that matters. The files are inert on their own — the protocol block in `CLAUDE.md` is what tells every future session to read them. Skip it and you've written documentation nobody opens.

### Every session after that

Just start working, or say *"continue"*. The agent reads `STATE.md` and `ROADMAP.md`, greps `LEARNINGS.md` for whatever it's about to touch, and tells you where things stand — without crawling the codebase.

Next steps in `STATE.md` are marked `▶️` (actionable now) or `⛔ [needs user]` (blocked on an API key, a judgement call, something only you can run or see). The agent starts the first `▶️` instead of asking you to pick, and mentions the `⛔` ones once as reminders. Without that distinction it stalls on a question the file could already answer — which is how v1.0.1 came about.

### Recording

Automatic at the end of each task, per the protocol block. You can also be explicit: *"record that"*, *"log this decision"*.

The timing is deliberate: **record at the end of each task, not batched at session end**, where a context cutoff silently loses the whole day's learnings.

## What a real entry looks like

The template fields exist to force out the part that's actually useful. A learning:

```markdown
## 2026-03-14 — Vite dev server must bind 0.0.0.0 for the Docker healthcheck

**Trigger:** vite.config.ts, docker-compose.yml, healthcheck, "connection refused"

**What happened:** Container healthcheck failed with ECONNREFUSED while
`curl localhost:5173` worked fine from inside the container.

**Cause:** Vite binds 127.0.0.1 by default. The healthcheck hits the container IP,
not loopback.

**Do instead:** `server.host: '0.0.0.0'` in vite.config.ts.

**Don't retry:** Raising the healthcheck `start_period`. It looks like a timing
issue and it isn't — spent 40 minutes there.
```

That last field is the point of the whole project. `Don't retry:` is the line that stops the next session — human or agent — from walking into the same forty minutes.

And a decision:

```markdown
## 2026-03-02 — Server-side sessions, not JWTs

**Chose:** Redis-backed sessions with an opaque cookie.

**Why:** Instant revocation is a hard requirement (support needs to kill a session
on a stolen-device report). Redis is already deployed for the job queue.

**Rejected:** JWTs — revocation needs a denylist, which is the session table again,
with worse ergonomics and a token that stays valid until expiry.

**Affects:** src/auth/, middleware/session.ts

**Revisit if:** we ever need to authenticate across services that can't reach Redis.
```

## Design notes

A few choices that are load-bearing:

**`STATE.md` has a hard ~150-line cap.** It's the only file read in full every session, so it has to describe the *present*. Anything that becomes past tense gets moved out. Without the cap it grows into exactly the wall of text you were trying to avoid reading — a knowledge base that costs as much to load as the codebase has failed at its one job.

**Every learning carries a `Trigger:` line** naming the files, modules, or error strings it applies to. Nothing reads `LEARNINGS.md` top to bottom; it gets grepped:

```bash
grep -i -A6 "healthcheck" docs/lore/LEARNINGS.md
```

An entry nobody can find is an entry that doesn't exist. The `Trigger:` line is the search index, so it's written to be generous with keywords.

**Field keys stay English, prose doesn't have to.** Write entries in whatever language you work in — `Trigger:`, `Why:`, `Rejected:` stay English so grep keeps working across a mixed-language repo.

**Failed approaches are the highest-value entries**, above decisions and far above status. A decision can often be inferred from the code that implements it. A dead end leaves no trace at all — the only evidence it ever happened is the record you wrote.

**There's an explicit "don't record this" list.** Skills that record everything produce a file that's expensive to read and mostly restates the diff. The test is: *would a competent developer with full repo access still not know this?* If `git log` covers it, it doesn't go in.

**Stale entries get corrected or deleted, not left.** Recorded knowledge is trusted knowledge; a wrong entry is worse than a missing one. `STATE.md` claims are treated as claims to verify cheaply, not as ground truth.

## How this differs from session-handoff tools

There are good tools for surviving a context cutoff — [claude-handoff](https://github.com/REMvisual/claude-handoff), `transfer-context`, and similar. They snapshot a conversation so the *next* conversation can pick it up.

Lore is solving an adjacent problem. A handoff is a message from one session to the immediate next one, and it ages out. Lore is **cumulative and permanent**: month-three you benefits from a dead end you hit in week two. It's committed to the repo, so it survives machine changes, agent changes, and teammates — and a human onboarding onto the project gets the same value from it that the agent does.

They compose fine. Use a handoff tool for mid-task compaction; use lore for what the project has permanently learned.

It's also not personal/global agent memory (preferences, your habits across all projects). Lore is per-project, in the project, versioned with it.

## Contributing

Issues and PRs welcome. The templates are opinionated on purpose, but if a field consistently goes unfilled in real use, that's a bug worth reporting — every field is supposed to earn its place.

## License

MIT — see [LICENSE](LICENSE).
