# TEFCA Exchange Purposes

The Common Agreement enumerates the Exchange Purposes (XPs) under which information may be exchanged across the TEFCA network of networks. Each XP carries different policy obligations, minimum-necessary scoping, and audit expectations.

**Always verify the current XP list and the current SOPs against the RCE's published documents** — the framework is evolving and Common Agreement v2.0 added or restructured XPs (notably Required Information for Coverage).

## Contents

- How XPs Work
- Treatment
- Payment
- Healthcare Operations
- Public Health
- Government Benefits Determination
- Individual Access Services (IAS)
- Required Information for Coverage / Eligibility
- XP Selection in Practice
- Audit Requirements
- State Law and Sensitive Categories
- Practical Notes

---

## How XPs Work

When a Participant / QHIN issues a query or push, the request asserts an Exchange Purpose. The responding Participant / QHIN must:

1. Verify the asserted XP is supported by the responder.
2. Apply the minimum-necessary scoping for that XP.
3. Apply any consent / privacy obligations applicable to that XP.
4. Log an audit event tagged with the XP, requester, responder, patient, and time.
5. Return data according to the XP's content rules.

The XP is **not** a free-form purpose statement — it is a defined value with specific policy and SOP obligations.

---

## Treatment

### Definition

Treatment exchange — the provision, coordination, or management of healthcare and related services by one or more healthcare providers, including consultation between providers, referrals, and care planning.

### Common Use

- A specialist queries a referring provider for clinical context before a consult
- An emergency department pulls a patient summary on a presenting patient
- A receiving hospital pulls records from the transferring hospital
- A care manager retrieves recent labs and meds for chronic-disease management

### Obligations (high level — verify SOPs)

- HIPAA Treatment provisions apply; no patient authorization typically required
- Audit events with full per-disclosure metadata
- Minimum-necessary applies but is interpreted broadly for treatment

### Example Flow

1. ED doctor's EHR queries via the local QHIN.
2. Query carries XP=Treatment, patient demographics, and (after PDQ / matching) the patient ID.
3. Responding QHINs return matched records from their participants.
4. Records flow back as C-CDA (or FHIR in the FHIR Roadmap).
5. ED EHR ingests / displays / reconciles.

---

## Payment

### Definition

Activities undertaken by a health plan or provider for or related to payment — claims management, eligibility verification, prior authorization, utilization review, audits, billing.

### Common Use

- Payer queries patient records to adjudicate a claim
- Provider sends prior authorization request with supporting documentation
- Eligibility verification before a procedure
- Coordination of benefits between primary and secondary insurers

### Obligations

- HIPAA Payment provisions apply
- Minimum-necessary scoping is generally tighter than Treatment
- Stronger audit and accounting requirements

### Example Flow

1. Provider submits prior auth via Da Vinci PAS or via QHIN query under XP=Payment.
2. Payer responds with auth status and required documentation.
3. Both sides audit.

---

## Healthcare Operations

### Definition

A wide category covering quality assessment and improvement, credentialing, care coordination, population health, training, accreditation, legal services, business management.

### Common Use

- Quality measurement and reporting
- Population-health analytics on enrolled patients
- Care coordination across networks for a covered population
- Risk adjustment and HEDIS-style measure calculation
- Credentialing and provider directory exchange

### Obligations

- HIPAA Healthcare Operations applies
- Minimum-necessary scoping is strict and granular
- Often bulk / batch rather than single-patient — Bulk Data Access ($export) is the FHIR mechanism

### Example Flow

1. Payer requests a yearly bulk export of patient records for HEDIS measure calculation.
2. Request is scoped via Group (enrolled members) and resource types (Patient, Condition, Observation, MedicationRequest, etc.).
3. Responding QHIN authenticates via SMART Backend Services and returns NDJSON.

---

## Public Health

### Definition

Activities authorized by law for public health surveillance, investigation, reporting, intervention.

### Common Use

- Electronic Case Reporting (eCR) of reportable conditions
- Immunization registry submissions
- Syndromic surveillance
- Reportable lab result submission
- Birth / death / fetal-death reporting (vital records)
- Cancer / disease registry submissions

### Obligations

- HIPAA Public Health permits disclosure to public health authorities authorized by law
- Triggered by reportable condition criteria (RCKMS / state-specific)
- Audit events tagged with the receiving public health authority

### Example Flow

1. Provider EHR detects a reportable condition (per RCKMS rules).
2. eICR (CDA) or eCR (FHIR) is generated and sent through the provider's QHIN under XP=Public Health.
3. APHL AIMS or state public health intermediary routes to the appropriate jurisdictional public health authority.
4. Public health authority responds with a Reportability Response (RR) describing reportability and follow-up actions.

---

## Government Benefits Determination

### Definition

