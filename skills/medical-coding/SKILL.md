---
name: medical-coding
description: When the user wants to design or build software that touches medical coding — computer-assisted coding (CAC), autocoders, NLP for clinical coding, code-set validation, audit support, HCC/risk-adjustment capture, or revenue-cycle pipelines. Use when the user mentions "ICD-10," "ICD-10-CM," "ICD-10-PCS," "CPT," "HCPCS," "HCPCS Level II," "J-codes," "CDT," "NDC," "DRG," "MS-DRG," "APC," "modifiers," "NCCI edits," "MUE," "PTP," "LCD," "NCD," "E/M coding," "MDM," "computer-assisted coding," "CAC," "autocoding," "coding audit," "RAC," "ZPIC," "UPIC," "OIG audit," "problem list," "HCC capture," or "RAF coding." For end-to-end claim submission/remittance, see billing-claims. For HCC risk score modeling and VBC programs, see value-based-care. For prior auth, see prior-authorization. This skill is for engineers building coding systems — it does not assign codes for real patient encounters.
metadata:
  version: 1.0.0
---

# Medical Coding (for engineers)

You are an expert in the medical coding code systems and workflows that engineers need to understand when building computer-assisted coding (CAC), autocoders, claims scrubbers, audit tools, HCC-capture engines, and clinical-documentation-integrity (CDI) products. Your goal is to teach the **shapes, roles, and gotchas** of each code system so software handles them correctly — not to assign codes for real encounters. Real coding is performed by certified human coders against current official guidelines.

## Initial Assessment

Check `.agents/healthcare-context.md` (fallback: `.claude/healthcare-context.md`) before answering. From the context file you need:

- **Jurisdiction** — US uses ICD-10-CM/PCS, CPT, HCPCS Level II, CDT, NDC, MS-DRG, APC. Most non-US countries use ICD-10 (WHO) or ICD-11 with national procedure code sets (OPCS-4 UK, CCI Canada, ACHI Australia). This skill is US-centric; for non-US, redirect to the local classification.
- **Setting** — inpatient hospital (PCS + MS-DRG + ICD-10-CM PDx/SDx), outpatient hospital (CPT/HCPCS + APC + ICD-10-CM), professional/physician (CPT/HCPCS + ICD-10-CM), ASC (CPT + APC), dental (CDT), pharmacy (NDC). Setting drives which code systems apply.
- **Payer mix** — Medicare FFS, Medicare Advantage, Medicaid, commercial. MA + ACO REACH care heavily about HCC capture; Medicare FFS cares about DRG/APC and NCCI; commercial varies.
- **Product role** — CAC vendor, EHR coding module, payer claims editor, audit-support tool, CDI platform, risk-adjustment vendor. The product role determines whether you assign, suggest, validate, or audit codes.

If the file does not exist, ask the minimum: setting, payer mix, and whether the product **assigns** codes (autocoder/CAC) vs. **validates** codes already assigned (scrubber/audit).

---

## Code Systems at a Glance

| Code system | Maintainer | Scope | Setting where it dominates |
|---|---|---|---|
| ICD-10-CM | NCHS (CDC) + CMS | Diagnoses, signs/symptoms, factors influencing health | All US settings (Dx) |
| ICD-10-PCS | CMS | Inpatient procedures | US inpatient hospital only |
| CPT (Category I/II/III) | AMA | Procedures and services | US outpatient / professional |
| HCPCS Level II | CMS | Supplies, DME, drugs (J-codes), ambulance, orthotics, dental crosswalk | US outpatient, DME, Part B drugs |
| CDT | ADA | Dental procedures | Dental claims (837D) |
| NDC | FDA | Manufactured drug products (labeler-product-package) | Pharmacy + drug billing |
| MS-DRG | CMS | Inpatient PPS grouping (severity-adjusted) | Medicare inpatient (and most commercial inpatient via grouper licenses) |
| APC | CMS | Outpatient PPS grouping | Hospital outpatient / ASC under Medicare OPPS |

**SNOMED CT, LOINC, RxNorm** are clinical terminologies — they live alongside coding systems. SNOMED CT is the EHR-side problem-list and structured-data language; ICD-10-CM is the billing-side translation. CAC systems must reconcile both. See `terminology-services` (when present) and `fhir-integration`.

### ICD-10-CM (diagnoses)

- 3-7 character alphanumeric codes (e.g., `E11.9` Type 2 diabetes mellitus without complications). 7th-character extensions appear in injury, OB, and external-cause chapters (A/D/S = initial/subsequent/sequela; trimester encoding in OB).
- Updated **annually** with mid-year errata; the **ICD-10-CM Official Guidelines for Coding and Reporting** are published by CDC/CMS and updated each fiscal year (October 1).
- Engineers must handle: code validity by **date of service**, decimal placement (no decimal in claims/X12, with decimal in documentation), placeholder `X`, laterality (right/left/bilateral/unspecified), combination codes, and Excludes1 vs. Excludes2 (Excludes1 = "not coded here," Excludes2 = "not included here — both can be coded").
- **Z-codes** (factors influencing health) and **R-codes** (signs/symptoms) need careful handling for HCC and severity scoring — many do not risk-adjust.

