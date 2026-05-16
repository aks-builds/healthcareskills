# SDOH Coding and Capture

Reference for capturing, coding, and exchanging social determinants of health (SDOH) and Health-Related Social Needs (HRSN) data in a population health platform. Verify current Gravity Project FHIR IG versions, USCDI data class versions, and CMS payment-year SDOH adjuster status against authoritative sources before implementing.

## Contents

- Terminology landscape
- ICD-10-CM Z55-Z65
- LOINC for screening instruments and answers
- SNOMED CT for SDOH conditions, goals, interventions
- Screening instruments
- Area-level deprivation indices
- Gravity Project FHIR IG
- USCDI SDOH data classes
- Operational use
- Verification expectations

---

## Terminology landscape

SDOH data cuts across multiple terminologies. A coherent platform uses each in its lane:

| Layer | Terminology | Role |
|-------|-------------|------|
| Billable diagnosis codes | ICD-10-CM Z55-Z65 | Claims-side flagging; coarse but billable |
| Screening instrument items and answers | LOINC | Question and answer codes for standardized instruments |
| Conditions, goals, interventions, problems | SNOMED CT | Clinical-side coding via Gravity Project IG |
| Service categories | SNOMED CT / local | Referral category coding |
| Resources / programs | Open Referral / Aunt Bertha taxonomies | Vendor-side resource catalogs |

Maintain crosswalks centrally; an HRSN screening response should map deterministically to a SNOMED CT problem and (where billable and clinically appropriate) an ICD-10-CM Z-code.

---

## ICD-10-CM Z55-Z65

The Z55-Z65 block — "Persons with potential health hazards related to socioeconomic and psychosocial circumstances" — covers (verify against current ICD-10-CM tabular):

- Z55 Problems related to education and literacy
- Z56 Problems related to employment and unemployment
- Z57 Occupational exposure to risk factors
- Z58 Problems related to physical environment (verify current scope)
- Z59 Problems related to housing and economic circumstances
- Z60 Problems related to social environment
- Z62 Problems related to upbringing
- Z63 Other problems related to primary support group, including family circumstances
- Z64 Problems related to certain psychosocial circumstances
- Z65 Problems related to other psychosocial circumstances

Operational caveats:
- **Coding rates are very low**. Studies show single-digit-percent Z-code capture rates even when screening identifies a much higher prevalence.
- **Reimbursement implications vary by payer**. Z-codes are billable but typically not primary diagnoses for payment.
- **CMS HCC inclusion is evolving**. Some Z-codes have been added to certain Medicare HCC categories; verify the current payment-year specification before relying on Z-codes for risk adjustment.

Z-codes alone underestimate HRSN prevalence dramatically. Pair with screening instruments for accurate population view.

---

## LOINC for screening instruments and answers

LOINC has codes for standardized screening instruments — both the panel (the instrument as a whole) and the individual questions and standardized answers. Examples (verify each LOINC code against the current LOINC release before implementing):

- **PRAPARE** (Protocol for Responding to and Assessing Patient Assets, Risks, and Experiences).
- **AHC-HRSN** (CMS Accountable Health Communities Health-Related Social Needs screening tool).
- **Hunger Vital Sign** (2-item food insecurity screen).
- **PHQ-2 / PHQ-9** (depression).
- **GAD-7** (anxiety).
- **AUDIT-C / AUDIT** (alcohol use).
- **CMS / Joint Commission instruments** as adopted by individual organizations.

Capture both:
- The **panel** LOINC for the whole instrument.
- The **item** LOINC for each question, with the answer coded against the LOINC answer list (or appropriate value set).

This preserves the structured response and enables longitudinal comparison.

---

## SNOMED CT for SDOH conditions, goals, interventions

The Gravity Project FHIR IG uses SNOMED CT for the post-screening clinical layer:

- **Condition / problem** — e.g., "homelessness," "food insecurity," "transportation insecurity," "social isolation" coded against SNOMED CT.
- **Goal** — patient-level goal SNOMED CT codes (e.g., "improve food security").
- **Intervention / procedure** — referral, counseling, resource provision.
- **Service Request** — for the referral itself.

Verify each SNOMED CT concept against the current US Edition release.

---

## Screening instruments

Common standardized HRSN / SDOH screening instruments (each with specific licensing, scoring, and supported settings):

