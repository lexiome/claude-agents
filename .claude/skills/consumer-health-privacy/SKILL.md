---
name: "consumer-health-privacy"
description: "Privacy/compliance frame for <your-app>. Use when touching data flow, storage, logging, sharing, consent, deletion, export, third-party processors, model vendors, or writing a privacy policy. Establishes that the regime is decided PER FLOW — the B2C vault is FTC Health Breach Notification Rule + state consumer-health law + GDPR Art. 9, while the B2B extraction API is a HIPAA Business Associate flow. Triggers on: privacy, consent, deletion, export, PHI, data sharing, breach, BAA, provider, clinic, policy, retention, processor."
---

# Consumer health privacy

Working notes from one product, written by a non-lawyer. Not legal advice, may be out of date or wrong, and not a compliance checklist. Check current law with a qualified lawyer in your jurisdiction. No warranty (see LICENSE).

## 1. Decide the regime PER FLOW, not per company

<your-app> has **two products** with different answers. Never apply one answer to both.

| Flow | Who we are | What applies |
|---|---|---|
| **B2C vault** — the person stores their own and their family's records | Neither Covered Entity nor Business Associate | **FTC HBNR**, **state consumer-health law** (§3), **GDPR Art. 9** for EU users |
| **B2B extraction API** — a customer sends us clinical notes | **Business Associate** if that customer is a Covered Entity | **HIPAA** + a signed **BAA**, before any real note |

The API is *already* a provider-facing product; it simply has no real-PHI customers
yet. **Real clinical notes must not go through the API until a BAA exists.**
@legal holds the veto on any real-PHI flow through it.

Do not claim "HIPAA compliant" for the vault. For a non-covered entity that is
misleading, and misleading privacy claims are themselves an FTC risk.

## 2. Tripwire — when HIPAA *does* reach the vault

**Stop and ask @legal** for any feature where a provider has an account, or where we
receive data from a provider rather than from the patient. That includes:
- A clinician/clinic creates an account, pays us, or directs patients to us as part of care.
- White-label, employer, payer, or clinic-sponsored rollouts.
- We ingest from a provider on the provider's behalf, or under a provider contract.

**Not** a BA relationship:
- A patient sends a share link or export to their own doctor. The doctor's receipt is
  the doctor's HIPAA problem, not ours.
- The patient pulls their own records via patient-directed access (Cures Act / SMART
  on FHIR) — but check the aggregator's own terms.

Rule of thumb: **whose behalf, and who pays or controls.** If the provider is our
customer or directs the flow, treat it as HIPAA.

## 3. State consumer-health law — a significant practical gap

This may be the first rule to apply once US users sign up.

- **Washington My Health My Data Act** — has a **private right of action** (individuals
  can sue). Applies to WA consumers wherever we are. Requires *separate* consent to
  collect and *separate* consent to share, a delete right reaching backups, and a
  dedicated consumer health data privacy policy.
- **Nevada** and **Connecticut** — similar consumer-health-data rules.
- **California** — CMIA can treat apps storing medical information as providers of
  health care; CCPA/CPRA treats health data as sensitive personal information.

## 4. FTC Health Breach Notification Rule

Applies to vendors of **personal health records** not covered by HIPAA. A record drawing
from multiple sources (manual entry, photo, imports) very likely qualifies.

- "Breach of security" includes **unauthorised disclosure**, not only intrusion. The
  2024 amendments confirm this. See the FTC's GoodRx, BetterHelp and Premom actions —
  all pixel/SDK leaks.
- A disclosure to a **processor is not automatically a breach** — it must be authorised:
  disclosed in the policy, consented to, and covered by contract.
- Our third-party processors owe **us** breach notice. Put that in their contracts.

**Deadlines — know them in advance.** Verify them at ftc.gov and with counsel; these change.
- **GDPR Art. 33: 72 hours** to the regulator — the shortest and most stable number.

Practical consequence: **no third-party analytics or ad SDK ever receives health data.**
Self-host analytics. Then verify the usual leak paths carry no health content: URLs,
page titles, referrers, and **error reports (Sentry)**.

