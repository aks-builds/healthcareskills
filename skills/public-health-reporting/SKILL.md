---
name: public-health-reporting
description: When the user wants to design, build, or troubleshoot reporting from clinical systems to public health authorities. Use when the user mentions "public health reporting," "ELR," "Electronic Laboratory Reporting," "eCR," "electronic case reporting," "eICR," "Reportability Response," "RR," "immunization registry," "IIS," "VXU," "syndromic surveillance," "BioSense," "NSSP," "cancer reporting," "NAACCR," "NPCR," "SEER," "birth registry," "EBR," "death registry," "EDR," "EDRS," "reportable conditions," "RCKMS," "CSTE," "APHL AIMS," "NEDSS," "ONC g.10," "Promoting Interoperability public health," "state HIE submission," or "COVID case reporting." For the underlying HL7 v2 message mechanics, see hl7-v2. For FHIR resources used by these IGs, see fhir-integration. For terminology bindings (LOINC, SNOMED CT, ICD-10-CM, CVX), see terminology-services.
metadata:
  version: 1.0.0
---

# Public Health Reporting

You are an expert in public health reporting interfaces between clinical systems (EHRs, LIS, immunization modules) and public health authorities (state/local health departments, CDC programs, registries). Your goal is to help engineers and informaticists design, implement, and certify the reporting streams required by federal and state law without inventing requirements that don't apply to the specific program or jurisdiction.

## Initial Assessment

Check `.agents/healthcare-context.md` (fallback: `.claude/healthcare-context.md`) before answering. Focus on:

- **Organization type and HIPAA role** — hospital, LIS, FQHC, payer, state HIE, vendor.
- **Jurisdictions** — each US state defines its own reportable conditions list and submission gateway; territorial and tribal jurisdictions also have their own.
- **Source systems** — EHR vendor, LIS vendor, immunization module, registration/ADT feed.
- **Transport posture** — direct point-to-point, state HIE, APHL AIMS Platform, NEDSS gateway, Direct secure messaging.
- **Certification / incentive program participation** — ONC Health IT Certification (g.10), HHS Promoting Interoperability (formerly Meaningful Use), MIPS Promoting Interoperability category.

If the file is missing, ask only what is needed for the current stream (which authority, which condition, which transport) and offer to save it.

---

## Streams at a Glance

| Stream | Standard(s) | Primary trigger | Authority |
|--------|-------------|-----------------|-----------|
| Electronic Laboratory Reporting (ELR) | HL7 v2.5.1 ORU^R01 (LRI/ELR profiles), FHIR US Public Health Lab Reporting IG | Reportable test result from LIS | State/local public health, CDC |
| Electronic Case Reporting (eCR) | FHIR US Public Health eCR IG (eICR + Reportability Response) | Encounter / problem / order matches RCKMS rules | State/local public health via APHL AIMS |
| Immunization (IIS) | HL7 v2.5.1 VXU^V04 + QBP^Q11 / RSP^K11, FHIR US Core Immunization | Vaccine administered or queried | State/territorial IIS |
| Syndromic surveillance | HL7 v2.5.1 ADT^A04 / A08 (PHIN Messaging Guide for Syndromic Surveillance) | ED/UC/inpatient registration & updates | NSSP BioSense Platform via state |
| Cancer | NAACCR XML / HL7 CDA cancer reporting, FHIR mCODE (emerging) | Reportable neoplasm diagnosed/treated | State central cancer registry, NPCR, SEER |
| Vital records (birth) | EBR via SAVER / NCHS interfaces, FHIR Vital Records Birth Reporting IG (emerging) | Live birth | State vital records office, NCHS |
| Vital records (death) | EDRS (state systems), FHIR Vital Records Death Reporting IG | Death | State vital records office, NCHS |
| STD/HIV/TB | State-specific; often ELR + supplemental case report | Reportable communicable disease | State/local public health, CDC |
| Healthcare Surveys / Reportable Events | Program-specific | Outbreak, AFM, novel pathogen | CDC NHSN, state HAI programs |

---

## Reportable Conditions and Decision Support (RCKMS)

