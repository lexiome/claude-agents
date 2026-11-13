---
name: one-glance
description: Check that an app screen sends the eye to its one job at a glance, through font size, weight and colour. Use when designing or reviewing any screen or type change, before showing screens to the founder, or when someone says the UI has no focus, looks flat, or everything competes. A passing design audit does not prove this.
---

# One glance

A screen is right when someone sees its one job first, without reading. Consistency isn't enough: a design audit script measures sameness, not attention.

This file and your `docs/planning/product/design.md` are the source of truth.

Check the screen with two founder tests too (`CLAUDE.md`): could a tired, distracted person do its one job without thinking (the drunk test), and would a grandma understand every word (the grandma test)?

## 1. Name the job

| Screen | One job | Second |
|---|---|---|
| Home | Answer the waiting medicine | See what was added |
| Reminders | Answer the waiting medicine | See what's coming |
| Medicines | Find a medicine | Its dose and times |
| Medicine details | Which medicine and dose this is | Its reminder |
| Add a medicine | Type the name and save | Optional fields |
| Records | Pick a record type | Counts |
| Sign in, Sign up | Get in | Other ways in |

A new screen needs a row here first. If it has two jobs, say so instead of making both loud.

## 2. Spend emphasis once

- One thing per screen uses size, weight and dark colour together: the job. Everything else gets one lever at most.
- A page title that isn't the job, such as a greeting, doesn't outrank it.
- Three text levels on a screen is the target. If a fourth appears, justify it.
- Labels support values. Lighter or smaller is fine, but see the limits below.

## 3. Test it

Run on real renders at 390x844, every language you ship, and look at the output:

1. **Blur:** Gaussian blur about 8px at 2x. What stands out must be the job.
2. **Grayscale:** the order must still read without colour.
3. **First-read list:** write the first three things seen. The first must be the job.

Show the founder before, after, and the blur of each. Say what you actually looked at.

## 4. Limits: never trade these for emphasis

- Older users: text floor 13px, no pale grey labels, contrast at least 4.5:1 for text, and 3:1 for control outlines.
- Some scripts (e.g. Urdu Nastaliq) have one weight, so hierarchy comes from size and colour there. Test each script separately.
- Large and Extra large must still read and not clip.
- Doses, allergies and times stay fully readable, never de-emphasised to the point of being missed.
- Filing clerk rule (`clinical-safety-gate`): emphasis follows the task, never implies urgency or risk about a medicine.
- All changes go through tokens, then the standard and audit are updated.