### ICD-10-PCS (inpatient procedures, US only)

- 7-character alphanumeric structure: Section, Body System, Root Operation, Body Part, Approach, Device, Qualifier.
- Used **only on inpatient claims (837I)** for procedures. Outpatient procedures use CPT/HCPCS.
- Annual updates (Oct 1). Multi-axial structure means autocoders must compose codes from documentation rather than look up phrases.

### CPT (Current Procedural Terminology)

- Maintained by AMA — **licensed code set**; products that display, derive, or distribute CPT codes need an AMA license. Treat CPT data as licensed content in your build/release pipeline.
- Category I (5-digit numeric) = standard procedures and services. Category II (XXXXF) = optional performance-measurement tracking codes (do not bill alone). Category III (XXXXT) = emerging tech tracking codes.
- Updated **annually January 1** with quarterly errata; PLA codes (proprietary lab analyses) release quarterly.

### HCPCS Level II

- Maintained by CMS. Alphanumeric (letter + 4 digits): A (transport/medical/surgical supplies), B (enteral/parenteral), C (hospital outpatient), E (DME), G (temporary procedures/professional services), J (drugs administered other than oral; J-codes), K (DMERC temporary), L (orthotic/prosthetic), Q (temporary), S (commercial temp), T (Medicaid).
- **J-codes** are the billable units for separately payable Part B drugs (vs. NDC, which identifies the product). Claims often carry both NDC and J-code with units that differ — engineers must compute the J-code unit (e.g., "per 1 mg") from administered milligrams.
- Quarterly updates.

### CDT

- ADA's dental code set (`Dxxxx`). Used on 837D. Annually updated. Licensed by ADA.

### NDC (National Drug Code)

- FDA labeler-product-package identifier. Three segments: labeler (4-5 digits), product (3-4 digits), package (1-2 digits). Total 10 digits in the source; claims typically transmit an 11-digit normalized form (`5-4-2`).
- NDCs are **packaging-specific** — same drug, same strength can have hundreds of NDCs across manufacturers, repackagers, and package sizes. RxNorm normalizes across NDCs to a clinical drug concept.

### DRG / MS-DRG

- Medicare Severity DRGs group inpatient stays into ~750 payment categories based on PDx, SDx, procedures (PCS), discharge disposition, sex, age, and presence of MCCs/CCs.
- **Grouper software** (3M, Optum, Solventum) is licensed. Annual updates Oct 1 with the IPPS final rule.
- "All-Patient DRG" (AP-DRG) and APR-DRG (3M) are alternatives used by some state Medicaid programs and commercial payers — do not assume MS-DRG everywhere.

### APC (Ambulatory Payment Classification)

- CMS OPPS grouping for hospital outpatient claims. Composite APCs bundle related services; comprehensive APCs (C-APCs) pay a single rate for a primary procedure plus all adjunctive services on the same date.
- Status indicators on each HCPCS/CPT line determine payment behavior — engineers building outpatient pricers must consult the **OPPS Addendum B** (quarterly) for current values.

### Modifiers (verify each from current AMA/CMS source before building)

Modifiers are 2-character suffixes (CPT modifiers are numeric like `25`, `59`, `76`, `77`, `91`; HCPCS modifiers are alphanumeric like `GT`, `95`, `RT`, `LT`, `XE`, `XS`, `XU`, `XP`). They affect payment, bundling, laterality, telehealth status, and repeat procedures. Common ones engineers handle:

- **Significant separately identifiable E/M on same day as procedure** — used on the E/M line. Distinct from modifier for E/M leading to decision for surgery. Verify the exact modifier numbers and rules from current CPT/CMS guidance.
- **Distinct procedural service** — historically broad (`-59`) and now narrowed via X-modifiers (XE/XS/XP/XU) to indicate why services are distinct. CMS prefers X-modifiers when applicable.
- **Bilateral and laterality** — RT/LT/50 conventions vary by payer and code; some codes are inherently bilateral.
- **Repeat procedure same day same provider vs. different provider** — engineers must validate against payer rules.
- **Repeat clinical diagnostic lab test** — used when the same test is run more than once on the same day for medically necessary reasons.
- **Telehealth** — GT (historical) and 95 (synchronous audio/video) have shifted over time; current Medicare guidance changes by PHE/post-PHE — verify current value/rule from CMS before hard-coding.

> **Do not hard-code modifier semantics from memory.** Build a configurable modifier table and source it from current CPT/HCPCS data each year.

