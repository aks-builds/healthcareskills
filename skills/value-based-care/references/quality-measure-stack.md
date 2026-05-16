# Quality Measure Stack — HEDIS, eCQM, MIPS, Star Ratings

Reference for engineers building quality measure engines for VBC analytics. Describes each major measure framework at a high level — what it is, who maintains it, who uses it, and how it intersects with other frameworks. Does **not** enumerate specific measure IDs, numerator / denominator definitions, or current performance-year measure lists — those change annually.

> Verify all measure IDs, specifications, technical guidance, and submission requirements against the **current technical specification** from the relevant maintainer (NCQA HEDIS, CMS eCQI Resource Center, CMS QPP, CMS Star Ratings). Measures change every measurement year.

## Contents

- Framework comparison at a glance
- HEDIS
- eCQMs (electronic Clinical Quality Measures)
- MIPS CQMs (Quality Payment Program)
- Star Ratings
- Cross-cutting concerns
- Engineering patterns
- Common pitfalls

---

## Framework Comparison at a Glance

| Framework | Maintainer | Primary use | Data sources | Submission |
|---|---|---|---|---|
| **HEDIS** | NCQA | Commercial, MA, Medicaid plan quality and accreditation | Administrative + supplemental + ECDS | NCQA-certified vendor + plan submission |
| **eCQMs** | CMS (specs hosted at eCQI Resource Center) | CMS quality programs (PI, Hospital, MIPS Quality category eCQM option) | CEHRT FHIR / QDM data | QRDA I (patient-level) / QRDA III (aggregate) |
| **MIPS CQMs** | CMS (per measure steward) | MIPS Quality category (registry / claims / QCDR option) | Claims + registry + EHR | Registry, QCDR, claims, or web interface |
| **Star Ratings** | CMS | MA Part C and Part D contract performance | HEDIS + CAHPS + HOS + operational | Computed by CMS from multiple sources |

> Verify current measure lists, data source allowances, and submission paths from each maintainer.

---

## HEDIS

### What it is

The **Healthcare Effectiveness Data and Information Set** is NCQA's measure set for plan quality. HEDIS is widely used by:

- **Commercial plans** — health plan quality and accreditation
- **Medicare Advantage** — feeds Star Ratings
- **Medicaid** — Medicaid managed care quality and state quality programs
- **ACA marketplace** — quality reporting

### Maintainer

**NCQA (National Committee for Quality Assurance)**. NCQA publishes annual **HEDIS Technical Specifications** with measure definitions.

### Domains

HEDIS measures span:

- **Effectiveness of care** — preventive screenings, chronic care management
- **Access / availability** — appointment access, network adequacy
- **Patient experience** — typically tied to CAHPS surveys
- **Utilization** — service use rates
- **Electronic Clinical Data Systems (ECDS)** measures — measures that can use clinical EHR data in addition to administrative claims

### Data sources

- **Administrative** — claims data (mostly)
- **Hybrid** — administrative + sampled medical record review
- **ECDS** — administrative + structured clinical data from EHRs / HIEs / case management systems / registries

### Compliance and certification

Plans typically use an **NCQA-certified HEDIS engine** vendor and undergo **HEDIS Compliance Audit** by NCQA-licensed auditors. The certified-vendor pattern reduces the engineering scope but does not eliminate it — supplemental data, ECDS data feeds, and quality improvement reporting still need to be built.

> Verify current HEDIS specifications, certification, and audit requirements with NCQA.

### Measurement year vs. reporting year

HEDIS measurement year is the calendar year of clinical service; reporting happens in the following year. Engineers must keep measure-year-specific reference data (value sets, denominator definitions) separately.

---

## eCQMs (electronic Clinical Quality Measures)

### What it is

eCQMs are FHIR/CQL-based quality measures defined for **electronic** reporting from **certified EHR technology (CEHRT)**.

### Definition format

- **Clinical Quality Language (CQL)** — the logic language used to define measure populations (denominator, numerator, exclusions).
- **Quality Data Model (QDM)** or **FHIR** — data model the CQL runs against. CMS is progressively migrating eCQMs to FHIR.
- **Measure Authoring Tool (MAT)** — CMS-published tool for authoring eCQM logic.
- **Value Set Authority Center (VSAC)** — repository of NLM-curated value sets used in eCQMs (and MIPS CQMs).

