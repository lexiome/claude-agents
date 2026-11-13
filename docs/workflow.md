# How the team works (shared protocol)

The one idea: **the repo is a shared notice board.** Roles don't message each other — they read notes off the board and pin their own notes back. The board is `docs/`, which git-syncs, so it's the same on every device.

## Where things live (the board sections)

| Section | What's stored there |
|---|---|
| `docs/workstreams/cto/` | Build tickets — engineering jobs (`NNN-slug.md`) |
| `docs/workstreams/scientist/` | Science ideas the scientist proposes (`sci-NNN-*.md`) |
| `docs/planning/market_analysis/` | Market notes (competitors, pricing, weekly research) |
| `docs/planning/product/` | Product specs (`spec-NNN-*.md`) — owned by the PM |
| `docs/planning/business/` | Strategy & finance |
| `docs/decisions/` | "We decided X" records (ADRs) — so nothing gets re-argued |
| `docs/automation-log.md` | The diary — one line every time a role does something |
| `docs/roles/` + `.claude/agents/` | Who the roles are |

## The golden rule (every role, every time)

- **Before you start:** read the **tail of** `docs/automation-log.md` (the last ~40 lines — recent activity) + the recent items in your own section + `docs/decisions/`.
  - **Read the log ONCE.** Do not read the whole file, and never re-read overlapping line ranges — that is pure waste. One tail read per run.
  - Older history lives in `docs/automation-log-archive.md`. Open it **only** if the tail genuinely does not answer your question. Ask: *has this already been done? was it already decided?* Don't repeat work; don't re-propose settled calls.
- **Work only the delta:** look at what changed since your last run (your cursor in the log / the last week), not everything.
- **When you finish:** append **one line** to `docs/automation-log.md`. If your output needs another role to act, **file a ticket in their section** and reference it.
  - **Log entries are max ~300 characters:** *what you did + the outcome + the ticket/PR number.* That's it. The full reasoning, findings, and detail belong in the **ticket** (or PR/ADR), never the log. The log is an index that points at the record — not the record itself. A 300-char line that says "did X, result Y, see CTO-NNN" beats an 80-line essay every time.

## The handoff chain (how a ticket flows, and how the loop closes)

```
Scientist / Market   →  raise an idea / finding (sci-NNN / market note)
   →  Product Manager  prioritizes it + writes a spec (what / why / success / scope)
   →  CTO triages the spec → spawn cto-NNN build tickets (link back),  or reject with a reason
   →  Developer     implements → opens PR
   →  Code-reviewer + Security  (+ Clinical — REQUIRED when the change affects extraction correctness)  review
   →  Eval/QA        confirms the accuracy gate (no regression)
   →  CTO            sets status: approved
   →  Human (founder)   does the final merge
   →  ticket marked DONE + one line in the diary          ← loop closed
```

Nobody messages anybody. Every step is: **read the board → do my bit → write to the board.**

**Clinical is not optional on correctness.** Any ticket that changes *what* gets extracted or *whether an extraction is right* (fields, mappings, dose/name parsing, ICD/SNOMED, gold-set labels) requires a `@clinical-validator` sign-off before `@cto` approves — not a cc. It is an AI aid, not a licensed clinician, but no correctness change ships without a clinical read.

## Answering "where are we / what's left" (don't crawl)

- **Status/planning questions → read `ROADMAP.md` (Now/Next) + grep for open tickets** (`git grep -l "status: open" docs/workstreams/`). Do NOT read every file to reconstruct state — ROADMAP *is* the status view (the PM keeps it current); grep finds the rest cheaply.
- **Archive closed work:** when a ticket is `done`, move it to `docs/workstreams/<role>/done/`. The active folder stays small, so the rare full scan is tiny. Same principle as the cursor jobs — read the index/delta, not everything.

## Tech-design review (the trigger)

Not every ticket needs a design review. Convene one only when a ticket picks a model, method or architecture, or materially changes compute or cost. Then the scientist critiques efficiency, eval-qa critiques accuracy, and the CFO critiques cost, BEFORE the ticket is approved to build. Record the call as an ADR in `docs/decisions/`. Routine tickets (bug fixes, plumbing, small changes) skip this.


## Definition of done + cleanup (so nothing piles up)
A ticket is **not done** until all four: (a) status set, (b) moved to `done/` (via `just close`), (c) one log line, (d) committed. Three of four is not done.

Cleanup has explicit owners — it is not "someone, later":
- **@product-manager, weekly:** triage every open `sci-*` ticket — promote to a spec, park with a
  reason, or close. Open tickets do not accumulate unbounded. **"Done" for a research ticket means
  its finding has landed** in an ADR, a spec, or a CTO ticket (shipped or still open and tracked) —
  close it via `just close scientist <id> done` with a one-line "superseded by X" note under its own
  `## Outcome` section; the CTO/ADR ticket carries the work forward, the sci ticket doesn't need to
  stay open to prove the finding happened. If genuinely nothing has consumed it yet, park it open
  with a stated reason (why it's not being acted on, and what would trigger picking it up) rather
  than closing it. Cap: if open `sci-*` count exceeds 15, this triage runs before that week's other
  PM work.
- **Monthly audit:** auto-archive done/approved tickets, delete flagged orphans, verify `automation-log.md` is committed. Anything needing judgment → **file a ticket**, not a log line.
- **Every role:** to close a ticket, run your close script (e.g. `just close <role> <id> <status>`): it sets the status, moves the ticket to `done/` and logs it in one step.


## Twice = ticket (never fix the same thing twice)
- **First time** a problem happens (a deploy fails, a test breaks, a bug shows up) → fix it.
- **Second time the SAME problem happens** → STOP. Don't just patch it again. File a **root-cause review ticket** (`@cto` for build/deploy/test/infra, `@product-manager` for planning issues).
- The best fix turns it into a **guard** — a test, a CI check, or a hook — so it can't come back. Add the guard, then delete any prose rule it replaces (see `docs/lessons.md`).
