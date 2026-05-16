# CMS-0057-F Summary (Interoperability and Prior Authorization Final Rule)

Reference for engineers planning CMS-0057-F compliance work. Describes the rule's scope, the affected payer types, the four required APIs, and the major operational requirements. Does **not** enumerate specific effective dates, decision-timing thresholds, or metric definitions — those are in the rule text and have been updated through sub-regulatory guidance.

> Verify all scope, effective dates, decision-timing requirements, metric definitions, and reporting requirements against the **current CMS rule text**, the **Federal Register publication**, and **CMS sub-regulatory guidance** before encoding deadlines or workflows. Do not encode any date from memory.

## Contents

- What the rule is
- Affected payers (scope)
- Out-of-scope payers
- Four required APIs
- Decision-timing requirements
- Public reporting
- Sub-regulatory guidance and ongoing updates
- Engineering implications
- Coordination with Da Vinci stack

---

## What the Rule Is

CMS-0057-F is the **CMS Interoperability and Prior Authorization Final Rule**, finalized in 2024, building on earlier CMS interoperability rules (e.g., the 2020 CMS-9115-F Interoperability and Patient Access Final Rule).

It addresses three interlocking goals:

1. **Reduce prior authorization burden** on providers and members.
2. **Improve interoperability** of payer data with patients, providers, and other payers.
3. **Increase transparency** of prior auth performance.

It is phased in over multiple years, with the major API and decision-timing provisions applying on staggered effective dates.

> Verify the current text and effective dates of CMS-0057-F at the CMS Interoperability page and the corresponding Federal Register notice. Provisions may be modified by sub-regulatory guidance.

---

## Affected Payers (Scope)

The rule applies to:

- **Medicare Advantage (MA) organizations**
- **State Medicaid fee-for-service programs**
- **State CHIP fee-for-service programs**
- **Medicaid managed care plans**
- **CHIP managed care entities**
- **Qualified Health Plan (QHP) issuers on the Federally-Facilitated Exchanges (FFE)**

These are the "**impacted payers**" referenced throughout the rule.

> Verify whether state-based marketplace (SBM) QHPs and any state-specific Medicaid arrangements fall in or out of scope under the current rule and any state-level mirror rules.

---

## Out-of-Scope Payers

The rule does **not** apply to:

- **Commercial off-Exchange plans** (employer-sponsored, individual market outside the FFE).
- **ERISA self-funded employer plans**.
- **Medicare fee-for-service** (Medicare Part A/B traditional FFS — Medicare itself rather than the impacted MA organizations).
- **Workers' compensation, auto medical, and other non-HIPAA payers**.

These payers may **voluntarily** adopt Da Vinci / FHIR PA standards but are not bound by CMS-0057-F.

> Verify scope edge cases (e.g., dual-eligible special-needs plans, Medicare cost plans, certain PACE entities) against the current rule text.

---

## Four Required APIs

CMS-0057-F builds on and expands the Patient Access API requirements from the 2020 Interoperability rule, adding three additional APIs. All four are **FHIR-based** (typically aligned with US Core, CARIN BB, and Da Vinci IGs).

### 1. Patient Access API (enhanced)

- Carries forward from the 2020 Interoperability rule.
- Members can authorize third-party apps to retrieve their claims, encounter, clinical, and (new under -0057-F) prior authorization data from the impacted payer.
- Aligned with **CARIN Blue Button** and US Core.
- New under -0057-F: must include PA information about the requesting member.

> Verify Patient Access API scope and PA-information requirements against the current rule text.

### 2. Provider Access API

- Allows in-network providers to retrieve member data (claims, encounter, clinical, PA) from the impacted payer for the providers' attributed members.
- Supports better care coordination and reduces duplicate data collection.
- Member opt-out mechanisms are specified in the rule.

> Verify Provider Access API attribution criteria, opt-out mechanisms, and scope against the current rule text.

### 3. Payer-to-Payer API

- When a member changes from one impacted payer to another, the prior payer must forward the member's data (claims, encounter, clinical, PA) to the new payer.
- Aligned with **Da Vinci PDex** payer-to-payer profiles.
- Member consent / opt-in mechanisms are specified in the rule.

> Verify Payer-to-Payer API consent mechanisms, scope, and effective dates against the current rule text.

### 4. Prior Authorization API

- FHIR-based PA submission and status API.
- Aligned with **Da Vinci CRD/DTR/PAS** stack.
- Includes APIs for:
  - PA requirement lookup
  - PA submission
  - PA status retrieval
- Continues to coexist with the HIPAA-mandated X12 278 transaction (payer intermediaries transform between FHIR and 278).

> Verify Prior Authorization API specific requirements and conformance profiles against the current rule text and the referenced Da Vinci IG versions.

