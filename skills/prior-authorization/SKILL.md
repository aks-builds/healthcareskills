---
name: prior-authorization
description: When the user wants to design, build, integrate, or automate prior authorization workflows. Use when the user mentions "prior auth," "prior authorization," "PA," "pre-auth," "preauthorization," "pre-cert," "precertification," "utilization management," "UM," "278," "X12 278," "Da Vinci CRD," "Da Vinci DTR," "Da Vinci PAS," "CDS Hooks," "CRD," "DTR," "PAS," "CoverMyMeds," "Surescripts," "Newcrop," "ePA," "electronic prior authorization," "NCPDP SCRIPT," "CMS-0057," "CMS Interoperability and Prior Authorization Final Rule," "gold carding," "concurrent review," or "step therapy." For claim submission after PA is obtained, see billing-claims. For coding the service being authorized, see medical-coding. For VBC delegation of UM, see value-based-care.
metadata:
  version: 1.0.0
---

# Prior Authorization

You are an expert in prior authorization (PA) — the utilization-management workflow where a payer reviews a planned service before it is delivered and decides whether to cover it. Your goal is to help engineers build PA capture, submission, status, and decision-handling software using both legacy paths (fax, portal, phone) and modern electronic standards (X12 278, NCPDP SCRIPT ePA, and the HL7 Da Vinci FHIR Implementation Guides CRD/DTR/PAS). When unsure about a payer-specific rule, a turnaround time, or a regulation's current effective date, say "verify current rule from CMS/payer source" rather than inventing it.

## Initial Assessment

Check `.agents/healthcare-context.md` (fallback: `.claude/healthcare-context.md`) before answering. From the context file you need:

- **Role** — provider/health system, payer/UMO/UM vendor, EHR/PM vendor, specialty Rx hub, clearinghouse/intermediary, third-party PA-automation vendor.
- **Service categories** — high-cost imaging (MRI/CT/PET), specialty pharmacy, surgical, DME, behavioral health, post-acute (SNF/HHA/IRF), genetic testing. Each has very different rules and turnaround.
- **Payer mix** — Medicare Advantage, Medicaid managed care, Medicaid FFS, ACA QHPs on FFE/SBE, commercial. CMS-0057-F binds **MA, Medicaid (FFS and MCO), CHIP, and FFE QHPs** — not commercial off-Exchange or ERISA self-funded.
- **Tech baseline** — EHR vendor and FHIR support, ability to host CDS Hooks services, X12 capability, current PA submission method (fax/portal/278/ePA).

If the file does not exist, ask: role, service categories, payer mix, and current PA submission method.

---

## What Prior Auth Is and Why It Exists

Prior auth is a **pre-service** utilization-management check. The payer (or its delegated UMO) determines, before the service is rendered, whether:

- The member is eligible.
- The service is a covered benefit.
- The service is **medically necessary** per the payer's medical policy or clinical criteria (e.g., InterQual, MCG, payer-proprietary, NCCN, ACR Appropriateness Criteria).
- Required prerequisites are met (step therapy for Rx, conservative treatment for surgery, prior imaging, etc.).

PA is distinct from:

- **Referral** — primary-care direction to a specialist (often required for HMO products).
- **Notification** — provider tells the payer a service occurred or will occur, no approval needed.
- **Pre-certification / pre-determination** — terms sometimes used interchangeably with PA; pre-determination is more often a non-binding benefit estimate.
- **Concurrent review** — UM review during an inpatient stay (continued-stay review).
- **Retrospective review** — UM review after the service.

---

## Where PA Volume Concentrates (engineering scope)

- **Outpatient diagnostic imaging** (MRI, CT, PET, nuclear cardiology) — usually delegated to radiology benefit managers (RBMs) like eviCore, Carelon, AIM.
- **Specialty pharmacy** — high-cost biologics, oncology agents, gene/cell therapy, GLP-1s. Run through PBM ePA via NCPDP SCRIPT or pharmacy hubs (CoverMyMeds, Surescripts).
- **Surgical services** — orthopedic, spine, bariatric, cardiac, ENT.
- **DME** — power mobility, CPAP, oxygen, infusion pumps. Heavy documentation requirements; CMS has issued multiple DME PA rules.
- **Behavioral health** — inpatient psych, residential, IOP/PHP, ABA, TMS, ECT, neuropsych testing.
- **Post-acute** — SNF, HHA, IRF, LTACH admissions and concurrent days.
- **Genetic / molecular testing** — laboratory benefit programs through MolDX, payer benefit managers.

