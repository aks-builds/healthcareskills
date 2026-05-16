# CMS Patient Access API (CMS-9115-F and Related Rules)

The CMS Interoperability and Patient Access Final Rule (**CMS-9115-F**), published in 2020 under the 21st Century Cures Act, requires certain CMS-regulated payers to expose patient data via a standards-based FHIR API. The rule has been extended by subsequent CMS rulemaking, most notably the **CMS Advancing Interoperability and Improving Prior Authorization Processes Final Rule (CMS-0057-F)** finalized in 2024, which adds API requirements for providers, prior authorization, and provider-to-provider data exchange.

This reference summarizes the scope. **Always verify against the current CMS regulations and CMS technical specifications before implementation; CMS continues to issue clarifying guidance and the compliance dates have been adjusted multiple times.**

## Who must comply

CMS-9115-F applies to **payers** in CMS-regulated programs:

- Medicare Advantage (MA) organizations
- Medicaid Fee-for-Service and managed care organizations
- CHIP Fee-for-Service and managed care
- Qualified Health Plan (QHP) issuers on the federally-facilitated exchanges

It does **not directly** apply to:

- Commercial group / individual plans outside the exchanges (though many follow voluntarily)
- Original Medicare (CMS itself is the implementer for Medicare beneficiaries via Blue Button 2.0)
- ERISA self-funded plans (different regulatory regime; some choose to comply voluntarily)

The 2024 CMS-0057-F extends API requirements (notably **Prior Authorization API** and **Provider Access API**) and adjusts timing. Verify current applicability.

## What must be exposed (Patient Access API scope)

The Patient Access API must expose, on request by the patient (or their authorized app), the following data classes:

### Claims and encounters data

- Adjudicated claims and encounter data, including cost-sharing information
- Provider remittance data and patient cost-sharing
- A history typically extending **5 years back** from the request date (verify current retention requirement)

### Clinical data

- **USCDI** clinical data classes the payer maintains. Importantly, payers historically did not maintain extensive clinical data — they exchange what they have, which is often limited to claims-derived data and any clinical exchange they participate in.
- As payer-clinical data exchange (via TEFCA, HIE participation, or direct provider data) increases, more clinical content will be available.

### Formulary / coverage information

- For MA / Part D and Medicaid where applicable, formulary information must be exposed.

### Pending and active prior authorization (added by CMS-0057-F)

- Information on prior authorization status — pending, approved, denied, with reasons — for items / services the payer processes.

## Technical standards

The Patient Access API is built on:

- **HL7 FHIR R4** — the data shapes
- **HL7 US Core** — the FHIR profiles aligned with USCDI
- **CARIN Blue Button Implementation Guide** — claims and EOB representation (`ExplanationOfBenefit` profile family)
- **DaVinci PDex Implementation Guide** — broader payer data exchange
- **SMART App Launch Framework** — OAuth 2.0-based app authorization, with patient-launch context

Authentication and authorization typically follow:

- **OpenID Connect** on top of OAuth 2.0
- **Identity proofing at IAL2** (see `identity-proofing-levels.md`)
- **Refresh tokens** for ongoing app access
- A **registration process** for third-party apps that meets the rule's no-special-effort standard — payers can have an app-registration step, but cannot impose arbitrary barriers

## Provider Access API (CMS-0057-F)

Adds a payer-to-provider API that lets providers retrieve patient data from a payer for their attributed patients. This is for **provider** consumption, not patient consumption.

## Payer-to-Payer API (CMS-0057-F)

Adds a payer-to-payer data exchange API so when a patient moves between payers, the new payer can request the patient's data from the prior payer.

## Prior Authorization API (CMS-0057-F)

Operational API for prior authorization — submit requests, check status, retrieve responses. Built on the **DaVinci PAS (Prior Auth Support)** and related implementation guides.

## Provider rule (the ONC Information Blocking side)

Separate from CMS-9115, the **ONC Information Blocking** rule under the 21st Century Cures Act applies to **providers**, **health IT developers**, and **HINs/HIEs**. It requires that EHI (Electronic Health Information) not be interfered with — there are 8 enumerated exceptions, and the default behavior is **share**. Most provider-side patient portal data exposure operates under this rule rather than CMS-9115.

## Information Blocking exceptions (the 8 categories)

1. **Preventing Harm** — narrow; harm must be likely and substantial
2. **Privacy** — must follow a privacy law (HIPAA + state); often used for adolescent confidentiality, SUD records, abuse documentation
3. **Security** — narrow technical security concerns
4. **Infeasibility** — extraordinary circumstances; not "inconvenient"
5. **Health IT Performance** — temporary system unavailability
6. **Content & Manner** — alternate format with reasonable terms
7. **Fees** — reasonable, cost-based, non-discriminatory
8. **Licensing** — reasonable, non-discriminatory licensing terms

A provider blocking content must be able to point to a documented exception. The default is share.

## USCDI versions

USCDI (US Core Data for Interoperability) is the data class set. Each version adds more classes.

- USCDI v1, v2, v3, v4, v5 — verify the version applicable to your compliance year.
- Each USCDI version maps to a corresponding US Core IG version with FHIR profiles for each data class.

## Engineering implications

### Payer portal building blocks

- FHIR R4 server profiled to US Core / CARIN Blue Button / DaVinci PDex
- SMART on FHIR app registration and authorization server
- IAL2 identity proofing for member access
- AAL2 authentication for the API
- Patient-search-and-match against the member master
- Cost-sharing data assembly (often the hardest non-FHIR-shape problem)
- Claims `ExplanationOfBenefit` mapping (CARIN Blue Button)
- Audit log of every API access as `AuditEvent`

### Provider portal building blocks

- FHIR R4 server profiled to US Core for the matching USCDI version
- SMART on FHIR app launch (patient-launch from portal, provider-launch from EHR)
- Per-resource authorization scopes
- ONC Information Blocking analysis at every release point — default share, exception documented
- Adolescent confidentiality / sensitivity filter (see SKILL.md "Adolescent Confidentiality")
- 42 CFR Part 2 segmentation if SUD records are in scope
- Audit log of every release decision

### Third-party app handling

- Patients may authorize **third-party apps** (Apple Health, CommonHealth, consumer portals) to receive data. The rule does not allow the payer / provider to add barriers based on the app's privacy practices — though they may provide patient-facing transparency about the app.
- The portal's authorization server should support standard SMART scopes and the standard app-registration flow.

## Verify-current sources

- CMS-9115-F regulation text and CMS technical guidance
- CMS-0057-F regulation text and CMS technical guidance
- ONC Information Blocking rule (45 CFR Part 171)
- HL7 US Core Implementation Guide (current version)
- HL7 CARIN Blue Button Implementation Guide
- HL7 DaVinci Implementation Guides (PDex, PAS, DTR, CRD)
- HL7 SMART App Launch Framework

Implementation timing has shifted multiple times. Always check current CMS-announced compliance dates before locking your roadmap.