## 5. GDPR (if any EU user)

- Health data is **special category** (Art. 9) — needs an Art. 9 condition, normally
  **explicit consent**, on top of an Art. 6 basis.
- **DPIA (Art. 35)** is triggered: large-scale special-category processing with new
  tech (LLM + photo). Write it before launch; a short template suffices.
- Also: an **Art. 28 DPA** with every processor, **SCCs / Data Privacy Framework** for
  non-EU transfers, a **RoPA** entry, and an **Art. 27 EU representative** if targeting EU.
- **Cheapest path at launch: US-only, EU users geo-blocked or waitlisted.** That defers
  the DPIA, the DPA work and Art. 27 entirely.

## 6. The photo-ingest gate: keep it, reword it

*(One product's reasoning; confirm with counsel.)*

A BAA is probably the wrong *primary* instrument for the vault: we would not be a CE
or BA, so a model vendor has nothing to sign as our HIPAA subcontractor. The real risks
are model-vendor **training/retention**, **prompt caching and abuse-monitoring logs**,
unauthorised-disclosure risk under HBNR, and GDPR Art. 28 / transfer terms.

**Do not drop the gate without @legal sign-off.** It should require:
1. **Vertex AI only** for real user data — never the AI Studio / Gemini API-key path;
   consumer and free-tier terms can permit training on inputs.
2. Written confirmation of settings: **no training on our data**, abuse-logging
   exemption or **zero data retention**, prompt/context caching **off**, region pinned.
3. Google Cloud data processing terms (CDPA) accepted and filed.
4. Privacy policy discloses the processor and the photo processing, with **in-flow
   consent at upload**.
5. A defined retention for the uploaded image — preferably never persisted, or deleted
   after OCR (OCR is local Tesseract, which helps).
6. @security-reviewer sign-off, and the live e2e runs **synthetic data only**.
7. A BAA is **optional** belt-and-braces. Never market it, and never let it justify a
   "HIPAA compliant" claim.

The work does not shrink; only the label changes.

## 7. Non-negotiables in this product

- **Consent is per-person.** Managing another adult requires their recorded consent;
  they must see who has access (view log) and be able to revoke it.
- **Consent paths that differ** and must not be conflated: minors (parent/guardian),
  adults lacking capacity (power of attorney), and deceased family members. The GDPR
  household exemption does not cover us — we are the controller.
- **Delete means delete** — including backups, caches, derived data and any extraction
  cache keyed to their content. **Publicly, state a purge window** ("backups purge
  within X days") and make the code match it. Do not publish "we delete everything"
  unless your storage snapshots and processor logs actually comply.
- **Export** must be real and machine-readable, not a screenshot.
- **Never log raw clinical text.** No real patient data in the repo, ever (CLAUDE.md).
- **Cache keys must not embed PHI** (enforce this with a test or assertion).
- **Audit public claims** before launch: grep the marketing site, hero copy and pitch
  for "HIPAA", "secure", "private", "never shared", "bank-grade". Say what we actually
  do. Do not casually say "not subject to HIPAA" either.

## 8. Before real users

- **Breach runbook** written in advance — HBNR timelines run from *discovery*.
- Privacy policy matching actual flows, **plus** a separate consumer-health-data policy
  for WA/NV. ToS covering: not for diagnosis or medical advice, user responsibility for
  accuracy, liability cap.
- App-store health declarations and privacy labels — only if we go native.
- **Your own jurisdiction:** check whether the country you operate from, and each country you launch in, has a data protection law, and launch only where you have checked. Where the founder lives also affects incorporation and GDPR "establishment": raise with counsel.

**Outside counsel is genuinely required** for: one session with a health-tech privacy
lawyer before real users (bring this analysis, the policy drafts, the consent flows and
the processor list); entity formation and the first signed customer/provider contract;
any breach; and anything in §2 before it ships. Do not pay a lawyer to redo this
analysis — pay them to check the state-law and FTC points and review the policy.

## 9. Output

State the flow and the regime you are applying, and why. Then findings, most-severe
first. Flag anything that would make a public privacy claim untrue.
