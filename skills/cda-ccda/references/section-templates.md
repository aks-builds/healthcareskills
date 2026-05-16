# C-CDA Section and Entry Templates

Reference for the most common C-CDA 2.1 section-level and entry-level templates, with required entry cardinality and value-set bindings. **Always confirm OID extensions, cardinality, and value-set URLs against the current C-CDA IG release and Companion Guide** — names and bindings shift across versions.

## Contents

- Conventions
- Allergies and Intolerances
- Medications
- Problem List
- Procedures
- Results (Labs)
- Vital Signs
- Encounters
- Plan of Treatment
- Immunizations
- Social History
- Functional Status
- Health Concerns
- Goals
- Assessment / Assessment and Plan
- Reason for Referral
- Hospital Course
- Entry Cardinality Notation
- Common Coding Bindings
- Authoring Tips

---

## Conventions

- **Section template** identifies a section of the document body (e.g., Allergies).
- **Entry template** identifies a structured `<entry>` inside the section.
- Most sections have an **Entries Required** variant and an **Entries Optional** variant — the IG distinguishes them by different templateId extensions and/or OIDs.
- **Cardinality** is given as `[min..max]` where `*` is unbounded.
- All OIDs and template extensions below are illustrative — verify against the IG.

---

## Allergies and Intolerances

- **Section LOINC code**: `48765-2` Allergies and adverse reactions Document
- **Section template (Entries Required)**: `2.16.840.1.113883.10.20.22.2.6.1`
- **Section template (Entries Optional)**: `2.16.840.1.113883.10.20.22.2.6`
- **Cardinality**: Required in CCD

### Entry Templates

| Template | Cardinality in section |
|----------|------------------------|
| Allergy Concern Act | `[1..*]` in Entries Required |
| Allergy - Intolerance Observation | `[1..*]` nested in each Allergy Concern Act |
| Reaction Observation | `[0..*]` nested in each Allergy - Intolerance Observation |
| Severity Observation | `[0..1]` nested |
| Allergy Status Observation | `[0..1]` nested |

### Bindings

- Allergy / Intolerance type: SNOMED CT value set
- Allergen substance: RxNorm (for drug allergens), SNOMED CT or UNII (for substances), or NDF-RT class codes per the IG-bound value set
- Reaction: SNOMED CT
- Severity: SNOMED CT severity value set
- Status: clinical-status value set

---

## Medications

- **Section LOINC code**: `10160-0` History of Medication use Narrative
- **Section template (Entries Required)**: `2.16.840.1.113883.10.20.22.2.1.1`
- **Cardinality**: Required in CCD

### Entry Templates

| Template | Cardinality |
|----------|-------------|
| Medication Activity | `[1..*]` |
| Medication Information | `[1..1]` nested in each Medication Activity |
| Indication | `[0..*]` nested |
| Instructions | `[0..*]` nested |
| Medication Supply Order | `[0..*]` nested |
| Drug Vehicle | `[0..*]` nested |

### Bindings

- Medication code: RxNorm (semantic clinical drug TTYs preferred; verify TTY restrictions per IG)
- Route of administration: SNOMED CT
- Dose form: SNOMED CT or NCI Thesaurus per IG
- Frequency / timing: PIVL_TS or EIVL_TS per IG patterns

---

## Problem List

- **Section LOINC code**: `11450-4` Problem list - Reported
- **Section template (Entries Required)**: `2.16.840.1.113883.10.20.22.2.5.1`
- **Cardinality**: Required in CCD

### Entry Templates

| Template | Cardinality |
|----------|-------------|
| Problem Concern Act | `[1..*]` |
| Problem Observation | `[1..*]` nested in each Problem Concern Act |
| Problem Status (Observation) | `[0..1]` nested |
| Health Status Observation | `[0..1]` nested |

### Bindings

- Problem code: SNOMED CT (US Core / CCD typical)
- Problem type: SNOMED CT problem-type value set
- Clinical status: clinical-status-code value set
- Verification status: verification-status value set

---

## Procedures

- **Section LOINC code**: `47519-4` History of Procedures Document
- **Section template (Entries Required)**: `2.16.840.1.113883.10.20.22.2.7.1`
- **Cardinality**: Required in CCD (Procedures section was elevated in later C-CDA 2.1 errata; verify)

### Entry Templates