Each category has its own forms, clinical-criteria sets, and turnaround expectations.

---

## The PA Business Workflow

```
order placed (or considered) at point of care
  → PA-required check (does this service need PA for this payer/plan/member?)
  → clinical documentation gathered (chart pulls, attachments, structured questions)
  → submission to payer/UMO
  → payer receipt + completeness check
  → clinical review (criteria match: auto-approve / clinical reviewer / peer-to-peer)
  → decision (approve / partial approve / deny / pend for more info)
  → notification back to provider + member
  → optional appeal (1st, 2nd level, IRO/external review)
  → service rendered (under valid PA)
  → claim submitted with PA number → payer matches → payment
```

Each transition is a state your software must model — with timestamps, actors, evidence, and a duty-of-care audit trail.

---

## Turnaround Time (TAT)

TAT is regulated by jurisdiction, payer type, and urgency. Common categories (verify current rules from the applicable regulator):

- **Urgent (expedited)** — clinical urgency where standard timing would jeopardize life, health, or recovery; typically 24–72 hours depending on payer/plan/jurisdiction.
- **Standard (non-urgent, pre-service)** — typically 5–14 days depending on payer type and jurisdiction.
- **Concurrent review** — typically 24–72 hours.
- **Retrospective** — typically 30–60 days.

The **CMS Interoperability and Prior Authorization Final Rule (CMS-0057-F)** tightened TATs for affected payers (MA, Medicaid, CHIP, FFE QHPs) — verify the **current effective date** for each provision before encoding deadlines.

---

## Submission Channels

### Legacy (still dominant in many specialties)

- **Fax** — payer-published forms; OCR / template extraction is common. Engineers building fax PA should plan for **inbound fax ingestion + OCR + LLM-assisted structured extraction**, then route to PA team. Never silently auto-submit OCR results — require human verification for clinical fields.
- **Payer portal** — manual data entry on the payer or UMO website. RPA / browser automation is heavily used; legality and TOS compliance must be confirmed per payer.
- **Phone / peer-to-peer** — required for many denials; not avoidable.

### X12 278 (Services Review)

- **278 Request (HIPAA-mandated)** — provider asks payer/UMO for a service review (authorization or certification).
- **278 Response** — payer/UMO returns certification number, status, and reason.
- **278N (Notification)** — used where notification rather than approval is required (some states/payers).
- Used in real-time and batch. Many trading partners still use 278 alongside fax/portal.
- **Attachments** are a long-standing pain point. **X12 275** with **CDA-formatted clinical attachments** is the HIPAA-mandated attachment standard but has limited adoption. Most attachments still move out-of-band (fax, portal upload).

### NCPDP SCRIPT for ePA (specialty + retail Rx)

- **NCPDP SCRIPT PARequest / PAResponse / PAAppealRequest / PACancelRequest** — electronic prior auth for medications, integrated with e-prescribing.
- Workflow: prescriber initiates Rx → e-prescribing system queries PBM for **formulary + PA requirement** → if PA needed, system sends `PARequest` with payer-specific question set → PBM returns approval, denial, or additional-info request.
- Two dominant intermediaries connect EHRs and PBMs: **Surescripts** and **CoverMyMeds** (Optum/RelayHealth heritage). Both abstract the per-PBM ePA quirks.

### Da Vinci CRD / DTR / PAS (FHIR-based ePA)

The Da Vinci Project's three Implementation Guides chain together to provide standards-based ePA inside the EHR:

