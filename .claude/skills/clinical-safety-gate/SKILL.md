---
name: "clinical-safety-gate"
description: "Patient-safety gate for <your-app>. Use BEFORE shipping any user-facing feature, label, or health content (explainers, social posts, doctor summary, relation/side-effect displays). Holds the \"filing clerk, not a doctor\" line, the US/EU medical-device boundary for a PATIENT-facing app, and the record-integrity rules. Triggers on: new vault feature, any copy a patient reads, medical explainer, relation/side-effect display, doctor summary, alerts, warnings, sharing."
---

# Clinical safety gate

<your-app>'s positioning rule: **a filing clerk, not a doctor.** We organise a person's
records. We do not diagnose, recommend treatment, or tell someone what to do.

Run this on anything a patient or caregiver will read.

## 1. The advice line

**BLOCK** — never ship:
- Tells the user they **have** a condition, or implies it ("your numbers suggest...").
- Recommends starting, stopping, or changing a medicine or dose.
- Says a drug **will** treat/fix something, or ranks options.
- **Reassures** — "normal", "nothing to worry about", "this is fine".
- **Absence claims** — "no interactions", "no known allergies", "no side effects".
  We can only say what *is recorded*, never that nothing exists.
- **Attributes a symptom to a drug.** Causal language is the sharpest edge (see §3).
- **Urgency or triage language** — "seek care immediately", "this is serious".

**ESCALATE, do not auto-block** — unsolicited clinical alerts (duplicate-ingredient,
interaction, out-of-range). A cited, opt-in "the same ingredient appears twice — ask
your pharmacist" may be *safer* for patients and defensible. Force a deliberate review
with @clinical-validator; never build one by reflex.

**ALLOW**:
- Restating what is in the user's own record, with its source and last-confirmed date.
- **Reproducing a flag already printed on the source document.** Computing a *new*
  out-of-range flag is a different thing and is not allowed.
- Generic "ask your doctor or pharmacist." Never name a drug, class, or test that we
  derived from their data — "ask your doctor about a statin" is a recommendation.
- Neutral reference information clearly labelled as general and not personalised.

Note: tier-1 "in your record" surfaces **are** personalised. They are permitted only
under the §2 lane, and only with neutral, non-causal wording.

## 2. The device line — for a PATIENT-facing app

Do **not** reach for the statutory Clinical Decision Support exclusion
(FD&C §520(o)(1)(E)). It requires the software support a **healthcare professional's**
recommendation. Patient- and caregiver-facing software cannot use that exemption at
all. Citing it for <your-app> is a category error.

**Our actual lane:**
- §520(o)(1)(D) — transfer, storage, display, communication of health/lab data
  **without interpretation**.
- §520(o)(1)(A) — administrative support.
- FDA **Mobile Medical Apps** enforcement discretion for patient tools that: organise
  and track health information; give patients access to information about their
  conditions and treatments; help patients communicate with clinicians.

**The device question is about intended use** — is it for diagnosis, cure, mitigation,
treatment, or prevention of disease? That turns on our **claims and labelling**, what
we say it is for, not on the cleverness of the code.

**EU is stricter.** MDCG 2019-11 + MDR Rule 11: software applying patient-specific
data against a drug database, for information used in therapeutic decisions, can
qualify as medical device software. Assume EU exposure if any EU user is expected.

**Our design heuristic** (ours, *not* FDA's test — never present it as FDA's):

    restating a record -> general reference -> patient-specific interpretation -> recommendation

Stay left of "patient-specific interpretation." **Failing any one** of these loses the
lane; they are not a combined score:
1. Is the output **patient-specific** (uses their data to reach a conclusion)?
2. Would a reasonable user **rely on it** instead of asking a clinician?
3. Can the user **see the basis** and check it independently?
4. Is it framed as **time-critical**?

A "yes" to 1 with a "no" to 3 -> STOP, escalate to @clinical-validator + @legal.

FDA revised its CDS and General Wellness guidances in early 2026; re-verify before
relying on specifics. General Wellness almost certainly does not cover meds and
conditions — do not lean on it.

## 3. Known exposure: relations shown from the user's own list

Keep your own list of open findings here. Example pattern to look for: the app shows
drug->condition ("may treat") or drug->side-effect relations keyed off the user's own
medication list, tiered "In your record" / "Reference only". Review such features
against the device framing in section 2, not only the copy. Typical findings:

- **SAFETY: linking the user's own symptom to their own drug** ("possible side effect,
  you've also logged X"). No temporality check, and a real anchoring risk: it can steer
  someone away from getting a serious cause evaluated. A footer disclaimer does not cure
  a design that invites the inference. Remove it, or reword neutrally: "Some people
  taking this medicine report X. If you are having this, tell your clinician."
- **Accuracy: "used for X, also in your record"** reads as *why you take it*. Many drugs
  have many indications; the prescriber may have had a different reason. Say so.
- **Accuracy: "commonly used for"** when the data is label-level, not frequency data.
  Use "labelled uses include".
- **Accuracy: stale reference data.** Show a data-as-of date, cap the list, prioritise
  serious/common, say "listed on the label".
- **Nocebo / non-adherence.** A long, scary, unranked list can drive someone to stop an
  anticoagulant or anticonvulsant on their own.
- **Disclosure gap.** State explicitly that the app does not check for drug interactions.
- **Stale status.** A stopped med mis-extracted as active still shows side effects.
  Show last-confirmed dates.

## 4. Explainer and marketing copy only

These apply to **health explainers, posts, and marketing** — *not* to the vault
displaying the user's own record, which legitimately shows their own doses and brand
names because they are theirs.

- No dosing. No drug brand/product names.
- No self-diagnosis framing.
- Every number or threshold traced to a named guideline, **re-checked** at review time.
- Always a "see your doctor / clinician" line.
- Always labelled *"This is general info, not medical advice."*
- Plain language, short sentences, no assumed medical training.

## 5. Record integrity (whole app)

- **Extraction errors propagate.** A wrong RxCUI or a mg/mcg OCR slip drives the
  relations *and* the doctor summary. Anything extracted-and-unconfirmed must be
  labelled "extracted — please verify", and confirm-status must travel with anything shared.
- **Empty is not none.** "No allergies recorded" is not "No known allergies". Applies
  app-wide; allergies, anticoagulants, insulin and opioids get extra care.
- **Currency.** Show last-confirmed dates on meds and on reference data.
- **Doctor summary** is clinician-facing, so an error there causes harm. It must carry
  source links, patient-reported vs clinician-documented, confirm status, and date.
- **Emergency / caregiver use.** It may be read in an ER. Disclose incompleteness.
- **Sensitive categories** (HIV, mental health, reproductive, substance use) need
  deliberate handling in sharing and proxy consent.
- **Crisis content.** Decide what happens if free text signals self-harm.

## 6. Who runs this gate

A non-clinician cannot clear clinical accuracy alone. **Any PASS on patient-specific
output requires @clinical-validator.** @clinical-validator and @security-reviewer hold
a veto per CLAUDE.md. Device-boundary questions go to @legal as well.

## 7. Output

Report **PASS**, **PASS WITH CHANGES** (list them), **ESCALATE**, or **BLOCK** (name the
rule). On BLOCK, never ship — escalate to the founder.
