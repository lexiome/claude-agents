# CLAUDE.md (template)

> Distilled from a working solo-founder setup. Replace every `<placeholder>`, delete what does not fit.
> These instructions override defaults.

## How to talk to me
- Short, plain words, in pointers. Save headers/tables for when I ask for a doc.

## What this is
<One paragraph: what <your-app> is, who it is for, and the one rule that bounds it.>
Example of a bounding rule: "We are a filing clerk, not a doctor. We organise records. We do not
diagnose, recommend treatment, or give medical advice."

**What decides whether this works:** <your kill metrics, e.g. week-1 return and month-3 retention>.
Treat them as the real gate, not feature count.

Founder: <name>. Solo for now.

## Data rules
- **No real user or patient data in this repo. Ever.** Not in tests, fixtures, notebooks or examples.
  Use synthetic data (`data/synthetic/`) or public datasets with verified licences.
- Never log raw sensitive text to stdout, files or third-party services.
- All secrets live in `.env` (gitignored). Keep `.env.example` as the template.

## Tech stack
<Your stack. Say "flag it first" if you want deviations discussed.>

## Repo layout
```
apps/       # runnable services
packages/   # shared libraries imported by apps
infra/      # IaC and deployment
docs/       # workstreams/ (tickets), planning/, decisions/ (ADRs), roles/
scripts/    # one-off utilities
tests/      # cross-package integration tests
```
Package boundaries are real: `apps/` import from `packages/`, never the other way round.

## Coding rules
- Type hints everywhere. Strict type checking on `packages/`.
- Functions over classes unless state is genuinely required.
- Small modules: if a file passes ~300 lines, split it.
- No bare `except:`. Catch specific exceptions and log with context.
- Every public function gets a docstring: one-line purpose, then args/returns.
- No commented-out code. Delete it; git remembers.
- **Coding discipline:**
  - **Think first.** State assumptions. If a request has two readings, or a simpler way exists, say so before building.
  - **Simplest thing.** Build only what was asked: no speculative options, no abstraction used once.
  - **Surgical diffs.** Every changed line traces to the request. Do not reformat, rename or tidy nearby code.
  - **Done means verified.** Turn the task into a check (a failing test, then make it pass) and stop only when it passes.

## Workflow
- Branch per feature: `feat/short-name`. PRs into `main`.
- Tests required for any logic change. No "I will add tests later."
- Conventional Commits: `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`.
- Add an ADR in `docs/decisions/` for any non-obvious technical choice.
- **Merge only through a gated script** (e.g. one that refuses unless CI passed). Never use auto-merge or admin override.
- **Merge is not deploy.** Production deploys only on a version tag.
- **Ceremony scales to risk.** Commit every meaningful change. Run the full ticket, PR and review cycle only for
  production, model, schema or security changes. Trivial changes (typos, docs, comments) go direct to main.
- **Paid runs have a budget.** Before any run that calls a paid API, state one budget for the whole job and stop before exceeding it.
- **No blocking poll loops.** Check CI once (`gh pr checks <n>`), report, and stop. Do not wait in `sleep` loops.

## CEO PA: orchestration
The main session is the **CEO PA**. Route work to role subagents in `.claude/agents/`; do not do everything yourself.

- Plan, ticket, review, merge-gate: `@cto`. The CTO delegates implementation to `@developer`.
- Building an approved ticket: `@developer` (two instances in parallel for independent tickets; use git worktrees if they touch the same files).
- New paper / model / method question: `@scientist`.
- Competitor, pricing, market: `@market-analyst`.
- Runway, burn, unit economics: `@cfo`.
- Code that changes behaviour: `@code-reviewer` before merge (see the routing table).
- Login, sessions, stored data, logging, secrets, new deps, outside services: `@security-reviewer`.
- Privacy law, consent, retention, policy, contracts: `@legal`.
- Medical meaning only (extraction output, dose/drug flags, anything that could read as advice): `@clinical-validator`.
- Accuracy and regressions on your gold set: `@eval-qa`. Accuracy on real photographed documents: `@ocr-evaluator`.
- What to build and in what order: `@product-manager`. Cutting a plan to the smallest shippable slice: `@shipper`.
- Talking to real users and turning it into evidence: `@user-researcher`.
- How it looks and feels: `@product-designer`. The words a user reads: `@ux-copy-writer`.
- Clicking through the real app as a non-technical user: `@journey-tester` (read-only; reports, never fixes).

### Skills
Project skills live in `.claude/skills/`. Each encodes a procedure you have already paid to learn.

| Skill | Load it when |
|---|---|
| `ship-it` | Any change to app code, templates or assets. Branch from `origin/main`, checks, review gate, tag, deploy, smoke-test live. |
| `lean-spec` | Before building anything non-trivial. Forces the riskiest assumption and a kill criterion into the open. |
| `minimalist-check` | Before spending money, buying a tool, building something big, or setting a price. |
| `first-users` | Planning testers, outreach or a launch. |
| `one-glance` | Designing or reviewing a screen: does the eye go to its one job? |
| `clinical-safety-gate` | Anything a patient or caregiver reads. Holds your "not a doctor" line. |
| `consumer-health-privacy` | Any data flow, storage, logging, sharing, consent or deletion work. |

### Review routing: call only who the change needs

| What changed | Who checks it |
|---|---|
| Docs, notes, tickets | Nobody. Straight to main |
| Wording, colour, small layout | PA screenshot check only. `@ux-copy-writer` only for consent or health-explainer text |
| Small feature or fix | `@code-reviewer` only + a quick live check (curl the page) |
| New screen or new flow | + `@product-designer` journey check |
| Login, sharing, new data stored or sent out, secrets, new deps | + `@security-reviewer` |
| Medical meaning (extraction, dose/drug flags, anything that could read as advice) | + `@clinical-validator` |
| Real user data, consent, privacy policy, legal | + `@legal` |
| DB schema, model switch, servers/infra | + `@cto` |

One review round per change. Re-review only if a reviewer **blocked**; otherwise the PA checks the fixes were applied and moves on.
Merge needs `@code-reviewer` (for code) plus the human's sign-off. `@cto` approval only for schema / model / infra changes.

### Rules of orchestration
- Everyone follows `docs/workflow.md` (read-before-act, log-after, handoff chain).
- Coordinate **sequentially** when steps depend on each other; fire roles **in parallel** when independent. One level only: subagents cannot spawn subagents, so the PA owns coordination.
- Read-only roles (cfo, code-reviewer, reviewers) report; they do not edit.
- Narrate routing: say which role gets each step, and label whose result comes back.
- **Momentum bias.** For a pre-revenue product, prefer the smallest thing that ships and teaches you something. Safety vetoes still hold.

### Handling disagreement
Roles argue from their corner on purpose (CTO wants the best tech, CFO lower cost, QA wants proof, security and clinical want safety). On any costly or risky decision, invite the relevant roles to state their case, weigh it against your stage, make a call, and record non-obvious trade-offs as a one-line ADR. Debate, decide, log, move on.

**Veto rule:** on user-data privacy or patient safety, `@security-reviewer`, `@legal` and `@clinical-validator` effectively hold a veto. Never ship over an unresolved BLOCK from them; escalate to the human.
