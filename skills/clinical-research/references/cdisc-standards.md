# CDISC Standards

Reference for the CDISC (Clinical Data Interchange Standards Consortium) standards stack that governs clinical research data from collection through regulatory submission. **Always verify current standard versions against the FDA Data Standards Catalog and the CDISC published standards before implementing.** Required versions vary by submission type and by regulator (FDA, PMDA, NMPA).

## Contents

- Standards stack overview
- CDASH (Clinical Data Acquisition Standards Harmonization)
- SDTM (Study Data Tabulation Model)
- ADaM (Analysis Data Model)
- Define-XML
- Dataset transport formats (XPT, Dataset-XML, Dataset-JSON)
- SEND (Standard for Exchange of Nonclinical Data)
- Therapeutic Area User Guides (TAUGs)
- Controlled terminology
- End-to-end flow
- Verification expectations

---

## Standards stack overview

| Standard | Purpose | Stage of study |
|----------|---------|----------------|
| CDASH | Data collection field naming and structure | Protocol -> eCRF design |
| SDTM | Tabulated submission datasets | Database lock -> SDTM mapping |
| ADaM | Analysis-ready datasets | Analysis -> tables/listings/figures |
| Define-XML | Metadata wrapper for SDTM/ADaM | Submission packaging |
| Dataset-XML / Dataset-JSON | Dataset transport format | Submission packaging |
| SEND | Nonclinical (animal/tox) studies | Preclinical |
| TAUGs | Therapeutic area-specific guidance | Per-disease implementation |
| Controlled Terminology | Coded values for CDASH/SDTM/ADaM/SEND | Cross-cutting |

The forward flow is **CDASH -> SDTM -> ADaM**, with **Define-XML** describing each tabulated and analysis dataset. **Plan the mapping forward**: every CRF field designed in CDASH should have a known SDTM target variable at design time.

---

## CDASH (Clinical Data Acquisition Standards Harmonization)

**Purpose**: Standardize the structure, content, and field naming of data collected on case report forms (eCRFs).

**Scope**: Demographics, vital signs, adverse events, medical history, concomitant medications, lab results, exposure, disposition, study terms — collection-stage versions of what later becomes SDTM domains.

**Key concept**: CDASHIG (the CDASH Implementation Guide) maps each CDASH variable to its target SDTM variable, making forward traceability explicit at design time.

**Use during eCRF design**:
- Name eCRF fields per CDASH conventions.
- Tag each field with its target SDTM domain and variable in EDC metadata.
- Adopt CDASH question text and answer values where the protocol does not require customization.

Verify the current CDASHIG version against the CDISC library and FDA Data Standards Catalog.

---

## SDTM (Study Data Tabulation Model)

**Purpose**: A standardized, tabulated representation of all study data for regulatory submission. One row per observation; variables grouped into domains.

**Domain examples** (verify current SDTMIG for full list and structure):

| Class | Domain | Content |
|-------|--------|---------|
| Special-purpose | DM | Demographics |
| Special-purpose | SE | Subject Elements |
| Special-purpose | CO | Comments |
| Interventions | EX / EC | Exposure (planned / actual) |
| Interventions | CM | Concomitant Medications |
| Interventions | SU | Substance Use |
| Events | AE | Adverse Events |
| Events | DS | Disposition |
| Events | MH | Medical History |
| Findings | LB | Laboratory Test Results |
| Findings | VS | Vital Signs |
| Findings | EG | ECG Test Results |
| Findings | QS | Questionnaire (e.g., PRO instruments) |
| Findings | PE | Physical Examination |
| Findings | DA | Drug Accountability |
| Trial Design | TS | Trial Summary |
| Trial Design | TA | Trial Arms |
| Trial Design | TE | Trial Elements |
| Trial Design | TI | Trial Inclusion/Exclusion criteria |
| Trial Design | TV | Trial Visits |

**Key concepts**:
- **SDTMIG** (Implementation Guide) specifies how each domain is built.
- **Variable types** are core (required for all observations), expected (required if collected), permissible (optional), or sponsor-defined supplemental.
- **Supplemental qualifiers** (SUPP--) carry non-standard variables associated with parent domain rows.
- **RELREC** captures relationships between records across domains.

Verify the current SDTMIG version your submission must use against the FDA Data Standards Catalog (FDA requires specific versions for each submission type).

---

## ADaM (Analysis Data Model)

**Purpose**: Derive analysis-ready datasets from SDTM that support the tables, listings, and figures in the Clinical Study Report (CSR), with full traceability back to SDTM.

