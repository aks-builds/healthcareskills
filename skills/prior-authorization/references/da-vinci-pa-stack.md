# Da Vinci Prior Authorization Stack (CRD, DTR, PAS)

Reference for engineers building FHIR-based prior authorization. Describes the three HL7 Da Vinci implementation guides — CRD, DTR, PAS — and how they chain at order entry, documentation, and submission. Does not enumerate FHIR profile field-level detail or specific CDS Hook card structures — those are versioned in the IGs.

> Verify against the **current published version** of each Da Vinci IG at https://hl7.org/fhir/us/davinci-*  and the **payer's PAS implementation profile** before encoding specific profile details.

## Contents

- Why Da Vinci exists
- The three IGs at a glance
- CRD (Coverage Requirements Discovery)
- DTR (Documentation Templates and Rules)
- PAS (Prior Authorization Support)
- How they chain together
- Workflow walkthrough
- When each fires
- HIPAA / X12 278 bridging
- Engineering integration patterns
- Common misconceptions

---

## Why Da Vinci Exists

Legacy prior auth runs across fax, payer portals, and X12 278 — none of which integrates well with the EHR at the point of order entry. The Da Vinci Project (an HL7 accelerator) produced a stack of FHIR-based implementation guides intended to:

- Surface PA requirements **inside the EHR** at order entry
- Pre-fill payer questionnaires from **existing structured FHIR data**
- Submit prior auth as a **FHIR transaction** while preserving the HIPAA-mandated X12 278 as the legally-recognized record

This stack is the technical foundation for many of the CMS-0057-F Prior Authorization API requirements.

---

## The Three IGs at a Glance

| IG | Role | Trigger | Transport |
|---|---|---|---|
| **CRD** (Coverage Requirements Discovery) | At order time, surface whether PA is required, what documentation is needed, and what alternatives are covered. | **CDS Hooks** (`order-select`, `order-sign`) — the EHR calls the payer's CDS service. | CDS Hooks (JSON over HTTPS). Cards can return SMART app links into DTR. |
| **DTR** (Documentation Templates and Rules) | Fetch the payer's questionnaire and prerequisite rules, pre-fill from the EHR's FHIR data, and surface only the unanswered questions to the clinician. | Launched from a CRD card, EHR menu, or order workflow. | FHIR — Questionnaire + Library (CQL) + ValueSets pulled from the payer; QuestionnaireResponse built locally. Often runs as a SMART on FHIR app or EHR-native. |
| **PAS** (Prior Authorization Support) | Submit the prior auth (with the DTR-completed questionnaire and any attachments) to the payer/UMO; receive decision. | After DTR completion (or when launched directly with a complete payload). | FHIR Claim / ClaimResponse profiles. The payer's intermediary transforms to/from X12 278. |

---

## CRD (Coverage Requirements Discovery)

### Purpose

At order entry, before any submission, ask the payer "what does this service require?" The answer can be:

- No PA required for this service / member combination
- PA required — here is the questionnaire URL (DTR)
- PA not required but documentation must be retained
- Service is non-covered — here is the alternative

### Mechanics

- Uses **CDS Hooks** — an open spec for invoking decision-support services from the EHR.
- Common hooks: `order-select` (clinician adds an order), `order-sign` (clinician signs).
- The EHR sends the patient context, encounter, and proposed order to the payer's CDS service URL.
- The payer's CDS service returns one or more **cards** — display text, alternative-drug suggestions, links into DTR, etc.

### Engineering notes

- Stand up a **CDS Hooks consumer** in the EHR (most major EHRs support CDS Hooks natively or via SMART).
- Maintain a **CDS service catalog** — which payer/plan exposes which CDS service URL, with credentials and SLA.
- CRD responses are **advisory** — they describe the requirement, not the decision. Do not record a CRD response as a PA approval.

### Limitations

- Not all payers expose CRD endpoints.
- Latency at order entry must be low; design for timeouts and graceful fallback to legacy PA-requirement checks.

> Verify against the current Da Vinci CRD IG and the payer-specific CDS service contract.

---

## DTR (Documentation Templates and Rules)

### Purpose