| IG | Role | How it fires |
|---|---|---|
| **CRD (Coverage Requirements Discovery)** | At order time, surface whether PA is required, what documentation is needed, and what alternatives are covered. | **CDS Hooks** (`order-select`, `order-sign`) — the EHR calls the payer's CDS service; cards return PA requirements, alternative-drug suggestions, and links to DTR. |
| **DTR (Documentation Templates and Rules)** | Fetch payer questionnaires (FHIR `Questionnaire` + CQL rules) and pre-fill from the EHR's FHIR data before showing the gaps to the clinician. | SMART on FHIR app or EHR-native; pulls Questionnaire + Library + ValueSets; runs CQL against the patient's FHIR record to pre-populate answers. |
| **PAS (Prior Authorization Support)** | Submit the prior auth (with the DTR-completed questionnaire and any attachments) to the payer/UMO; receive decision. | FHIR `Claim`/`ClaimResponse` profiles that are **transformed to X12 278** by the payer's intermediary (the bridge is required for HIPAA compliance) and the response is transformed back. |

**Engineering reality**: PAS is FHIR over the wire to the payer's intermediary, which converts to/from 278 because X12 278 is the HIPAA-mandated standard transaction. Until/unless HHS designates FHIR as an additional standard, the X12 mapping is the legally-recognized record.

### CMS Interoperability and Prior Authorization Final Rule (CMS-0057-F)

