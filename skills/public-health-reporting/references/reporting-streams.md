# Public Health Reporting Streams

Reference for the major public health reporting streams that flow from clinical systems (EHRs, LIS, immunization modules, ADT feeds) to public health authorities. Always verify standard versions, value sets, and gateway endpoints against current ONC, CDC, APHL, and state public health agency guidance before implementation.

## Contents

- Electronic Laboratory Reporting (ELR)
- Electronic Case Reporting (eCR)
- Immunization Information Systems (IIS)
- Syndromic Surveillance (NSSP / BioSense)
- Cancer Reporting
- Vital Records (Birth and Death)
- Cross-cutting Notes

---

## Electronic Laboratory Reporting (ELR)

**Purpose**: Move reportable laboratory test results from a laboratory information system (LIS) or EHR-embedded lab module to public health.

**Primary standards / format**:
- HL7 v2.5.1 ORU^R01 constrained by the CDC ELR 2.5.1 Implementation Guide.
- HL7 v2.5.1 LRI (Lab Results Interface) IG overlaps for the lab-to-EHR leg.
- FHIR US Public Health Lab Reporting IG is the emerging path (verify current published version on HL7).

**Target authority**: State or local public health agency, with onward flow to CDC programs (e.g., NNDSS).

**Key data elements** (typical, verify per receiving IG):
- LOINC observation identifier (OBX-3) with UCUM units for numeric results.
- SNOMED CT or local result codes (OBX-5).
- Specimen segment (SPM) with collection date/time and source.
- Patient demographics with race/ethnicity (CDC code sets) and county FIPS.
- Provider NPI and ordering facility CLIA number.

**Transport options**: MLLP over VPN or TLS to the state gateway; sFTP batch; APHL AIMS Platform via secure web services or Direct.

**Trigger codes**: Pull from the current Reportable Conditions Trigger Codes (RCTC) value sets on VSAC rather than maintaining local copies. Verify against current RCTC publication date.

---

## Electronic Case Reporting (eCR)

**Purpose**: Send a structured case report from the EHR to public health automatically when criteria are met for a reportable condition.

**Primary standards / format**:
- eICR (electronic Initial Case Report) — originally HL7 CDA R2; the FHIR-based US Public Health eCR IG (eCR FHIR R4) is the current direction. Both remain in use depending on EHR version. Verify current published versions on HL7.
- Reportability Response (RR) — returned to the EHR indicating reportability per jurisdiction and any supplemental information requested.

**Target authority**: State, local, territorial, and tribal public health agencies, routed via the APHL AIMS Platform and evaluated by RCKMS.

**Trigger**: Encounter, problem, lab order/result, medication order, or other clinical event matching a current RCTC value set.

**Key participants**: EHR (trigger detection and eICR generation), APHL AIMS Platform (routing hub), RCKMS (jurisdiction-aware reportability rules engine maintained by CSTE and APHL), jurisdiction PHA (consumer).

**ONC criterion**: §170.315(f)(5) Transmission to public health agencies — electronic case reporting. Verify current criterion text on the ONC Certified Health IT Product List (CHPL).

---

## Immunization Information Systems (IIS)

**Purpose**: Submit and query immunization records against state and territorial registries.

**Primary standards / format**:
- HL7 v2.5.1 VXU^V04 for vaccination updates per the CDC IIS HL7 v2.5.1 Implementation Guide (verify current release).
- QBP^Q11 / RSP^K11 for query and forecast.
- FHIR US Core Immunization profile for FHIR-based exchange.

**Code systems**: CVX (vaccines administered) and MVX (manufacturers), both CDC-maintained. Look up current values from CDC IIS Standards before each release.

**Target authority**: State or territorial IIS; rolls up to CDC where applicable.

**Reporting requirements**: State law typically requires reporting all childhood vaccines; many states extend to adolescents and adults. Bidirectional flow (submit + query for history and forecast) is required for Promoting Interoperability credit.