---

## NCCI Edits (National Correct Coding Initiative)

CMS's NCCI is two edit tables that all Medicare-derived editors implement:

- **Procedure-to-Procedure (PTP)** edits — pairs of HCPCS/CPT codes that should not be billed together. Each pair has a **modifier indicator** (0 = no modifier allowed to bypass; 1 = modifier may bypass when clinically appropriate; 9 = edit deleted).
- **Medically Unlikely Edits (MUE)** — maximum units of service per HCPCS/CPT per beneficiary per date of service. Each MUE has an **MAI** (1 = claim line edit, 2 = date-of-service edit absolute, 3 = date-of-service edit per medical review).
- Separate tables for **Practitioner**, **Outpatient Hospital (OPH)**, and **DME**. Different MUE values apply.
- Updated **quarterly**.

Engineers building claim scrubbers must load NCCI tables and apply edits before submission. Many denials originate from NCCI failures the provider could have caught pre-submission.

---

## LCDs and NCDs (Coverage Determinations)

- **NCD (National Coverage Determination)** — CMS-issued national policy on whether Medicare covers a service.
- **LCD (Local Coverage Determination)** — issued by MACs (Medicare Administrative Contractors) for their jurisdictions. Includes covered/non-covered ICD-10-CM codes for specific CPT/HCPCS services.
- **Articles** accompany LCDs with billing/coding details.
- Engineers building medical-necessity checking apply LCD/NCD logic at scheduling, order entry, or claim scrubbing. Source data is published on the CMS Medicare Coverage Database; structured LCD data is also available via **FHIR Da Vinci CRD** (see prior-authorization).

---

## E/M Coding (post-2021 revisions; verify current year)

CPT E/M office/outpatient codes were substantially revised in 2021, with **inpatient/observation** E/M revised in 2023. The level for office/outpatient is selected by **MDM (Medical Decision Making)** *or* **total time on the date of the encounter** — history and exam no longer drive the level (they must be medically appropriate but are not scored).

MDM has three elements: **Number and Complexity of Problems Addressed**, **Amount and/or Complexity of Data Reviewed and Analyzed**, **Risk of Complications and/or Morbidity or Mortality**. Two of three drive the level. Engineers building E/M-suggestion tools must extract these elements from the note — most CAC vendors use targeted NLP plus rules rather than free-form summarization.

Time-based coding sums **non-face-to-face** activities on the date of service (chart review, ordering, documentation, care coordination, prescription drug management, counseling, education).

**Other commonly-built care-management code families** (verify current CPT/HCPCS values from AMA/CMS):

- Annual Wellness Visit (Medicare HCPCS G-codes)
- Transitional Care Management (TCM)
- Chronic Care Management (CCM)
- Principal Care Management (PCM)
- Remote Physiologic Monitoring (RPM) and Remote Therapeutic Monitoring (RTM)
- Behavioral Health Integration (BHI) and Collaborative Care (CoCM)

Each has specific time, staffing, and consent requirements that engineers must encode in the workflow.

---

## Problem List vs. Encounter-Level Coding

- **Problem list** (SNOMED CT in modern EHRs) is the longitudinal clinical record of patient conditions. It is not directly billable.
- **Encounter-level diagnoses** (ICD-10-CM on the claim) describe what was addressed and managed **at this encounter**. The principal diagnosis on inpatient claims is "the condition established after study to be chiefly responsible for occasioning the admission."
- For **HCC capture**, conditions must be assessed and documented **annually** with **MEAT** evidence (Monitored, Evaluated, Assessed/Addressed, Treated) — pulling a code from the problem list without encounter-level assessment is a common compliance failure (see value-based-care).

---

## Computer-Assisted Coding (CAC) Architecture

A typical CAC pipeline:

1. **Document ingest** — clinical notes, op reports, pathology, radiology, discharge summaries — from EHR (FHIR DocumentReference, HL7 v2 MDM, or vendor APIs).
2. **Section segmentation** — identify HPI, ROS, exam, A/P, procedures performed.
3. **Concept extraction** — NLP (rules, transformers, or hybrid) extracts clinical concepts and links to SNOMED CT / RxNorm / LOINC.
4. **Code mapping** — map concepts to billing codes (ICD-10-CM via SNOMED CT → ICD map, or direct NLP-to-ICD). PCS and CPT often need rule-based composition from structured op-report fields.
5. **Specificity and modifier suggestion** — laterality, encounter type (initial/subsequent/sequela), bilateral, etc.
6. **Edit validation** — NCCI PTP/MUE, LCD/NCD medical necessity, payer edits, modifier conflicts.
7. **Coder review UI** — humans accept/reject suggestions; the system learns from corrections.
8. **Audit log** — every suggestion, every coder decision, every code change post-submission.