Once CRD (or another trigger) indicates PA is required, fetch the **payer's questionnaire** and **prerequisite logic**, run it against the patient's FHIR record to pre-fill answers, and surface only the **gaps** to the clinician.

### Mechanics

- Payer publishes a **FHIR `Questionnaire`** + **`Library`** (CQL rules) + **`ValueSet`**s.
- DTR client (SMART on FHIR app or EHR-native module) pulls these resources.
- Client executes the CQL against the patient's FHIR record (locally on the EHR's FHIR server) to pre-populate `QuestionnaireResponse` items.
- Clinician reviews and answers only the items not pre-filled.

### Engineering notes

- **CQL execution** can run client-side (in the SMART app) or server-side. Most reference implementations run client-side for speed and PHI minimization.
- **Pre-fill provenance** — record which answers were system-filled vs. clinician-entered. Clinician must be able to override pre-filled answers with a documented reason.
- **Version pinning** — pin to the payer's questionnaire and library version; payers update over time.
- **PHI minimization** — only the data needed to answer the questionnaire should leave the EHR; the questionnaire response can then be packaged for PAS submission.

### Limitations

- Many payers do not yet publish DTR-ready Questionnaires; legacy free-form clinical narratives still dominate.
- CQL authoring is a specialized skill; payer-published libraries may have defects that require operational workarounds.

> Verify against the current Da Vinci DTR IG.

---

## PAS (Prior Authorization Support)

### Purpose

Submit the prior auth — with the DTR-completed questionnaire and any attachments — to the payer / UMO, and receive the decision.

### Mechanics

- Uses FHIR `Claim` and `ClaimResponse` profiles, plus supporting resources (Patient, Coverage, Encounter, ServiceRequest, MedicationRequest, DocumentReference for attachments, QuestionnaireResponse from DTR).
- The payer's intermediary receives the FHIR bundle and **transforms it to X12 278** for downstream adjudication systems, because X12 278 is the HIPAA-mandated standard for prior authorization.
- The payer's decision flows back as `ClaimResponse` — approval, denial, pend, or partial approval — with authorization number, valid date range, authorized codes and units, and conditions/limits.

### Engineering notes

- **HIPAA conformance**: PAS does not displace X12 278 as the legally-recognized transaction until/unless HHS designates FHIR as an additional standard. The payer-side X12 278 mapping is the authoritative record.
- **Identifiers**: maintain correlation between your PAS submission ID, the payer's authorization number, and the underlying X12 278 reference number — this is essential for audit and for matching the authorization to the eventual claim.
- **Attachments**: attach via FHIR `DocumentReference` linked from the Claim. Out-of-band attachment (X12 275 / fax / portal upload) remains common for clinical evidence too large or complex for FHIR.

### Limitations

- Adoption is uneven. Many payers still require fax/portal/278 for certain service categories.
- Per-payer profile variation — even within PAS, payers may require their own profile extensions or specific code systems.

> Verify against the current Da Vinci PAS IG and the payer's PAS-specific implementation profile.

---

## How They Chain Together

```
EHR order entry
   │
   │ CDS Hooks (order-select / order-sign)
   ▼
Payer CRD CDS service
   │
   │ card: PA required → DTR app link
   ▼
DTR SMART app (or EHR-native)
   │ pulls Questionnaire + Library + ValueSet from payer
   │ executes CQL against EHR's FHIR data
   │ surfaces gaps to clinician
   ▼
QuestionnaireResponse complete
   │
   │ PAS submission (FHIR Claim + supporting bundle)
   ▼
Payer PAS endpoint (intermediary transforms to X12 278)
   │
   │ adjudication
   ▼
ClaimResponse (approval / denial / pend) → back to EHR
   │
   │ authorization stored as structured object
   ▼
Service rendered → claim submitted with PA number → adjudicated against PA
```

---

## Workflow Walkthrough

