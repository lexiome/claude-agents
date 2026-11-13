---
name: "ship-it"
description: "<your-app>'s build-and-deploy loop. Use for ANY change to app code, templates, or static assets — from first branch to verified-live. Prevents the two failures that have actually bitten us: branching from a stale local main (nearly reverted a whole redesign) and reporting \"done\" without hitting the live URL. Triggers on: build, implement, fix, ship, deploy, release, PR, merge, tag, hotfix."
---

# Ship it

Follow in order. Do not skip steps 1, 7, or 9.

## 1. Branch from `origin/main` — never from local main

Local `main` drifts. It has been **15 commits behind** while also ahead. A PR built on
stale main once carried a large deletion that would have reverted a redesign.

```bash
git fetch origin -q
git worktree add -b feat/<short-name> <scratch-dir>/<name> origin/main
```

Work **in that worktree**. Never edit the primary working dir for a feature.
Sanity-check before trusting any file you are about to edit:
```bash
git rev-list --left-right --count origin/main...HEAD   # expect 0  0
```

## 2. Smallest change that ships

One feature per branch. If it needs a decision, use the `lean-spec` skill first.

## 3. Safety gate (if user-facing)

Any patient-visible feature, label, or health copy → run the **`clinical-safety-gate`**
skill. Any data flow / storage / sharing / logging change → run
**`consumer-health-privacy`** and route `@security-reviewer`.

## 4. Local checks — all of them, fix everything (example commands: use your own stack)

```bash
.venv/Scripts/python -m ruff check packages/ tests/ scripts/
.venv/Scripts/python -m ruff format --check packages/ tests/ scripts/
.venv/Scripts/python -m mypy packages/
.venv/Scripts/pytest.exe tests/ -q
```
Front-end-only change (HTML/CSS/images): these do not apply — instead grep for tests
asserting on the markup you changed, and render the page to check it.

## 4b. Journey check: new screens and new flows only

A **new screen or new flow**: **@product-designer** walks the relevant journeys in
`docs/planning/product/key-journeys.md` at a true 390px as a careful 60-year-old first-time user
(taps, seconds, screenshots). Over 60 s or over the tap budget, a dead end, low contrast or a tap
target under 48px fails the change. Wording, colour or small layout tweaks: the PA checks a
390px screenshot instead (founder rule).

## 4c. Design consistency gate: EVERY UI change (founder rule, non-negotiable)

Before review, take a 390px screenshot of the changed screen next to a neighbouring screen that
already follows `docs/planning/product/design.md`. Check: font, title/subtitle/label/row text
sizes, row height and padding, card/segment and divider style, borders, button shape, spacing.
Any mismatch fails the change. No one-off sizes, paddings, radii, borders or colours in page
CSS: use the shared tokens. Missing rule → add it to `design.md` + tokens first.

## 5. Commit + PR

Conventional commits. Push the branch, open a PR against `main`.

## 6. Review gate

Call only the reviewers the change needs — see the **review routing table** in CLAUDE.md.
Code changes need `@code-reviewer` + **founder sign-off**; `@cto` only for schema / model /
infra. One round; re-review only after a BLOCK. Docs-only changes go direct to main.

## 7. Merge ≠ deploy

Merge only through your gated merge script (e.g. `scripts/safe_merge.sh <pr>`, which refuses unless CI passed). No `--auto`, no `--admin`.

Merging ships **nothing**. Production deploys **only on a `vX.Y.Z` tag** (record this as an ADR).

```bash
git fetch origin --tags -q && git tag -l 'v*' | sort -V | tail -1   # find current
git tag vX.Y.Z <merge-commit> -m "<what>" && git push origin vX.Y.Z
```

## 8. Deploy

From a clean checkout of the merged `main` (the worktree works if its tree matches):
```bash
<your-deploy-command> --image-label vX.Y.Z
```
Check your deploy tool's login first. If the login has expired, the human re-authenticates;
never paste credentials into a chat or a file in the repo.

## 9. Smoke-test live — before saying "done"

Never report success from a green deploy log. Hit the real host:
```bash
curl -s -o /dev/null -w "%{http_code} %{content_type}\n" https://<your-app-host>/<asset>
curl -s https://<your-app-host>/<page> | grep -o "<expected marker>"
```
Confirm the **new** thing is present and the **old** thing is gone.

If you have more than one surface (static site vs API), check the right host for the
change: a deploy of one does not touch the other.

## 10. Clean up

```bash
git worktree remove <path> --force; git worktree prune; git branch -d feat/<name>
```
Then report: what shipped, the tag, and the smoke-test evidence.