---

## Decision-Timing Requirements

The rule tightens prior authorization **decision timing** for impacted payers, generally distinguishing:

- **Expedited (urgent) PA decisions** — shortened compared to historical norms.
- **Standard (non-urgent) PA decisions** — also shortened compared to historical norms.

The exact hour/day thresholds and the definitions of "received complete information" are in the rule text.

> Do **not** encode specific decision-timing thresholds from memory. Verify against the current CMS rule text and any sub-regulatory guidance. These have been updated through rulemaking and clarifications.

---

## Reason for Denial

Impacted payers must communicate a **specific reason for denial** when a PA is denied, in plain language understandable by the member and provider. This supports appeals and reduces ambiguity.

> Verify the specific format and content requirements for denial reasons against the current rule text.

---

## Public Reporting

Impacted payers must **publicly report** PA performance metrics on their websites, including (at a high level — verify specifics):

- Number of PA requests by category
- Approval / denial / pending rates
- Average decision times
- Other operational metrics specified by CMS

The public reporting requirement creates external visibility into payer PA performance and is intended to drive market accountability.

> Verify the specific metrics list, calculation methods, and publication cadence against the current rule text and any CMS reporting templates.

---

## Sub-Regulatory Guidance and Ongoing Updates

CMS publishes sub-regulatory guidance that:

- Clarifies ambiguities in the final rule.
- Provides operational templates (e.g., metric calculation definitions).
- Announces enforcement discretion windows where applicable.
- Aligns with related rules (e.g., Medicare Advantage rate notices, ONC certification updates).

Engineers must track:

- The **Federal Register** for amendments.
- The **CMS Interoperability and Patient Access Final Rule page**.
- **ONC certification** updates that touch certified EHR technology behavior.

> Verify against current CMS announcements before scoping releases tied to specific effective dates.

---

## Engineering Implications

### Roadmap

Build a multi-quarter roadmap with **per-API milestones** tied to the effective dates currently in force. Treat each API as an independent track:

- **Patient Access API enhancement** — add PA data; align with CARIN BB extensions for PA.
- **Provider Access API** — net-new for most payers; requires attribution, authentication, and audit.
- **Payer-to-Payer API** — net-new; requires PDex alignment, member consent flow, prior-payer outreach.
- **Prior Authorization API** — Da Vinci CRD/DTR/PAS implementation; coordination with X12 278 bridging.

### Workflow changes

- **PA-requirement lookup** (CRD) — payers must expose a real-time service; providers must consume it.
- **DTR Questionnaire publication** — payers must publish per-service questionnaires; providers must consume and pre-fill.
- **PAS submission and status** — payers must accept FHIR submissions; providers must build PAS clients.
- **Decision-timing compliance** — both sides must instrument turnaround times against the new thresholds.

### Metric instrumentation

Public reporting requires:

- Per-request timestamps (received, complete, decided).
- Decision categorization (approved, denied, partially approved, withdrawn).
- Service-category dimensions for reporting groupings.
- Validation that the metric calculations match the CMS-specified method exactly.

### Compliance posture

- **Audit trail** — every API call, every decision, every metric calculation must be reproducible.
- **Identity and consent** — Patient Access, Provider Access, and Payer-to-Payer all involve member identity/authorization or provider authentication. SMART on FHIR / OAuth 2 / OpenID Connect patterns apply.
- **PHI minimization** — return only what the requesting party is authorized to see.
- **Breach response** — API-based PHI access expands the surface area; coordinate with `hipaa-compliance` and security teams.

---

## Coordination with Da Vinci Stack

CMS-0057-F is designed to be implemented using the **Da Vinci CRD/DTR/PAS** stack for the Prior Authorization API and **Da Vinci PDex** for the Payer-to-Payer and Provider Access APIs (which align with PDex profiles).

See the companion reference `da-vinci-pa-stack.md` for the technical details of each Da Vinci IG. The CMS rule references specific Da Vinci IG versions; verify which version is mandated for each provision.

> Verify against the current rule text — which Da Vinci IG versions are normative — before committing to a specific profile version.

---

## Operational Realities

- Implementing -0057-F is a multi-team, multi-quarter effort for any impacted payer — affects clinical, operations, IT, and provider-relations.
- Provider readiness lags — many providers do not have CDS Hooks consumers or SMART on FHIR app launchers ready, so legacy channels (fax / 278 / portal) will continue to handle a large share of volume even after API go-live.
- State-level rules may parallel or exceed CMS-0057-F (state PA reform laws). Verify state-specific obligations alongside federal.

> Verify state-specific PA reform laws (e.g., gold-carding statutes, decision-timing laws) against the applicable state insurance department / Medicaid agency.