### Where eCQMs are used

- **CMS Promoting Interoperability (PI) program** — formerly Meaningful Use, for hospitals.
- **MIPS Quality category** — the eCQM option (alongside MIPS CQMs, claims-based, and QCDR options).
- **Hospital Inpatient Quality Reporting** and other CMS programs.

### Submission

- **QRDA Category I** — patient-level XML containing the data used for measure calculation.
- **QRDA Category III** — aggregate XML containing measure-level summary data.

### Engineering notes

- eCQM logic runs over a **FHIR data lake** (modern) or **QDM data store** (legacy).
- **Value-set pinning** — VSAC value sets are versioned. Pin to the measurement-year version.
- **CEHRT requirements** — the source EHR must be certified to feed eCQM-eligible data.
- **CQL engine selection** — multiple open-source CQL engines exist (CMS-published reference engine, plus commercial implementations). Verify behavior matches the CMS reference.

> Verify current eCQM list, CQL versions, FHIR profile requirements, and CEHRT criteria at the **CMS eCQI Resource Center**.

---

## MIPS CQMs (Quality Payment Program)

### What it is

The **Quality Payment Program (QPP)** has two tracks for eligible Medicare clinicians:

1. **MIPS (Merit-based Incentive Payment System)** — clinicians scored across four categories.
2. **Advanced APM track** — clinicians sufficiently participating in an Advanced APM (e.g., qualifying MSSP / ACO REACH tracks) earn QP status and exemption from MIPS.

### MIPS categories

MIPS scoring spans four categories with **category weights that change annually**:

- **Quality** — measures from eCQMs, MIPS CQMs, Medicare Part B claims-based measures, or QCDR (Qualified Clinical Data Registry) measures.
- **Cost** — claims-based; no reporting effort. Uses TPCC (Total Per Capita Cost), MSPB-C (Medicare Spending Per Beneficiary - Clinician), and episode-based measures.
- **Improvement Activities (IA)** — attestation.
- **Promoting Interoperability (PI)** — CEHRT use, public health reporting, e-prescribing, etc.

> Verify the **current performance year's category weights** at the CMS QPP page. Weights have shifted multiple times.

### Quality category measure options

For the Quality category, clinicians choose measures from:

- **eCQMs** — submitted via QRDA III (aggregate) or QRDA I (patient-level via registry).
- **MIPS CQMs** — registry-spec measures submitted via QCDR / registry.
- **Medicare Part B claims-based measures** — calculated by CMS from claims for small practices.
- **QCDR measures** — specialty-specific measures published by a Qualified Clinical Data Registry.

### Submission paths

- **Registry / QCDR**
- **EHR (eCQM)**
- **Web interface** (historical; verify current availability)
- **Medicare Part B claims** (for small practices)
- **CMS Web Interface for groups** (historical; verify current status)

### APP (APM Performance Pathway)

Clinicians in certain APM tracks may report quality via the **APM Performance Pathway (APP)** — a streamlined reporting pathway aligned with MSSP and similar models. Verify current APP scope and measure set.

> Verify MIPS rules, category weights, measure list, submission paths, and APP scope at the current CMS QPP page.

---

## Star Ratings

### What it is

**CMS Medicare Advantage and Part D Star Ratings** are composite quality ratings for MA and Part D contracts. They drive **MA bonus payments** and **rebate retention** that fund supplemental benefits.

### Mechanism

Star Ratings are computed by CMS from:

- **HEDIS** measures (Part C clinical quality)
- **CAHPS** (Consumer Assessment of Healthcare Providers and Systems) — member experience survey
- **HOS** (Health Outcomes Survey) — health-status / functional measure
- **Operational measures** — appeals, complaints, call center, member retention, etc.
- **Part D-specific measures** — adherence, MTM, etc.

CMS publishes **measure weights** and **cut points** annually. Cut points historically used a clustering algorithm; recent updates introduced **Tukey outlier methodology** for cut-point determination.

### Engineering notes