**Key dataset types**:
- **ADSL** (Analysis Dataset Subject Level) — one row per subject, with treatment, populations, and key derived variables.
- **BDS** (Basic Data Structure) — one row per parameter per analysis timepoint per subject (e.g., ADLB for labs, ADVS for vital signs, ADQS for questionnaires).
- **OCCDS** (Occurrence Data Structure) — one row per occurrence (e.g., ADAE for adverse events, ADCM for concomitant meds).

**Key principles** (per ADaMIG):
- **Traceability** — every ADaM variable must trace back to SDTM (or be a clearly-derived analysis variable with documented derivation).
- **Self-documenting** — variable names, labels, derivation rules embedded.
- **Analysis-ready** — no further processing required to produce the analyses.

Verify the current ADaMIG and ADaM Implementation Guide for Therapeutic Areas as applicable.

---

## Define-XML

**Purpose**: An XML-based metadata file that describes the contents and structure of submitted SDTM and ADaM datasets — the "data dictionary" required by regulators.

**Contents**:
- Each dataset (location, structure, key variables).
- Each variable (name, label, type, length, controlled terminology, derivation for ADaM).
- Codelists used.
- Computational algorithms.
- Origin (CRF page, derived from, electronic source).

**Versions**: Define-XML 2.0 vs. 2.1 differ in structure and validation rules. Verify which version your submission target requires.

**Required for FDA submission** under the Study Data Technical Conformance Guide.

---

## Dataset transport formats

| Format | Purpose | Status |
|--------|---------|--------|
| **XPT (SAS Transport v5)** | Legacy required format for FDA submission | Historical; being replaced |
| **Dataset-XML** | XML-based dataset transport | Emerging; verify FDA acceptance |
| **Dataset-JSON** | JSON-based dataset transport | Newer; verify FDA acceptance status |

Historically FDA has required SAS Transport (XPT v5) files for submission datasets. Newer formats (Dataset-XML, Dataset-JSON) are being phased in; verify current acceptance against FDA Data Standards Catalog and Study Data Technical Conformance Guide.

---

## SEND (Standard for Exchange of Nonclinical Data)

**Purpose**: The CDISC standard for nonclinical (animal toxicology, safety pharmacology, carcinogenicity) study data submission to FDA.

- Required for IND, NDA, BLA submissions that include nonclinical studies — verify current FDA requirement scope.
- Domains parallel SDTM but tailored to nonclinical (e.g., BG body weight gain, FW food and water consumption, MA macroscopic findings, MI microscopic findings).
- SENDIG specifies the implementation. Verify current version.

---

## Therapeutic Area User Guides (TAUGs)

**Purpose**: CDISC-published guidance for specific therapeutic areas (oncology, cardiovascular, diabetes, neurology, infectious disease, dermatology, etc.) that interpret CDASH/SDTM/ADaM in the context of a disease.

- Often essential for accurate SDTM mapping of TA-specific assessments (e.g., RECIST in oncology, MDS in cardiology).
- Verify the current TAUG version for each applicable disease.

---

## Controlled terminology

CDISC publishes **Controlled Terminology** (CT) — coded values for variables across CDASH, SDTM, ADaM, and SEND.

- Published as a versioned package on the **NCI EVS** (Enterprise Vocabulary Services) FTP / API.
- Refreshed quarterly (verify current cadence).
- Bound to variables in CDASHIG and SDTMIG.

Use the CT version aligned to the IG version your submission targets. Do not mix.

---

## End-to-end flow

```
Protocol
   │
   ▼
CDASH eCRF design ────────┐
   │                       │
   ▼                       ▼
EDC database (raw)    EDC metadata: each CRF field → target SDTM variable
   │
   ▼
SDTM mapping ────────────► Define-XML (SDTM)
   │
   ▼
ADaM derivation ─────────► Define-XML (ADaM)
   │
   ▼
Tables, Listings, Figures (CSR)
   │
   ▼
eCTD submission (FDA / EMA / PMDA / NMPA)
```

Every variable in a CSR table cites an ADaM variable, which traces to SDTM, which traces to the eCRF field, which traces to the protocol-defined assessment. Plan the reverse trace before designing the eCRF.

---

## Verification expectations

Before each study database build, lock, and submission:

1. Verify the **FDA Data Standards Catalog** for required CDISC standard versions per submission type.
2. Verify current **CDASHIG**, **SDTMIG**, **ADaMIG**, **SENDIG**, **Define-XML** versions against CDISC publications.
3. Verify applicable **TAUG** versions for each therapeutic area in scope.
4. Verify current **Controlled Terminology** package against NCI EVS.
5. Verify **PMDA Japan** and **NMPA China** standards if those regulators are in scope — they have their own catalogs.
6. Verify **EU CTIS** data expectations if EU CTR 536/2014 applies.
7. Run **CDISC validation** (Pinnacle 21, OpenCDISC, or successor tools) before each submission.

Standards versions evolve every year — do not assume the version used in a prior study still applies.