| Template | Cardinality |
|----------|-------------|
| Procedure Activity Procedure | `[0..*]` |
| Procedure Activity Observation | `[0..*]` |
| Procedure Activity Act | `[0..*]` |

### Bindings

- Procedure code: SNOMED CT, CPT, HCPCS, or ICD-10-PCS depending on care setting and IG binding
- Body site: SNOMED CT
- Status: completed / aborted / cancelled (Act-Status value set)

---

## Results (Labs)

- **Section LOINC code**: `30954-2` Relevant diagnostic tests/laboratory data Narrative
- **Section template (Entries Required)**: `2.16.840.1.113883.10.20.22.2.3.1`
- **Cardinality**: Required in CCD

### Entry Templates

| Template | Cardinality |
|----------|-------------|
| Result Organizer | `[1..*]` |
| Result Observation | `[1..*]` nested in each Result Organizer |

### Bindings

- Result test code: LOINC
- Result value: PQ (Physical Quantity with UCUM units) for numerics; CD/CWE (SNOMED CT) for codified values
- Interpretation: HL7 ObservationInterpretation
- Reference range: low/high PQ values

---

## Vital Signs

- **Section LOINC code**: `8716-3` Vital signs
- **Section template (Entries Required)**: `2.16.840.1.113883.10.20.22.2.4.1`
- **Cardinality**: Required in CCD

### Entry Templates

| Template | Cardinality |
|----------|-------------|
| Vital Signs Organizer | `[1..*]` |
| Vital Sign Observation | `[1..*]` nested in each Vital Signs Organizer |

### Bindings

- Vital sign code: LOINC (specific vital signs value set)
- Value: PQ with UCUM units

Typical observations grouped: heart rate, respiratory rate, body temperature, systolic BP, diastolic BP, oxygen saturation, body height, body weight, BMI, head circumference, pulse oximetry.

---

## Encounters

- **Section LOINC code**: `46240-8` History of encounters
- **Section template (Entries Required)**: `2.16.840.1.113883.10.20.22.2.22.1`
- **Cardinality**: Recommended (Required if Encounters data is being conveyed)

### Entry Templates

| Template | Cardinality |
|----------|-------------|
| Encounter Activity | `[1..*]` |
| Encounter Diagnosis | `[0..*]` nested |
| Service Delivery Location | `[0..1]` nested |
| Indication | `[0..*]` nested |

### Bindings

- Encounter type: CPT E/M, SNOMED CT, or HL7 v3 ActEncounterCode per IG
- Encounter diagnosis: ICD-10-CM
- Service location type: SNOMED CT or HealthcareServiceLocation value set

---

## Plan of Treatment

- **Section LOINC code**: `18776-5` Plan of treatment
- **Section template**: `2.16.840.1.113883.10.20.22.2.10`
- **Cardinality**: Recommended in CCD, Required in some doc types

### Entry Templates

| Template | Cardinality |
|----------|-------------|
| Planned Observation | `[0..*]` |
| Planned Procedure | `[0..*]` |
| Planned Medication Activity | `[0..*]` |
| Planned Act | `[0..*]` |
| Planned Encounter | `[0..*]` |
| Planned Supply | `[0..*]` |
| Planned Immunization Activity | `[0..*]` |

---

## Immunizations

- **Section LOINC code**: `11369-6` History of Immunization Narrative
- **Section template (Entries Required)**: `2.16.840.1.113883.10.20.22.2.2.1`
- **Cardinality**: Recommended in CCD

### Entry Templates

| Template | Cardinality |
|----------|-------------|
| Immunization Activity | `[1..*]` |
| Immunization Refusal Reason | `[0..1]` nested |
| Immunization Medication Information | `[1..1]` nested |

### Bindings

- Vaccine code: CVX
- Manufacturer: MVX
- Refusal reason: NIP refusal reason value set (verify)

---

## Social History

- **Section LOINC code**: `29762-2` Social history Narrative
- **Section template**: `2.16.840.1.113883.10.20.22.2.17`
- **Cardinality**: Recommended in CCD

### Entry Templates

| Template | Cardinality |
|----------|-------------|
| Social History Observation | `[0..*]` |
| Smoking Status — Meaningful Use | `[0..1]` recommended |
| Tobacco Use | `[0..*]` |
| Birth Sex Observation | `[0..1]` |
| Pregnancy Observation | `[0..*]` |
| SDOH observations (housing, food insecurity, transportation, etc.) | `[0..*]` per the SDOH value set bindings |

