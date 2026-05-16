# Clinical Data Model Comparison

Reference for choosing among the major clinical Common Data Models (CDMs). Each was built for a different primary purpose; picking the wrong CDM for the use case costs years of rework. Verify current versions, tooling, and governance against each CDM's own documentation before commitment.

## Contents

- The Decision in One Page
- OMOP CDM
- PCORnet CDM
- FDA Sentinel CDM
- i2b2
- Comparison Table
- Multi-CDM Strategies
- When to Build a Custom Mart
- Common Pitfalls

---

## The Decision in One Page

| Use case | Primary recommendation |
|----------|------------------------|
| Real-world evidence, observational research, network studies, comparative effectiveness | OMOP CDM (OHDSI) |
| PCORI-funded patient-centered comparative-effectiveness studies, PCORnet network participation | PCORnet CDM |
| FDA-required post-market drug or device surveillance, distributed regulatory queries | Sentinel CDM |
| Cohort discovery and self-service research feasibility at an academic medical center | i2b2 |
| Hospital operations, finance, revenue cycle, real-time clinical ops | Custom operational mart, sharing the bronze layer with whatever CDM serves research |
| Value-based care, ACO attribution, total cost of care | Custom VBC mart, often layered on top of an OMOP-derived patient/encounter spine |

Many large organizations end up running **two or three** CDMs in parallel on the same bronze layer. This is normal. Choosing one to the exclusion of all others tends to leave a use case poorly served.

---

## OMOP CDM

### Owner and Community

OHDSI (Observational Health Data Sciences and Informatics), an open international research community. Current production version: **v5.4** (verify; v6.x has been in development).

### Primary Purpose

Observational research, real-world evidence, comparative-effectiveness research, drug safety, and patient-level prediction across institutions through standardized analyses.

### Structure

- Person-centric, event-based, longitudinal.
- Standard tables: `person`, `observation_period`, `visit_occurrence`, `condition_occurrence`, `drug_exposure`, `procedure_occurrence`, `measurement`, `observation`, `device_exposure`, `death`, `note`, `note_nlp`, plus derived eras (`condition_era`, `drug_era`, `dose_era`).
- Cost, payer, and provider tables for economic and provenance analysis.
- All coded data normalized to OMOP **Standard Concepts** (typically SNOMED CT for conditions, RxNorm for drugs, LOINC for measurements). Source codes preserved alongside the standard mapping.

### Vocabularies