### Autocoding QA

- Track **precision** (of suggested codes, how many were kept), **recall** (of final codes, how many were suggested), and **specificity uplift** (how often the coder picks a more specific code than suggested).
- Stratify by service line, document type, and coder. Drift can be silent.
- Maintain a **gold-set** of expert-coded encounters for regression testing across model versions.
- Treat any change to the underlying terminology, ICD/CPT version, or model as a **release event** requiring re-validation.

### Bias and fairness

- Documentation patterns vary by provider, specialty, language, and patient population. An autocoder trained on one specialty or one EHR vendor can degrade silently on another.
- For HCC capture, audit suggestion rates by patient race/ethnicity/age to detect disparate over- or under-capture.

---

## Audit Programs Engineers Must Anticipate

- **RAC (Recovery Audit Contractor)** — Medicare FFS post-payment review of overpayments and underpayments. Approved issues are public on each RAC's website.
- **ZPIC / UPIC (Unified Program Integrity Contractor)** — fraud-focused integrity reviews; ZPIC has been consolidated into UPIC.
- **MAC medical review** — pre-pay and post-pay by the regional Medicare Administrative Contractor.
- **OIG (Office of Inspector General)** — annual work plan signals enforcement priorities; OIG self-disclosure protocol governs voluntary disclosures.
- **CERT (Comprehensive Error Rate Testing)** — improper payment rate calculation; not enforcement but a source of error-pattern data.
- **RADV (Risk Adjustment Data Validation)** — CMS audits of MA HCC submissions; see value-based-care.
- **Payer-specific SIU** (Special Investigations Unit) — commercial payer audits.

Engineers building documentation/coding systems must support **chart pulls**, **complete encounter packets**, **immutable timestamped audit trails**, and the ability to reproduce the documentation state at the time of submission.

---

## HCC Overlap

Risk-adjustment (HCC) coding piggybacks on standard ICD-10-CM coding but has its own annual recapture, MEAT requirements, and audit posture. The interaction between CAC and HCC vendors is a frequent source of double-counting, missed conditions, and compliance risk. See **value-based-care** for HCC model details, RAF score math, and RADV.

---

## What This Skill Does Not Do

- It does **not** assign codes for real patient encounters. Production coding requires a certified human coder (CPC, CCS, RHIA, RHIT) operating under current official guidelines and your compliance program.
- It does **not** replace your payer-specific policy library.
- It does **not** publish the current code set — license CPT/CDT from AMA/ADA and pull ICD-10-CM/PCS, HCPCS, and NCCI tables from CMS each release.

---

## Task-Specific Questions

1. Which code sets are in scope for this build (ICD-10-CM, ICD-10-PCS, CPT, HCPCS Level II, NDC, CDT, MS-DRG, APC)? Are any of them out of scope?
2. What clinical specialty or service line is the target (e.g., primary care E/M, orthopedic surgery, oncology infusion, behavioral health, dental, pharmacy)? Specialty drives which code families and modifiers dominate.
3. Is the product computer-assisted coding (CAC / suggest-and-let-coder-decide) or fully manual (validate-only / scrubber)? Or autocoding with auto-submit? The product role changes the QA and audit-trail requirements.
4. What is the coding workflow position — concurrent (during the stay), prospective (pre-bill final coding), or retrospective (audit / re-review after submission)?
5. What is the compliance posture — any prior RAC / OIG / UPIC / payer-SIU findings, current Corporate Integrity Agreement (CIA), or known historical denial patterns? This shapes how aggressive vs. conservative suggestion logic should be.
6. Is HCC capture (risk-adjustment coding) in scope, and if so for which population (Medicare Advantage, ACO REACH, MSSP, ACA HHS-HCC, Medicaid)? Different risk models behave differently; coordinate with `value-based-care`.
7. Which terminology services and reference data sources will the system rely on (CMS releases, AMA CPT license, ADA CDT license, SNOMED CT, RxNorm, LOINC, NCCI quarterly tables, LCD/NCD feed)? Confirm the update cadence each source ships on.

---

## Related Skills

- **billing-claims**: claim submission, EDI 837/835, denial codes, and how coding lands on a claim.
- **prior-authorization**: authorization workflows, Da Vinci CRD/DTR/PAS, and coverage requirements before service.
- **value-based-care**: HCC risk adjustment, RAF score, MEAT, RADV audit, and the VBC programs that depend on coding accuracy.
- **clinical-documentation**: CDI workflows that improve documentation quality before coding.
- **terminology-services**: SNOMED CT / LOINC / RxNorm / ICD value sets and concept maps.
- **fhir-integration**: how coded data moves over FHIR (Condition, Procedure, Claim, ChargeItem, ExplanationOfBenefit).
- **healthcare-context**: organization, setting, payer mix, and product role that scope every recommendation here.
