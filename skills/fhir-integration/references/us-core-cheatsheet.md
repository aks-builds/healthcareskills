# US Core Cheat Sheet

Quick reference for US Core profiles. **Always verify against the current US Core IG version in scope** (US Core 6.x / 7.x / 8.x have meaningful differences in must-support, value sets, and required elements). Names below are generic — exact profile URLs and version tags must come from the published IG.

## Contents

- How to read this sheet
- Patient and Demographics
- Conditions, Allergies, and Health Concerns
- Medications
- Encounters and Care Team
- Observations (Labs, Vitals, Social History, SDOH)
- Procedures, Diagnostic Reports, Imaging
- Immunizations
- Documents and Clinical Notes
- Provenance and Goal
- Payer and Coverage
- Search-Parameter Must-Supports
- Key Bindings at a Glance
- Common Validation Issues

---

## How to Read This Sheet

Each profile entry lists:

- **Resource** — base FHIR resource the profile constrains
- **Anchors** — must-support elements you nearly always need
- **Bindings** — required or extensible value sets (system + concept)
- **Notes** — gotchas

All version numbers and must-support specifics must be confirmed against the IG package you are using. US Core changes every year.

---

## Patient and Demographics

### US Core Patient

- **Resource**: Patient
- **Anchors**: identifier (at least one), name.family or name.given, gender, birthDate
- **Race / Ethnicity / Birth Sex / Gender Identity** — carried by US Core extensions (each has its own profile and value set)
- **Notes**: identifier slicing typically distinguishes MRN, SSN (rare), and other types

### US Core Practitioner / PractitionerRole

- **Resource**: Practitioner, PractitionerRole
- **Anchors**: identifier (NPI most often), name
- **Notes**: PractitionerRole ties a Practitioner to an Organization and Location

### US Core RelatedPerson

- **Resource**: RelatedPerson
- **Anchors**: patient, relationship, name

### US Core Organization

- **Resource**: Organization
- **Anchors**: identifier (NPI when available), active, name, address

### US Core Location

- **Resource**: Location
- **Anchors**: status, name, address

---

## Conditions, Allergies, and Health Concerns

### US Core Condition (problem-list-item / encounter-diagnosis / health-concern)

- **Resource**: Condition
- **Anchors**: category (problem-list-item / encounter-diagnosis / health-concern), code, subject, clinicalStatus, verificationStatus
- **Bindings**: code uses SNOMED CT for problem-list-item (US Core); ICD-10-CM for encounter-diagnosis
- **Notes**: must-support varies per category slice; recordedDate and onset[x] are usually must-support

### US Core AllergyIntolerance

- **Resource**: AllergyIntolerance
- **Anchors**: clinicalStatus, verificationStatus, patient, code
- **Bindings**: code uses SNOMED CT (for substance/condition) or RxNorm (for drug allergens)

---

## Medications

### US Core MedicationRequest

- **Resource**: MedicationRequest
- **Anchors**: status, intent, medication[x] (CodeableConcept or Reference), subject, authoredOn, requester
- **Bindings**: RxNorm — exact TTYs vary by US Core version (typically SCD/SBD/GPCK/BPCK; verify)
- **Notes**: when medication is a Reference, the referenced Medication resource also has a US Core profile

### US Core MedicationStatement / MedicationDispense

- Some US Core versions add MedicationDispense and constrain MedicationStatement. Confirm against the specific version.

---

## Encounters and Care Team

### US Core Encounter

- **Resource**: Encounter
- **Anchors**: status, class, type, subject, participant (with type and period), period, reasonCode or reasonReference, hospitalization (for inpatient), location
- **Bindings**: class uses v3 ActEncounterCode; type uses CPT or SNOMED CT

### US Core CareTeam

- **Resource**: CareTeam
- **Anchors**: status, subject, participant.member, participant.role

---

## Observations (Labs, Vitals, Social History, SDOH)

Multiple Observation profiles share a base shape but specialize:

### US Core Laboratory Result Observation

- **Anchors**: status, category (laboratory), code (LOINC), subject, effective[x], value[x] or dataAbsentReason
- **Bindings**: code = LOINC; valueQuantity.code = UCUM; valueCodeableConcept may use SNOMED CT

### Vital Signs Profiles