### Bindings

- Smoking status: SNOMED CT (Meaningful Use smoking status value set)
- SDOH: LOINC + SNOMED CT per SDOH-related templates

---

## Functional Status

- **Section LOINC code**: `47420-5` Functional status assessment Narrative
- **Section template**: `2.16.840.1.113883.10.20.22.2.14`
- **Cardinality**: Recommended

### Entry Templates

| Template | Cardinality |
|----------|-------------|
| Functional Status Observation | `[0..*]` |
| Functional Status Result Observation | `[0..*]` |
| Functional Status Problem Observation | `[0..*]` |
| Caregiver Characteristics | `[0..*]` |

---

## Health Concerns

- **Section LOINC code**: `75310-3` Health concerns Document
- **Section template**: `2.16.840.1.113883.10.20.22.2.58`
- **Cardinality**: Recommended (Required in Care Plan)

### Entry Templates

| Template | Cardinality |
|----------|-------------|
| Health Concern Act | `[1..*]` |

---

## Goals

- **Section LOINC code**: `61146-7` Goals
- **Section template**: `2.16.840.1.113883.10.20.22.2.60`
- **Cardinality**: Recommended (Required in Care Plan)

### Entry Templates

| Template | Cardinality |
|----------|-------------|
| Goal Observation | `[1..*]` |

---

## Assessment / Assessment and Plan

- **Assessment Section LOINC**: `51848-0`
- **Assessment Section template**: `2.16.840.1.113883.10.20.22.2.8`
- **Assessment and Plan Section LOINC**: `51847-2`
- **Assessment and Plan Section template**: `2.16.840.1.113883.10.20.22.2.9`
- **Cardinality**: Required in Progress Note and Consultation Note; either Assessment alone or combined Assessment and Plan
- **Notes**: typically narrative-only; Plan entries (if structured) live in Plan of Treatment.

---

## Reason for Referral

- **Section LOINC code**: `42349-1` Reason for referral
- **Section template**: `2.16.840.1.113883.10.20.22.2.42`
- **Cardinality**: Required in Referral Note and Transfer Summary

---

## Hospital Course

- **Section LOINC code**: `8648-8` Hospital Course
- **Section template**: `1.3.6.1.4.1.19376.1.5.3.1.3.5`
- **Cardinality**: Required in Discharge Summary

---

## Entry Cardinality Notation

In templates the cardinality is paired with conformance verbs:

- **SHALL contain** `[1..1]` — exactly one
- **SHALL contain** `[1..*]` — at least one
- **SHOULD contain** `[1..1]` — best practice
- **MAY contain** `[0..1]` — optional
- **SHALL contain** `[0..1]` — at most one, but allowed null

`nullFlavor` (`NI`, `UNK`, `NAVU`, etc.) is permitted only where the IG explicitly allows it — not as a workaround for missing data on required elements.

---

## Common Coding Bindings

| Data | Typical binding |
|------|-----------------|
| Problem code (problem list) | SNOMED CT |
| Encounter diagnosis | ICD-10-CM |
| Lab result test | LOINC |
| Lab result value (numeric) | UCUM units |
| Lab result value (codified) | SNOMED CT |
| Vital sign code | LOINC |
| Vital sign value | UCUM |
| Medication | RxNorm |
| Allergy substance (drug) | RxNorm |
| Allergy substance (non-drug) | SNOMED CT or UNII |
| Immunization | CVX |
| Manufacturer | MVX |
| Procedure | SNOMED CT, CPT, HCPCS, or ICD-10-PCS |
| Document type | LOINC |
| Smoking status | SNOMED CT (MU value set) |
| Race / Ethnicity | OMB-derived value sets |

---

## Authoring Tips

- Build the narrative first; then build entries that mirror it. Cross-reference with `<reference value="#id"/>` and matching `ID` attributes in narrative.
- Use Entries Required section templates whenever you have structured data. Entries Optional is a last resort.
- Don't ship sections with `nullFlavor` if you actually have data — the C-CDA Scorecard penalizes this.
- Validate against the C-CDA Schematron, then the NIST validator, then the Scorecard. Each catches different errors.
- Test round-trip rendering: generate the C-CDA, render with the stylesheet, confirm narrative + entries agree.
- For USCDI compliance, walk the data classes and confirm each maps to either a structured entry or, at minimum, a narrative element.