The **Reportable Conditions Knowledge Management System (RCKMS)** is the authoritative, jurisdiction-aware rules engine maintained by CSTE/APHL. It encodes "is this case reportable to this jurisdiction?" logic so that EHR triggers don't need to be hard-coded per state.

- The condition list is maintained by **CSTE** (Council of State and Territorial Epidemiologists).
- Trigger codes (LOINC for tests, SNOMED CT / ICD-10-CM for diagnoses, RxNorm for treatments) are published as the **Reportable Conditions Trigger Codes (RCTC)** value sets on VSAC.
- Verify the current trigger code set against VSAC (https://vsac.nlm.nih.gov) before each release — the value sets are updated regularly.

EHRs subscribe to RCTC for triggering and then send the eICR to AIMS, which routes through RCKMS to determine reportability per jurisdiction and returns a Reportability Response.

---

## Electronic Laboratory Reporting (ELR)

ELR moves reportable lab results from the LIS (or EHR with LIS module) to public health.

### Standards

- **HL7 v2.5.1 ORU^R01** is the de facto national standard, constrained by the **CDC ELR 2.5.1 Implementation Guide** (HL7 v2.5.1 Lab Results to Public Health, Release 1, Errata).
- The **HL7 v2.5.1 LRI** (Lab Results Interface) IG governs lab-to-EHR; ELR is a public-health-facing constraint that overlaps significantly.
- **FHIR US Public Health Lab Reporting IG** (LDH / LRH) is the emerging path; verify the current version on HL7.

### Required elements (typical)

- OBR-31 reason for study / ordered test indication, OBX-3 LOINC observation identifier, OBX-5 value (with UCUM units for numerics), OBX-6 reference range/units, OBX-11 result status.
- SPM segment for specimen (collection date/time, source, body site).
- PID with full demographics including race/ethnicity (CDC code sets), language, address with county FIPS, and where allowed, SSN — minimum-necessary still applies.
- Provider (OBR-16) and ordering facility (ORC-21) identifiers including NPI and CLIA number.

Codes shown in any example are illustrative — verify LOINC, SNOMED CT, and SNOMED-CT-based result codes against the current RCTC value sets and the receiving jurisdiction's onboarding guide.

### Transport

- MLLP over VPN or TLS to the state public health gateway, or
- File drop (sFTP) of batched HL7 v2 messages, or
- APHL AIMS Platform via secure web services / Direct.

---

## Electronic Case Reporting (eCR)

eCR sends a structured case report from the EHR to public health automatically when criteria are met.

### Architecture

```
EHR ──(eICR FHIR Bundle or CDA)──> AIMS Platform ──(RCKMS)──> Jurisdiction PHA
  ^                                                                  │
  └────────────── Reportability Response (RR) ──────────────────────┘
```

- **eICR** = electronic Initial Case Report. Originally a HL7 CDA R2 document; the FHIR-based **US Public Health eCR IG** (eCR FHIR R4) is the current direction. Both are still in use depending on EHR version.
- **RR** = Reportability Response. Tells the EHR whether the case was reportable, to which jurisdiction(s), and what (if any) supplemental information is requested.
- The **APHL AIMS Platform** is the national routing hub.

### Triggering

The EHR uses the RCTC value sets to detect a trigger event (a problem, encounter diagnosis, lab order, medication order, or result that matches a condition). On match, the EHR generates an eICR and submits to AIMS.

COVID-19 expanded eCR adoption dramatically; HHS and CDC issued public health emergency guidance to accelerate onboarding, and many large EHR vendors now ship eCR as a built-in capability.

---

## Immunization Information Systems (IIS)

IIS are state/territorial registries that consolidate immunization records.

### Standards

- **HL7 v2.5.1 VXU^V04** for vaccination updates (CDC IIS HL7 v2.5.1 Release 1.5+ implementation guide).
- **QBP^Q11 / RSP^K11** for query and forecast.
- **FHIR US Core Immunization** profile for FHIR-based exchange (e.g., SMART on FHIR apps querying patient records).
- **CVX** (vaccine administered codes) and **MVX** (manufacturer codes) — CDC-maintained, look up current values from CDC IIS Standards.

### Reporting requirements

- State law typically requires reporting all childhood vaccines; many states extend to adolescents and adults.
- Forecasting/recommendation responses use the **CDC Immunization Calculation Engine (ICE)** logic.
- Bidirectional flow (submit + query for history and forecast) is required for Promoting Interoperability credit.

---

## Syndromic Surveillance (NSSP / BioSense)

Syndromic surveillance uses near-real-time emergency department, urgent care, and (increasingly) inpatient registration messages to detect outbreaks and aberrations.

### Standards

- **HL7 v2.5.1 ADT^A01 / A03 / A04 / A08** following the **PHIN Messaging Guide for Syndromic Surveillance** (CDC).
- Chief complaint, triage notes, diagnoses, and disposition flow in PV1, DG1, and OBX segments.
- Submission is to the **National Syndromic Surveillance Program (NSSP) BioSense Platform** via the state public health authority.

### Operational notes

- Data is submitted continuously (often every 30-60 minutes) — design pipelines for low-latency, idempotent retransmission.
- Patient identifiers are deidentified or pseudonymized for many analyses; check your state agreement for what may be sent.

---

## Cancer Reporting

- **NPCR** (National Program of Cancer Registries, CDC) funds most state central cancer registries.
- **SEER** (Surveillance, Epidemiology, and End Results, NCI) covers a subset of geographies and supports research.
- **NAACCR** (North American Association of Central Cancer Registries) publishes the data exchange standard — historically a fixed-width text record, now NAACCR XML.
- HL7 has published a CDA-based cancer reporting IG; **mCODE** (Minimal Common Oncology Data Elements, FHIR) is the emerging FHIR direction.
- Hospital cancer programs (Commission on Cancer accredited) have additional reporting obligations beyond state law.

Reportable items typically include the diagnosis, primary site, histology (ICD-O-3), stage, treatments, and outcomes for malignant and certain in situ neoplasms.

---

## Vital Records (Birth and Death)

- **EBR (Electronic Birth Registration)** and **EDR/EDRS (Electronic Death Registration System)** are state-operated.
- NCHS coordinates the national rollup via the **SAVER** / **STEVE** networks.
- The **FHIR Vital Records Death Reporting IG (VRDR)** and **FHIR Vital Records Birth Reporting IG (BFDR)** are the modernization path.
- Death reporting often involves multiple parties: certifier (clinician), funeral director, medical examiner/coroner, registrar.

---

## STD / HIV / Confidential Conditions

State law typically requires reporting, but with **enhanced confidentiality** beyond standard HIPAA:

- HIV is reported to public health but often with separate flows and restricted recipient lists.
- Some states have **partner services** (contact tracing) reporting overlays.
- 42 CFR Part 2 (SUD) records add another layer when substance use is involved.
- Verify your jurisdiction's statute and the state public health department's written guidance — do not infer carve-outs from federal law alone.

---

## Public Health Gateways and Transport

| Gateway | Operator | Role |
|---------|----------|------|
| APHL AIMS Platform | APHL | National hub for eCR, ELR, and many other streams |
| NEDSS Base System (NBS) | CDC | Surveillance system used by many states |
| State public health gateway | State PHA | Per-state ingress (e.g., MDPHnet, NY ECLRS, CalREDIE, FL Merlin) |
| State HIE | Regional | Acts as intermediary in some states |
| Direct secure messaging | DirectTrust HISPs | Used for low-volume eCR/ELR onboarding |

When recommending an integration, match the receiving authority's onboarding documentation — gateways differ in transport, authentication (mTLS, API key, OAuth), batch vs. real-time, and acknowledgment expectations.

---

## ONC Certification and HHS Incentive Programs

### ONC Health IT Certification (45 CFR Part 170)

The 2015 Edition + Cures Update certification criterion **§170.315(f)** covers public health transmission. Sub-criteria include:

- (f)(1) Transmission to immunization registries
- (f)(2) Transmission to public health agencies — syndromic surveillance
- (f)(3) Transmission to public health agencies — reportable laboratory tests and value/results
- (f)(4) Transmission to cancer registries
- (f)(5) Transmission to public health agencies — electronic case reporting
- (f)(6) Transmission to public health agencies — antimicrobial use and resistance reporting
- (f)(7) Transmission to public health agencies — health care surveys

The often-mentioned **g.10** criterion covers Standardized API for patient and population services (FHIR / SMART), which underpins eCR and other FHIR-based flows. Verify the current criterion text and Cures Update applicability on the ONC Certified Health IT Product List (CHPL).

### HHS Promoting Interoperability (Medicare / Medicaid / MIPS)

Eligible hospitals and clinicians earn PI credit by reporting to public health streams (immunization, syndromic, ELR, eCR, electronic reportable laboratory results). Required vs. optional measures and active engagement levels (pre-production, validation, production) change yearly — verify the current rule with CMS.

---

## Operational Concerns

- **Onboarding** to any public health authority is multi-step: registration, message validation, pre-production, production. Plan months, not weeks.
- **Test data** must be synthetic; many authorities supply test patient/test result fixtures.
- **Acknowledgments** vary: some streams require enhanced HL7 v2 ACK, some FHIR OperationOutcome, some custom. Persist and monitor.
- **Identity reconciliation** — public health rarely has an enterprise MPI; design for repeated submissions and duplicate handling.
- **Privacy** — HIPAA permits disclosure to public health authorities under 45 CFR §164.512(b), but minimum necessary, state law overlays, and 42 CFR Part 2 still apply. Verify the specific authority's legal basis before sending.
- **Audit** — every outbound and inbound message should produce an audit record (see audit-logging).

---

## Common Pitfalls

- Triggering on EHR problem lists alone without honoring RCTC value-set membership, missing reportable cases or generating false positives.
- Treating eICR and Reportability Response as fire-and-forget — the RR may request supplemental info or correct jurisdiction routing.
- Hard-coding state-specific routing in the EHR; AIMS + RCKMS exist precisely to centralize that logic.
- Sending PHI to syndromic surveillance beyond what the state agreement permits.
- Letting ELR backlog silently — public health depends on near-real-time signals during outbreaks.
- Skipping IIS query (only submitting), which forfeits forecasting benefit to clinicians.
- Assuming HIPAA pre-empts state confidentiality law for HIV/STD/SUD reporting — verify state statute.

---

## Task-Specific Questions

1. Which reporting stream is in scope — ELR, eCR, immunization (IIS), syndromic surveillance, cancer, vital records (birth/death), or a confidential-condition stream (HIV/STD/TB)?
2. Which state, local, territorial, or tribal public health agency is the receiving authority, and have you already identified their onboarding documentation?
3. Are you working in HL7 v2 (ORU^R01, VXU^V04, ADT) or FHIR (US Public Health IGs / Bulk Data), and is the EHR/LIS vendor's outbound capability already known?
4. Which gateway is the integration target — APHL AIMS Platform, NEDSS Base System, the state public health gateway directly, a state HIE, or Direct secure messaging?
5. What is the current automation level — manual reporting, batch sFTP, real-time MLLP, FHIR API, or a mix — and what is the desired end state?
6. Which incentive or certification obligations are in scope — ONC §170.315(f) certification criteria, HHS Promoting Interoperability for hospitals/clinicians, MIPS PI category — and what active-engagement level (pre-production, validation, production) are you targeting?
7. For triggering (eCR especially), are you already consuming the current RCTC value sets from VSAC, and is RCKMS configured for your jurisdictions?

---

## Related Skills

- **hl7-v2** — underlying ORU^R01, VXU^V04, ADT^A04/A08 mechanics
- **fhir-integration** — FHIR US Public Health IGs (eCR, Lab Reporting, Immunization)
- **terminology-services** — LOINC, SNOMED CT, ICD-10-CM, CVX, RxNorm bindings and VSAC value sets
- **ehr-integration** — EHR vendor configuration for eCR triggers and IIS submission
- **hipaa-compliance** — §164.512(b) public health disclosure, minimum necessary, state overlays
- **audit-logging** — outbound public health submission audit and acknowledgment tracking
- **clinical-decision-support** — RCTC trigger logic overlaps with CDS rule authoring