| Instrument | Domains | License |
|------------|---------|---------|
| PRAPARE | Housing, food, transportation, employment, education, social support, stress, safety, race/ethnicity, language | NACHC; free for use |
| AHC-HRSN (CMS) | Housing instability, food insecurity, transportation, utilities, interpersonal safety | CMS-published; free |
| Hunger Vital Sign | Food insecurity (2 items) | Children's HealthWatch; free |
| WE CARE | Multi-domain HRSN | Boston Medical Center; permission-based |
| THRIVE | Multi-domain | Cambridge Health Alliance; permission-based |
| Health Leads | Multi-domain | Health Leads; free |

Verify current license and scoring before implementing.

---

## Area-level deprivation indices

Area-level indices supplement individual-level screening by linking patient geocode to neighborhood-level deprivation:

- **Area Deprivation Index (ADI)** — University of Wisconsin; census-block-group level; national and state percentiles.
- **Social Vulnerability Index (SVI)** — CDC ATSDR; census-tract level; four themes (socioeconomic, household composition, minority status/language, housing/transportation).
- **Neighborhood Atlas** — ADI access platform.
- **CMS DAC (Distressed Communities Index)** in some payment models.

Verify current index version, vintage (e.g., ADI 2020 vs. earlier), and geographic resolution before integrating. Geocoding accuracy materially affects index assignment.

---

## Gravity Project FHIR IG

The **Gravity Project** is the HL7 FHIR Implementation Guide that standardizes SDOH/HRSN data exchange.

- Defines FHIR profiles for Patient, Condition, Observation, Goal, Procedure, ServiceRequest, Task, Consent and related resources for SDOH use cases.
- Specifies value sets bound to LOINC for screening, SNOMED CT for conditions/goals/interventions, ICD-10-CM Z-codes where billable.
- Covers screening, diagnosis, goals, interventions, and outcomes — the closed-loop referral cycle.

Verify the current Gravity SDOH Clinical Care IG version on HL7 (published versions evolve regularly) before implementing; profile names and bindings change.

---

## USCDI SDOH data classes

USCDI (United States Core Data for Interoperability) added SDOH data classes starting with v2/v3. Data classes typically include:

- SDOH assessments.
- SDOH problems / health concerns.
- SDOH goals.
- SDOH interventions.

USCDI is what ONC-certified EHRs must expose via FHIR (ONC §170.315(g)(10) Standardized API). Verify the current USCDI version applicable to your certification level.

---

## Operational use

### Capture

- Embed standardized screens (PRAPARE, AHC-HRSN, Hunger Vital Sign, etc.) in EHR encounter workflows.
- Capture answers as discrete LOINC-coded data, not free text.
- Derive SNOMED CT problems on positive screens via clinical decision rules.

### Code

- Map screening result -> SNOMED CT problem -> ICD-10-CM Z-code where billable.
- Persist the screening date, instrument version, and respondent (patient vs. proxy).

### Refer and resolve

- Route positive HRSN screens to closed-loop community resource referral platforms (Unite Us, findhelp.org, NowPow, Aunt Bertha).
- Track referral status (open, accepted, resolved, declined).
- Closed-loop status itself is part of the structured record.

### Integrate into analytics

- Add SDOH features to risk scoring and segmentation overlays.
- Validate for bias before adding SDOH-derived features to operational models.
- Stratify quality measures by SDOH status to expose equity gaps.

### CMS programs

- HRSN screening is increasingly tied to payment and quality programs. Verify current measure and payment specifications (e.g., HEDIS, eCQMs, CMS Stars, MA SDOH adjusters) for each measurement year.

---

## Verification expectations

Before implementing:

1. Verify current ICD-10-CM tabular for Z55-Z65 scope.
2. Verify current LOINC codes for each screening instrument and its answer values.
3. Verify current SNOMED CT US Edition for SDOH problem / goal / intervention codes.
4. Verify current Gravity Project FHIR IG published version.
5. Verify current USCDI version SDOH data classes.
6. Verify current CMS HCC / HHS-HCC / HEDIS payment-year specs for SDOH inclusion.
7. Verify license and current version of each screening instrument.
8. Verify current ADI / SVI vintage and geocoding pipeline accuracy.

Always pull from current sources — codes, bindings, and program rules evolve.