- **Annual measure list and weight changes** — Star measure list and weights shift each year. Pin reference data per Star year.
- **Cut points** — Star tier assignment depends on where a measure value falls in CMS's published cut points. Engineers must replicate the cut-point logic exactly to project Star outcomes.
- **Tukey methodology** — Tukey outlier-removal in cut-point determination was added in recent years; verify current methodology.
- **Display year vs. measurement year** — the Star Rating in effect for a given payment year reflects performance on measures from earlier years; track the lag.

> Verify current Star measure list, weights, cut points, and Tukey methodology at the CMS Star Ratings page.

---

## Cross-Cutting Concerns

### Value sets

Most modern measure frameworks (eCQMs, MIPS CQMs, HEDIS to a degree) use **value sets** to define populations and clinical concepts. Value sets are versioned and change with each measurement year.

- **VSAC** is the authoritative repository for CMS-program value sets.
- **NCQA** publishes value sets for HEDIS.

> Verify value-set versions per measurement year.

### Measure version pinning

Measure logic is **measurement-year specific**. A 2026 measurement-year denominator definition differs from a 2025 definition. Pin the engine to the measurement year, not "current."

### Data sources

- **Claims** — most administrative measures.
- **EHR / FHIR** — eCQMs and HEDIS ECDS.
- **Registry / supplemental data** — case management systems, immunization registries, lab results.
- **Surveys** — CAHPS, HOS.

### CEHRT certification

eCQM and PI measures require data from **certified EHR technology**. Verify the EHR's certification status and the specific CEHRT version supported.

### Audit retention

Quality reporting requires retention of source data and measure calculation history sufficient to defend the submitted values under audit. NCQA HEDIS Compliance Audit, CMS QPP audit, and Star Rating data validation all examine source data and calculation logic.

---

## Engineering Patterns

### Single measure engine, multiple frameworks

Build a measure engine that:

- Supports **CQL-based** (eCQM, FHIR / QDM) and **specification-based** (HEDIS, MIPS CQM via registry-spec) measure logic.
- Pins reference data (value sets, measure logic) per measurement year.
- Outputs **numerator / denominator / exclusions** per measure per member with full provenance.
- Supports **what-if** analysis (gap-close projection, intervention impact).

### Source data lake

Underlying the measure engine, a **FHIR-aligned data lake** ingests:

- Claims (837 / 835)
- Clinical (FHIR, HL7 v2, CDA)
- Supplemental (HEDIS supplemental data feeds, registries)
- ADT
- Surveys (CAHPS / HOS results when available)
- SDoH

Normalize to FHIR resources where the source supports it.

### Provider-facing dashboards

Quality dashboards must show:

- Per-measure performance against benchmark / target
- Care gaps with **patient-level** drill-down
- Trend over time
- Provider attribution (TIN / NPI hierarchy)

See the value-based-care SKILL.md for TIN / NPI hierarchy concerns — provider roster freshness is a top defect source.

### Submission pipelines

For programs requiring submission:

- **QRDA I and III** generation and validation for eCQM-based programs
- **NCQA-certified vendor** integration for HEDIS where applicable
- **Registry / QCDR** integration for MIPS CQMs
- **Star measure-specific data feeds** for Part C / D submissions

### Audit reproducibility

Every measure value must be reproducible from raw source data using the pinned measure logic and value sets. Treat each performance year as an immutable snapshot.

---

## Common Pitfalls

- **Hard-coding measure logic** rather than sourcing from the published specification each measurement year.
- **Mixing value-set versions across years** — using a 2026 value-set version with 2025 measurement-year claims.
- **Ignoring CEHRT certification** — building eCQM pipelines on uncertified EHR data invalidates submissions.
- **Treating Star cut points as fixed** — they shift each year, and the Tukey methodology change has moved them further.
- **Failing to replicate CMS calculation methodology exactly** — small drift produces unreliable performance projections.
- **Encoding MIPS category weights from memory** without updating each performance year.
- **Building HEDIS engines without NCQA certification path** — plan submissions require NCQA-certified vendors or in-house engines that pass NCQA audit; an uncertified engine is fine for internal performance management but not for plan reporting.
- **Missing the lag** between measurement year and the Star Rating / reporting year in effect for payment.

> Verify current technical specifications, value-set versions, measure lists, and methodology against the relevant maintainer (NCQA, CMS eCQI Resource Center, CMS QPP, CMS Star Ratings) for the applicable measurement year.
