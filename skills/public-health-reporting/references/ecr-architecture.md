# Electronic Case Reporting (eCR) Architecture

Reference for how eCR flows from the EHR to public health, the role of the APHL AIMS Platform and RCKMS, and what supporting gateways (NEDSS, state PHA systems) typically do. Verify current IG versions, AIMS service catalog, and jurisdictional onboarding requirements against current APHL, CDC, CSTE, and state public health agency documentation.

## Contents

- High-level data flow
- Triggering with RCTC
- eICR generation
- APHL AIMS Platform role
- RCKMS reportability decisioning
- Reportability Response (RR)
- NEDSS and state public health gateways
- Onboarding stages
- Operational considerations

---

## High-level data flow

```
                       (trigger match on RCTC)
        EHR ────────────────────────────────────► eICR (FHIR Bundle or CDA)
         ▲                                                │
         │                                                ▼
         │                                       APHL AIMS Platform
         │                                                │
         │                                                ▼
         │                                              RCKMS
         │                                  (jurisdiction-aware reportability rules)
         │                                                │
         │                                                ▼
         │                                Jurisdiction PHA(s) — state, local, tribal
         │                                  (often NEDSS or state-specific system)
         │                                                │
         └─────────────── Reportability Response (RR) ◄──┘
```

The architecture is intentionally centralized: rather than each EHR hard-coding per-jurisdiction reportability rules, EHRs send one eICR to AIMS and let RCKMS decide where (if anywhere) the case is actually reportable.

---

## Triggering with RCTC

The **Reportable Conditions Trigger Codes (RCTC)** value sets, published on VSAC, drive EHR-side triggering:

- LOINC for laboratory test orders and results.
- SNOMED CT and ICD-10-CM for diagnoses and problems.
- RxNorm for medications.
- CPT / HCPCS for procedures where applicable.

EHRs subscribe to RCTC and fire a trigger when a relevant clinical event lands. The trigger is condition-agnostic at the EHR side — RCTC encodes "this code is relevant to one or more reportable conditions somewhere"; the jurisdictional reportability decision happens downstream at RCKMS.

Verify RCTC value-set publication dates regularly; the value sets are updated as new conditions, codes, and outbreaks emerge.

---

## eICR generation

The **eICR (electronic Initial Case Report)** is the structured report the EHR generates on trigger:

- Originally specified as an HL7 CDA R2 document via the C-CDA-based eICR IG.
- The FHIR-based **US Public Health eCR IG** (eCR FHIR R4) is the current direction; both are still in active use depending on EHR version. Verify current published versions on HL7.

Typical eICR contents:

- Patient demographics (including race/ethnicity per CDC code sets where the EHR captures them).
- Encounter details and chief complaint.
- Problems, diagnoses, allergies.
- Relevant laboratory results.
- Medications administered or prescribed.
- Provider and facility identifiers.
- The specific trigger codes that fired.

The EHR should treat eICR generation as automatic and continuous, not manual — a clinician should not need to "submit" each case.

---

## APHL AIMS Platform role

The **APHL AIMS (AIMS = APHL Informatics Messaging Services) Platform** is the national routing hub for eCR and several other streams:

- Receives eICRs from EHRs (and other public health messages).
- Forwards each eICR to RCKMS for reportability evaluation.
- Routes to the correct jurisdiction(s) based on RCKMS output.
- Returns the Reportability Response to the EHR.
- Provides validation, monitoring, and onboarding tooling.

AIMS supports multiple transport options. Verify current onboarding requirements, authentication (mTLS, OAuth, API key), and supported message formats against the APHL AIMS Platform onboarding documentation.

---

## RCKMS reportability decisioning

The **Reportable Conditions Knowledge Management System (RCKMS)** is the rules engine maintained by CSTE in partnership with APHL:

- Encodes jurisdiction-aware reportability logic — "is this case reportable to this state/locality/tribe for this condition?"
- Evaluates each incoming eICR against the rules for the jurisdiction(s) implicated by the patient's address, encounter facility, and trigger.
- Produces a structured determination (reportable / may be reportable / not reportable for each jurisdiction evaluated) plus any supplemental information requests.

The condition list and trigger codes RCKMS evaluates against are maintained by CSTE. Verify current condition coverage and rule version against RCKMS and CSTE publications.

---

## Reportability Response (RR)

The **Reportability Response (RR)** is the structured document returned to the EHR:

- Confirms reportability per jurisdiction (or that no jurisdiction found the case reportable).
- Lists the receiving jurisdiction(s) and contact information.
- May request supplemental data (e.g., follow-up labs, exposure history).
- Documents the version of RCKMS rules used.

EHRs should consume the RR, surface it to clinical and public health users where appropriate, and treat any supplemental requests as actionable work items — not fire-and-forget.

---

## NEDSS and state public health gateways

Many state public health agencies operate on the **NEDSS Base System (NBS)**, a CDC-supplied surveillance platform, or on commercial / custom equivalents (e.g., MAVEN, state-built systems with names like CalREDIE, FL Merlin, NY ECLRS for the lab side).

- The eICR landing in a state from AIMS typically routes into NBS or the state's equivalent for case management.
- State agencies may also accept direct submissions for low-volume sources or pilots.
- The state public health gateway is responsible for state-level acknowledgment, case management workflow, and onward CDC submission.

Match each integration to the receiving authority's onboarding documentation; gateways differ in transport, authentication, batch vs. real-time, and acknowledgment expectations.

---

## Onboarding stages

Typical eCR onboarding stages (verify exact names and gates with APHL AIMS and the receiving jurisdiction):

1. **Registration** with APHL AIMS and the receiving PHA(s).
2. **Connectivity test** — establish secure transport and authentication.
3. **Message validation** — send test eICRs against synthetic patients; iterate on schema and content issues.
4. **Pre-production / validation** — produce real-volume messages routed to test instances; PHA reviews quality.
5. **Production** — go-live; ongoing monitoring.

For Promoting Interoperability credit, **active engagement** at one of the defined stages (verify CMS current rule) is what counts.

---

## Operational considerations

- **Identity reconciliation**: Public health rarely has an enterprise MPI. Design for repeated submissions, duplicate handling at the jurisdiction, and patient updates over time.
- **Privacy**: HIPAA permits disclosure to public health authorities under 45 CFR §164.512(b), but minimum necessary and state law overlays still apply. Verify the specific authority's legal basis before sending.
- **Audit**: Every outbound eICR and inbound RR must produce an audit record.
- **Test data**: Synthetic patients only for non-production stages.
- **Resilience**: Queue eICRs at the EHR side; AIMS or jurisdiction outages should not lose cases.

Always verify IG versions, RCTC value-set publication dates, AIMS service offerings, RCKMS coverage, and per-jurisdiction onboarding requirements against current authoritative sources.