1. **Clinician orders MRI** for a member.
2. **EHR fires CDS Hooks `order-select`**, calling the payer's CRD service with the order context.
3. **CRD returns a card** indicating PA is required and providing a link to the DTR Questionnaire.
4. **Clinician clicks the link**; the DTR SMART app launches with patient context.
5. **DTR fetches** the Questionnaire, Library, and ValueSets from the payer.
6. **DTR runs CQL** against the patient's FHIR record (Conditions, Observations, MedicationRequests, prior imaging) to pre-populate answers.
7. **Clinician sees only the gaps** — questions the CQL could not answer (e.g., subjective symptoms, plan of care).
8. **Clinician completes** the gaps, attaches any required documents (DocumentReference).
9. **DTR finalizes** the QuestionnaireResponse and hands off to PAS submission.
10. **PAS submission** sends a FHIR Claim bundle to the payer's PAS endpoint.
11. **Payer intermediary** transforms to X12 278; adjudication runs.
12. **ClaimResponse** returned — approval with authorization number, valid date range, authorized codes/units.
13. **EHR stores** authorization as a structured object linked to the order.
14. **Service rendered**; **claim submitted** with PA number; payer matches PA against claim and pays.

---

## When Each Fires

- **CRD** — at order time, before any submission. Result is **advisory**, not a PA decision.
- **DTR** — when CRD (or another trigger) indicates PA is required and a Questionnaire is available. Result is a completed QuestionnaireResponse, not a PA decision.
- **PAS** — when the submission payload is ready. Result is the actual PA decision (approval / denial / pend).

Common mistake: treating a CRD response or a DTR completion as a PA decision. Neither is. **Only PAS (or X12 278 response, or a fax/portal decision) is a PA decision.**

---

## HIPAA / X12 278 Bridging

PAS uses FHIR over the wire, but the **legally-recognized HIPAA transaction is X12 278**. The payer's PAS intermediary performs FHIR → X12 278 transformation on submission and X12 278 → FHIR on response.

Engineering implications:

- The 278 reference number and the FHIR resource identifiers must be reconcilable for audit.
- If HHS designates FHIR as an additional standard in the future, this bridging requirement may change — track regulatory developments.
- Companion guides may specify which side (PAS profile vs. 278 implementation) governs in case of conflict.

> Verify against current HHS HIPAA standards designations and the payer's PAS profile.

---

## Engineering Integration Patterns

### Channel-agnostic state machine

Even with full Da Vinci adoption, legacy channels (fax, portal, X12 278 direct, NCPDP SCRIPT ePA) remain in use for many payers and service categories. Build a **channel-agnostic PA state machine** where Da Vinci is one adapter among several.

### Authorization as structured object

Whatever the channel, store the authorization decision as a structured object:

- Authorization number
- Valid date range
- Authorized codes (CPT/HCPCS/NDC/CDT)
- Authorized units
- Authorized providers (ordering, rendering)
- Authorized place of service
- Conditions / limits
- Source channel and source reference (PAS ClaimResponse, X12 278 reference, portal screenshot, fax confirmation)

Never reduce a PA to a boolean.

### Audit trail

Every CRD call, DTR launch, PAS submission, and decision must be timestamped, attributed, and reproducible. This is required for both internal QA and regulator scrutiny (CMS, state DOI, NCQA, URAC).

### Latency budgets

CRD at order entry has a tight latency budget — design for graceful fallback if the payer's CDS service is slow or unavailable.

### PHI minimization

Send the minimum necessary patient data at each step:
- CRD typically needs limited context (member, plan, order).
- DTR fetches resources but executes CQL locally — keep patient data inside the EHR perimeter.
- PAS submits only what the payer requires for adjudication.

---

## Common Misconceptions

- **"Da Vinci replaces X12 278."** It does not. PAS is a FHIR front end; the legally-recognized HIPAA transaction is still X12 278.
- **"CRD is the PA approval."** No — CRD describes the requirement. Only PAS / 278 / equivalent returns the decision.
- **"DTR pre-fill eliminates the clinician."** No — clinician confirms pre-filled answers and answers the gaps; clinician sign-off is required.
- **"All payers support PAS."** No — adoption is uneven and varies by service category. Plan for legacy channels in parallel.
- **"FHIR replaces all attachments."** No — many attachments still move out-of-band (X12 275 with CDA, fax, portal upload) when too large or complex for the FHIR bundle.

> Verify all behavior against the current Da Vinci CRD/DTR/PAS IGs and the payer's published implementation profiles before committing to a design.