Determining eligibility for federal or state benefit programs — Social Security disability, VA benefits, Medicaid eligibility, etc.

### Common Use

- SSA queries medical records to adjudicate disability claims
- State Medicaid agencies query for eligibility documentation
- VA accesses non-VA records on behalf of veterans

### Obligations

- Specific federal authority typically authorizes the disclosure
- Strict audit and patient-notice expectations in some cases
- Often patient consent or notice is required depending on the program

### Example Flow

1. SSA receives a disability application referencing medical evidence from a hospital.
2. SSA queries via its QHIN under XP=Government Benefits Determination.
3. Hospital's QHIN returns records.
4. Both sides audit; the patient may receive a notice depending on program rules.

---

## Individual Access Services (IAS)

### Definition

Patient-directed retrieval of their own records via an IAS Provider that they have authorized.

### Common Use

- Personal health records (PHR) apps
- Apps that aggregate a consumer's records across providers
- Third-party services helping a consumer collect records for a second opinion, life-insurance application, or transition of care
- Digital wallets

### Obligations

- The IAS Provider must perform identity proofing (NIST IAL2) and enroll an authenticator (NIST AAL2). Verify against the current IAS SOP.
- The IAS request asserts the patient's identity to responding QHINs / Participants
- Strong audit with patient-visible disclosure accounting
- Patient must designate which data and from which sources

### Example Flow

1. Patient creates an account with an IAS Provider; identity-proofs to IAL2.
2. Patient enrolls a multi-factor authenticator (AAL2).
3. Patient designates the records they want and the sources to query.
4. IAS Provider issues query under XP=IAS through its QHIN.
5. Responding QHIN/Participant authenticates the IAS request, verifies the identity assertion, returns data subject to applicable law and consent.
6. Audit events on both ends; patient can see the disclosure log.

### IAS-Specific Risks

- Cost and operational complexity of IAL2 — building in-house is expensive
- Misuse / fraud — strong fraud detection on enrollment
- State law on sensitive data — IAS does not override Part 2, behavioral health, reproductive, HIV, genetic, minor consent rules
- Patient understanding of what they're authorizing — informed-consent UX matters

---

## Required Information for Coverage / Eligibility

(Added or formalized under Common Agreement v2.0 — verify scope and current SOP language.)

### Definition

Exchange of information needed to determine coverage or eligibility for benefits — a hybrid of Payment and Government Benefits Determination scoped to coverage determination specifically.

### Common Use

- Eligibility verification with greater data depth than core Payment XP
- Coverage determination support across payers

Verify the exact scope of this XP against the current Common Agreement and SOPs before designing for it.

---

## XP Selection in Practice

When designing a query or push:

1. Identify the **purpose** of the disclosure from the requester's perspective.
2. Map it to the most specific XP that fits — XPs are not interchangeable.
3. Confirm the responding side supports that XP and has data scoped accordingly.
4. Apply the corresponding consent, minimum-necessary, and audit obligations.

If the purpose doesn't cleanly fit an XP, the exchange may not be permitted under TEFCA — fall back to direct contractual arrangements or other mechanisms.

---

## Audit Requirements

Every XP-tagged exchange MUST produce an audit record with:

- Requester identity (organization + user where applicable)
- Responder identity
- Patient subject
- XP asserted
- Timestamp
- Records / resource types disclosed
- Outcome (success / failure / partial)

Retention requirements come from the Common Agreement — verify the current value. Audit records support HIPAA disclosure accounting (45 CFR 164.528).

---

## State Law and Sensitive Categories

TEFCA does **not preempt** state law on sensitive data categories. The responder must apply applicable state law for:

- **42 CFR Part 2** — Substance Use Disorder records (federal but with separate consent requirements)
- **Behavioral / Mental Health** — state-specific consent rules
- **Reproductive Health** — state-specific protections (increasingly variable)
- **HIV / AIDS** — state-specific consent
- **Genetic Information** — GINA + state-specific
- **Minor Consent** — state-specific (especially around adolescent reproductive, mental health, SUD)
- **Sexual Assault / Domestic Violence** — state-specific

These require segmentation in the responding system or blanket suppression depending on capability. Many EHRs today suppress entire encounters rather than segment data — operationally messy but currently common.

---

## Practical Notes

- **XPs are not free-form** — assert one of the defined values
- **Most exchange today is Treatment** — the highest-volume XP
- **IAS is the highest-friction XP** because of IAL2 requirements
- **Public Health is rising** as eCR adoption grows
- **Healthcare Operations bulk exchange** is the next high-growth area as payers and providers integrate
- **Sensitive-category data** is the persistent operational risk — design for segmentation early
- **The XP list and SOPs change** — pin to a specific Common Agreement and SOP version in your design docs, and revisit periodically