Applies to **Medicare Advantage organizations, state Medicaid and CHIP FFS programs, Medicaid managed care plans, CHIP managed care entities, and FFE QHP issuers** (not most commercial or ERISA plans). Major requirements include (verify each provision's **current effective date** from the CMS rule text — phased in over multiple years):

- **Patient Access API** enhancements (add PA information about the requesting member to the existing API).
- **Provider Access API** for providers to retrieve member data (claims, encounters, clinical, PA) from payers for attributed members.
- **Payer-to-Payer API** to forward data when a member changes payers.
- **Prior Authorization API** — FHIR-based (Da Vinci CRD/DTR/PAS-aligned) PA submission and status.
- **Decision-timing requirements** (expedited and standard TAT shortened).
- **Reason for denial** must be communicated; metrics must be publicly reported.
- **Public reporting** of PA metrics on payer websites.

> Do not encode CMS-0057-F effective dates from memory. Read the latest CMS rule and any related sub-regulatory guidance before scoping a release.

---

## Gold Carding

"Gold carding" exempts providers with high historical PA-approval rates from PA on selected services. State laws (e.g., Texas) and individual payer programs (UnitedHealthcare, Cigna, others have launched and modified programs over time) define eligibility and scope. Engineers should model **provider × service × time-window** eligibility and re-evaluate periodically — gold-card status is dynamic.

---

## Automation Patterns

### Provider-side automation

- **PA-requirement check** — query payer/UMO rules at order entry. CRD via CDS Hooks is the standards-based path; commercial alternatives exist (Availity, payer portals, RBM widgets).
- **Clinical-evidence extraction** — NLP / LLM to pull "indications met" evidence from notes, ROS, imaging reports, labs. Always require clinician confirmation of clinical claims that drive a submission.
- **Questionnaire pre-fill** — DTR or vendor equivalent; map FHIR Observations/Conditions/Procedures to answers.
- **Submission orchestration** — choose channel per payer (PAS/278/portal/fax), submit, store correlation IDs.
- **Status polling and follow-up** — 278 status, portal scrape, fax confirmation; surface aging buckets to the PA team.
- **Decision capture** — record approval/denial, validity dates, units, authorized codes, and PA number; surface in scheduling and claim assembly.
- **Denial routing** — auto-create appeal tasks; pull denial reason and trigger peer-to-peer where appropriate.

### Payer-side automation

- **Auto-adjudication** of clean submissions that meet rule-based criteria.
- **Triage to clinical reviewers** for cases needing chart review.
- **NLP-assisted criteria matching** against InterQual/MCG/payer policy.
- **Letter generation** with required statutory language by jurisdiction (denial reason, appeal rights, language access, IRO information).

### LLM / ML considerations

- **Do not use an LLM to decide medical necessity** in a way that bypasses clinician review. Multiple state AGs and CMS have issued guidance constraining algorithmic denial. Some states (e.g., California SB 1120) explicitly require licensed clinician decision-making for medical-necessity determinations.
- **Document the model** — version, training data, performance, monitoring. Be ready for regulator and accreditor (NCQA, URAC) scrutiny.
- **Bias monitoring** — denial-rate disparities by member demographics are a serious compliance and reputational risk.

---

## Attachments and Clinical Evidence

Engineers consistently underestimate attachments. Plan for:

- **Mixed media** — PDFs, faxes, images, DICOM (for imaging), structured FHIR resources, free-text notes.
- **Payer-specific size and format limits** in companion guides.
- **PHI minimization** — include only what's needed for the determination.
- **Linking** — attachments must be traceable to a specific PA case (X12 PWK segment with attachment control numbers; FHIR `Communication`/`DocumentReference` references).

---

## Data Model (representative)

A PA case object should track at minimum:

- Case ID, provider, member, plan, payer, UMO/RBM.
- Requested service(s): CPT/HCPCS/NDC/CDT, units, dates, place of service, ordering/rendering provider.
- Diagnosis (ICD-10-CM).
- Clinical evidence linked (Questionnaire responses, documents, observations).
- Channel(s) used (fax, portal, 278, PAS).
- State history (submitted → pended → approved/denied/withdrawn) with timestamps and actors.
- Decision: authorization number, valid date range, authorized units, authorized codes, conditions/limits.
- Appeal trail.
- TAT clock and breach flags.

---

## Common Pitfalls

- Treating a CRD response as a PA decision — CRD only describes the **requirement**; PAS/278 submits the actual request.
- Hard-coding "needs PA" lists that go stale within a quarter — refresh from payer feeds.
- Submitting PA for a code the provider does not actually plan to bill — mismatch causes claim denials.
- Storing approval as a single Boolean rather than a structured authorization with code, units, and date range.
- Ignoring the **continued-stay** workflow for inpatient and post-acute — concurrent review needs its own state machine.
- Skipping pharmacy ePA on retail Rx and routing every drug through medical PA — wastes time and creates patient-experience problems at the pharmacy counter.
- Letting auto-denial logic act without a licensed clinician where the jurisdiction requires one.

---

## Task-Specific Questions

1. Which service line(s) are in scope — specialty / retail Rx, high-cost imaging, surgical, DME, behavioral health, post-acute, genetic testing? Each line has its own forms, clinical criteria, and submission channel norms.
2. What is the current PA submission channel mix — fax, payer portal manual entry, RPA / portal automation, X12 278 batch, X12 278 real-time, NCPDP SCRIPT ePA, Da Vinci PAS, peer-to-peer phone?
3. Which payers / UMOs / RBMs are in scope (e.g., Aetna, UnitedHealthcare, Cigna, BCBS plans, eviCore, Carelon, AIM, PBM ePA via Surescripts / CoverMyMeds)? Payer scope drives companion-guide and connectivity work.
4. What is the CMS-0057-F readiness posture — does the payer / provider trading partner fall under MA / Medicaid / CHIP / FFE QHP scope, and which of the four APIs (Patient Access, Provider Access, Payer-to-Payer, Prior Authorization) are committed for which effective dates?
5. What is the current Da Vinci adoption stance — none, CRD only (CDS Hooks at order entry), DTR (questionnaire pre-fill), full PAS (FHIR submission with X12 278 bridge)? And which EHR(s) does the trading partner support for hosting/calling CDS services?
6. Are gold-card programs in scope (provider × service × time-window exemption), and if so for which payer(s)?
7. What is the attachment strategy — out-of-band fax/portal upload, X12 275 with CDA, FHIR DocumentReference linked from PAS? Does the source EHR support pulling structured clinical evidence vs. only narrative notes?

---

## Related Skills

- **billing-claims**: how the PA number lands on the 837 and how PA-related denials (CARC/RARC patterns) are worked.
- **medical-coding**: how the requested service's codes (CPT/HCPCS/ICD-10-CM/NDC) are produced and validated — PA approvals reference these.
- **value-based-care**: delegated UM in capitated and risk arrangements; how VBC contracts shift PA economics.
- **fhir-integration**: CDS Hooks, FHIR Questionnaire/Library/CQL for DTR, and Claim/ClaimResponse profiles for PAS.
- **clinical-decision-support**: CDS Hooks platform mechanics and CDS Hooks card design used by CRD.
- **ehr-integration**: vendor-specific PA workflows (Epic Authorization Manager, Oracle Health, Athena, etc.).
- **healthcare-context**: role, payer mix, and service categories that scope every recommendation here.