**ONC criterion**: §170.315(f)(1) Transmission to immunization registries.

---

## Syndromic Surveillance (NSSP / BioSense)

**Purpose**: Near-real-time emergency department, urgent care, and increasingly inpatient registration and discharge messages used to detect outbreaks and aberrations.

**Primary standards / format**:
- HL7 v2.5.1 ADT^A01 / A03 / A04 / A08 following the PHIN Messaging Guide for Syndromic Surveillance (CDC). Verify current PHIN guide version.

**Target authority**: NSSP BioSense Platform via the state public health authority.

**Key data elements**: Chief complaint, triage notes, diagnoses (DG1), and disposition flow primarily in PV1, DG1, and OBX segments.

**Operational notes**:
- Data is submitted continuously (often every 30-60 minutes). Pipelines must be low-latency and tolerant of idempotent retransmission.
- Patient identifiers are often deidentified or pseudonymized for analysis; check the state data use agreement for what may be sent.

**ONC criterion**: §170.315(f)(2) Transmission to public health agencies — syndromic surveillance.

---

## Cancer Reporting

**Purpose**: Report reportable malignant and certain in situ neoplasms to the state central cancer registry.

**Primary standards / format**:
- NAACCR XML (current; replacing legacy fixed-width text record). Verify current NAACCR version.
- HL7 CDA-based cancer reporting IG (verify current version on HL7).
- mCODE (Minimal Common Oncology Data Elements, FHIR) is the emerging FHIR direction. Verify current published version on HL7.

**Target authority**: State central cancer registry, with onward flow to NPCR (CDC) and/or SEER (NCI).

**Key data elements** (typical, verify per state and NAACCR data dictionary): Diagnosis, primary site, histology (ICD-O-3), stage, treatments, and outcomes.

Hospital cancer programs accredited by the Commission on Cancer have additional reporting obligations beyond state law.

**ONC criterion**: §170.315(f)(4) Transmission to cancer registries.

---

## Vital Records (Birth and Death)

**Purpose**: Register live births and deaths with the state vital records office; rolled up nationally by NCHS.

**Primary standards / format**:
- EBR (Electronic Birth Registration) and EDR/EDRS (Electronic Death Registration System) are state-operated.
- NCHS coordinates the national rollup via the SAVER and STEVE networks. Verify current network names and onboarding contacts with NCHS.
- FHIR Vital Records Death Reporting IG (VRDR) and FHIR Vital Records Birth Reporting IG (BFDR) are the modernization path. Verify current published versions on HL7.

**Target authority**: State vital records office, then NCHS.

**Parties**: Birth registration typically involves the hospital birth clerk and the state vital records office. Death registration often involves multiple parties — certifier (clinician), funeral director, medical examiner/coroner, registrar.

---

## STD / HIV / Confidential Conditions

State law typically requires reporting, but with enhanced confidentiality beyond standard HIPAA. Verify your jurisdiction's statute and the state public health department's written guidance — do not infer carve-outs from federal law alone.

- HIV reporting often has separate flows and restricted recipient lists.
- Some states have partner services (contact tracing) reporting overlays.
- 42 CFR Part 2 (SUD) records add another layer when substance use is involved.

---

## Cross-cutting Notes

- **Triggering**: Drive reportability detection from the current RCTC value sets on VSAC and from RCKMS, not from per-state hard-coded condition lists in the EHR.
- **Onboarding**: Multi-step (registration, message validation, pre-production, validation, production). Plan months, not weeks, per receiving authority.
- **Test data**: Use synthetic data only. Many authorities supply test patient and test result fixtures.
- **Acknowledgments**: Persist and monitor. Conventions vary by stream and authority.
- **Audit**: Every outbound and inbound message should produce an audit record.

Always verify standard versions, IG versions, value set publications, ONC criterion text, and gateway endpoints against current authoritative sources before implementation.