Managed in the OHDSI Athena vocabulary (https://athena.ohdsi.org). Includes mappings from ICD-9-CM / ICD-10-CM, CPT / HCPCS, NDC, local lab codes, and more, to OMOP standard concepts. Vocabulary is versioned; pin the version used by your build.

### Tooling

Rich open-source ecosystem:

- **ATLAS** — web UI for cohort definition, characterization, incidence, prediction setup.
- **WebAPI** — REST backend for ATLAS.
- **HADES** — R packages for analytics (`CohortMethod`, `PatientLevelPrediction`, `SelfControlledCaseSeries`, etc.).
- **Strategus** — module-based study orchestration for network studies.
- **DQD (Data Quality Dashboard)** and **Achilles** — quality and characterization.
- **Eunomia** — synthetic OMOP for testing.
- **Broadsea** — containerized OHDSI stack.

### Who Uses It

A large, growing international network of academic medical centers, integrated delivery networks, payer-providers, and pharma research groups. Network studies frequently span tens of institutions.

### Strengths

- Largest open community in clinical CDM space.
- Mature tooling, standardized analyses, reproducibility-friendly.
- Strong vocabulary harmonization, especially for drugs and conditions.
- Network-study patterns (federated queries) are first-class.

### Watch Out For

- Visit / encounter modeling is opinionated; mapping from EHR encounters to OMOP visits requires care.
- Less natural for operational reporting and revenue cycle.
- Cost table and payer modeling are less mature than the clinical side.
- Vocabulary mappings: ensure you use Athena's "standard" concepts in OMOP analytic columns; non-standard concepts in the standard columns silently break OHDSI tools.

---

## PCORnet CDM

### Owner and Community

PCORI (Patient-Centered Outcomes Research Institute) and the PCORnet network. Current version: **v6.x** (verify against PCORnet documentation).

### Primary Purpose

Patient-centered comparative-effectiveness research within the PCORnet research network. PCORnet runs distributed queries across participating Clinical Research Networks (CRNs) and Patient-Powered Research Networks (PPRNs).

### Structure

- Encounter-anchored, with `demographic`, `encounter`, `diagnosis`, `procedures`, `vital`, `lab_result_cm`, `prescribing`, `dispensing`, `medication`, `condition`, `death`, `pro_cm` (patient-reported outcomes), `pcornet_trial`, `enrollment`, `obs_clin`, `obs_gen` (genomic).
- Codes typically kept in source (ICD-10-CM, CPT, LOINC, RxNorm); less aggressive standardization than OMOP.
- PCORnet has its own data-characterization and curation cycle for network participation.

### Who Uses It

PCORnet network participants — large academic centers, health-system CRNs, and PPRNs participating in PCORI-funded research.

### Strengths

- Well-defined participation pathway for PCORI-funded studies.
- Patient-reported outcomes (`pro_cm`) and trial recruitment (`pcornet_trial`) are first-class.
- Curation and data characterization processes are well-defined for network compliance.

### Watch Out For

- Smaller open-tooling ecosystem than OMOP.
- Less natural fit for non-network use cases.
- Code standardization is shallower; downstream analyses must handle source-coded data.

---

## FDA Sentinel CDM

### Owner and Community

FDA, operated through the Sentinel Coordinating Center.

### Primary Purpose

Post-market surveillance of regulated medical products (drugs, biologics, devices, vaccines) via distributed analysis across health-system data partners.

### Structure

- Person, enrollment, encounter, diagnosis, procedure, dispensing, lab, and vital tables suited to administrative claims and EHR-augmented data.
- Designed for **distributed queries**: the FDA / Sentinel center sends a standardized program to each data partner, who runs it locally and returns aggregate results — patient-level data does not leave the partner.
- Strong support for incident-event identification, propensity score-based comparisons, and signal detection.

### Who Uses It

FDA-affiliated data partners (Optum, HealthCore, Kaiser-Permanente regions, others — verify current Sentinel partner list).

### Strengths

- The standard for regulator-mandated post-market surveillance.
- Distributed query pattern protects partner data.
- Stable, well-documented analysis programs and modules.

### Watch Out For

- Narrower than OMOP for general research.
- Participation pathway is FDA-driven; not a CDM you'd typically pick without an FDA collaboration.
- Tooling is more program-specific and less general-purpose than OHDSI.

---

## i2b2

### Owner and Community

Originally a Harvard / Partners HealthCare initiative; now an open community with academic-medical-center adoption worldwide. Often paired with **SHRINE** for federation across institutions.

### Primary Purpose

Cohort discovery and self-service research feasibility — "how many patients in our system meet criteria X?"

### Structure

- Star-schema **Entity-Attribute-Value** design centered on the `observation_fact` table.
- Dimension tables for patients, concepts, providers, and visits.
- An ontology tree drives the web UI's drill-down concept browser.

### Who Uses It

Many academic medical centers, especially those with strong NIH / CTSA program ties, often as the first port of call for an investigator to scope a feasibility study before requesting de-identified data extracts.

### Strengths

- Mature web UI for non-technical research-feasibility queries.
- Federation via SHRINE is well-established.
- Lower lift than a full OMOP build for institutions that just need cohort discovery.

### Watch Out For

- EAV structure is hard to use for complex longitudinal analytics — researchers will often want a de-identified extract in a different shape (OMOP, flat tables).
- Tooling ecosystem is older and growing more slowly than OMOP.
- Many institutions are migrating analytics work from i2b2 to OMOP while retaining i2b2 for feasibility / cohort-discovery only.

---

## Comparison Table

| Dimension | OMOP | PCORnet | Sentinel | i2b2 |
|-----------|------|---------|----------|------|
| Primary use case | RWE / observational research | PCORI comparative effectiveness | FDA post-market surveillance | Cohort discovery / feasibility |
| Person-centric vs. encounter-centric | Person | Encounter | Person | Patient (EAV) |
| Code standardization | Strong (OMOP Standard Concepts) | Modest (source codes) | Modest | None enforced; ontology-driven |
| Federated query support | Strong (Strategus, network studies) | Strong (distributed queries by design) | Strong (Sentinel-driven) | SHRINE |
| Tooling ecosystem | Largest (OHDSI) | PCORnet-specific | FDA-program-specific | i2b2 / SHRINE |
| Patient-reported outcomes | Possible (Observation table) | First-class (`pro_cm`) | Limited | Possible via ontology |
| Genomics | Possible (Observation; some extensions) | `obs_gen` table | Limited | Possible via ontology |
| Best community size and momentum | OMOP | PCORnet (in PCORI-network context) | FDA-program | Steady, smaller |
| Vendor / commercial offerings | Many (Tuva, Datavant, vendor accelerators) | Some | FDA-program | Some |

---

## Multi-CDM Strategies

Many large organizations end up running multiple CDMs.

### Pattern: Share the Bronze, Diverge at Silver/Gold

- Bronze layer is a single, raw, append-only landing zone for source feeds (FHIR Bulk, HL7 v2, X12, lab, pharmacy, devices).
- Silver layer is cleaned, EMPI-linked, terminology-normalized clinical data.
- Multiple gold CDMs (OMOP for research, an operational mart for hospital ops, a VBC mart for the ACO, a PCORnet mart for network participation) are derived from the same silver — each curating and shaping it to its own conventions.

This avoids parallel ingest / EMPI / terminology pipelines and keeps lineage tractable.

### Pattern: OMOP as the Backbone, Map Out to Others

If OMOP is the primary research CDM, PCORnet and Sentinel-style marts can be derived from OMOP through documented mappings. Some institutions publish OMOP-to-PCORnet ETL recipes.

### Pattern: Source-of-Truth Hierarchy

For any analytic question, name the canonical CDM that answers it. Reporting from a non-canonical mart should reference the canonical one. This prevents "two dashboards disagree" politics.

---

## When to Build a Custom Mart

Build a custom mart (not a CDM) when:

- The use case is operational (real-time census, throughput, denials, AR, scheduling) — no CDM is designed for it.
- The use case is regulatory / compliance with a non-CDM-aligned schema (UDS, MDR registry, state APCD, NHSN).
- The data does not fit cleanly in any CDM (e.g., highly facility-specific workflows).

Document the custom mart with the same care as a CDM: grain, source-of-truth declaration, refresh cadence, owner, acceptable use.

---

## Common Pitfalls

- **Picking OMOP for operational reporting.** Visit / encounter modeling, billing fidelity, and real-time refresh are not OMOP's strengths.
- **Picking a custom mart for research and trying to publish.** Without standard vocabularies and reproducibility patterns, network studies and peer-reviewed publication are harder.
- **Treating Athena vocabulary as static.** Athena releases drive map changes; pin and version-control the vocabulary used in each OMOP build.
- **Ignoring DQD on an OMOP build.** Data Quality Dashboard catches real bugs; running it is non-negotiable for a research-grade OMOP.
- **Letting source-coded data leak into OMOP standard columns.** OMOP analytics expect Standard Concepts; non-standard concepts silently break OHDSI tools.
- **Building one CDM and assuming it serves everyone.** It almost never does; the typical end state is two or three CDMs sharing a bronze layer.
- **Not tracking which patients participated in PCORnet, Sentinel, or other network builds.** Patient-level participation flags are necessary for compliance and for opt-out handling.
- **Forgetting the operational layer.** Research CDMs typically lag the floor by 24+ hours (Clarity / Caboodle nightly). Operational dashboards must come from real-time HL7 v2 streams, not the research CDM.
