---
name: "lean-spec"
description: "One-page spec + assumption test before building anything non-trivial. Use when a feature is proposed, scope is unclear, or someone says \"we should build X\" — especially accounts/login, notifications, new surfaces, or a platform change. Forces the riskiest assumption and a kill criterion into the open. Triggers on: should we build, new feature, scope, roadmap, MVP, prioritise, validate, is it worth it."
---

# Lean spec

<your-app> is pre-revenue and solo. The expensive mistake is not building slowly — it is
building the **wrong thing** well. Keep this to one page.

## 1. The spec (six lines, no more)

- **Problem** — the user's problem in one sentence, in their words, not ours.
- **Who** — which user: person managing their own records, family caregiver, or viewer.
- **Evidence** — what we actually observed. "A user said/did X." If this line is
  empty, say so plainly: *this is a guess.*
- **Smallest testable version** — the least we can build to learn.
- **Success** — the number that says it worked.
- **Kill criterion** — the result that makes us stop. Write it *before* building.

## 2. Surface the assumptions (VUBF, after Marty Cagan)

List what must be true, across four risks:
- **Value** — will they want it? will they switch?
- **Usability** — can they do it without help? (our users include older people)
- **Viability** — does it pay, or at least not cost us?
- **Feasibility** — can we build and run it?

Score each **Importance** (H/M/L) × **Evidence** (H/M/L).
**High importance + low evidence = test this first.** That is the riskiest assumption.

## 3. Cheapest test that could change our mind

Prefer, in order: ask 5 users → clickable prototype → fake door → thin real build.
Never jump to "thin real build" for a High/Low assumption.

State up front what result **validates** and what **invalidates**. If no result would
change the decision, do not run the test — just decide.

## 4. Your targets (example values: replace with your own; targets, not forecasts)

| Metric | Target |
|---|---|
| Update time after a doctor visit | ≤ 2 min |
| Still uploading after week 1 | ≥ 40% |
| Still active at month 3 | ≥ 25% |
| Free → paid | ≥ 5% |
| Extraction accuracy after confirmation | ≥ 98% |

Retention numbers like these are the kill metrics for a consumer health product. Treat them as the real gate.

## 5. Use it on the expensive-to-reverse calls

Accounts/login, payments and anything that drags in consent, roles and deletion duties
should pass through this skill before any build. They are assumptions until a user's
behaviour says otherwise.

## 6. Output

The six lines, the assumption table, the single riskiest assumption, and the one test
to run next. Recommend one option — do not present a survey.