- US Core layers profile on top of base FHIR Vital Signs profiles (BP panel, heart rate, body temp, body weight, body height, BMI, respiratory rate, oxygen saturation, pulse oximetry, head circumference, etc.)
- BP uses LOINC `85354-9` panel with systolic / diastolic components

### Smoking Status

- LOINC `72166-2` (Tobacco smoking status) with SNOMED CT value bindings

### Pediatric / SDOH / Pulse Oximetry / Pregnancy-related profiles

- Added incrementally across US Core versions — verify which profiles exist in the version in scope

---

## Procedures, Diagnostic Reports, Imaging

### US Core Procedure

- **Anchors**: status, code, subject, performed[x]
- **Bindings**: code = SNOMED CT, CPT, HCPCS, or ICD-10-PCS depending on setting

### US Core DiagnosticReport (Lab) / DiagnosticReport (Note)

- US Core defines distinct profiles for lab reports vs. report notes — verify
- **Anchors**: status, category, code, subject, effective[x], issued, performer, result (for lab) or presentedForm (for note)

### US Core ImagingStudy / ServiceRequest

- ImagingStudy links to DICOM via identifier and endpoints
- ServiceRequest used for orders (lab, imaging, referrals)

---

## Immunizations

### US Core Immunization

- **Anchors**: status, vaccineCode, patient, occurrence[x], primarySource
- **Bindings**: vaccineCode = CVX

---

## Documents and Clinical Notes

### US Core DocumentReference

- **Anchors**: status, type (LOINC), category, subject, date, author, content.attachment
- **Bindings**: type = LOINC document codes (e.g., CCD = 34133-9)
- **Notes**: heavy use for C-CDA attachments and clinical notes

---

## Provenance and Goal

### US Core Provenance

- Tracks who produced or asserted a resource
- **Anchors**: target, recorded, agent.who, agent.type

### US Core Goal

- **Anchors**: lifecycleStatus, description, subject

---

## Payer and Coverage

Later US Core versions include Coverage and related payer-side profiles, aligning with CARIN BB. Verify against the IG.

---

## Search-Parameter Must-Supports

US Core specifies required search parameters per resource. Common examples (verify per version):

| Resource | Common required searches |
|----------|-------------------------|
| Patient | `_id`, `identifier`, `name`, `birthdate`, `gender` |
| Condition | `patient`, `category`, `clinical-status`, `code` |
| Observation | `patient`, `category`, `code`, `date` |
| MedicationRequest | `patient`, `status`, `authoredon`, `intent` |
| Encounter | `patient`, `date`, `class`, `identifier` |
| AllergyIntolerance | `patient`, `clinical-status` |
| Immunization | `patient`, `status`, `date` |
| Procedure | `patient`, `date`, `code` |
| DocumentReference | `patient`, `category`, `type`, `date`, `period`, `status` |

Combination searches (`patient+category`, `patient+code`, etc.) are typically must-support — confirm against the specific US Core capability statement.

---

## Key Bindings at a Glance

| Data | Required code system |
|------|---------------------|
| Problem-list condition | SNOMED CT |
| Encounter diagnosis | ICD-10-CM |
| Lab / vital / SDOH observation | LOINC (code), UCUM (units) |
| Medication | RxNorm |
| Allergen (drug) | RxNorm |
| Allergen (substance) | SNOMED CT |
| Immunization | CVX |
| Procedure (outpatient) | SNOMED CT or CPT |
| Procedure (inpatient) | ICD-10-PCS or CPT |
| Document type | LOINC |
| Race / Ethnicity | OMB-derived value sets |

All bindings have a strength (required vs. extensible vs. preferred) — confirm in the IG before designing for strict enforcement.

---

## Common Validation Issues

- Missing must-support identifier slice — Patient or Practitioner with no identifier fails US Core
- Wrong value-set source for problem-list (using ICD-10-CM where SNOMED CT is required)
- Encounter without class or type — both are typically must-support
- MedicationRequest with a free-text medication.code instead of an RxNorm coding
- Observation.valueQuantity without UCUM `system` and `code`
- Asserting `meta.profile = us-core-patient` but missing must-support elements; `$validate` will fail
- Pinning to an older US Core version without checking what your trading partner is on

When in doubt, run `$validate` against the FHIR Validator with the explicit US Core IG package pinned, and read the OperationOutcome carefully.
