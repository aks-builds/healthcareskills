---
name: health-data-lake
description: When the user wants to design, build, or operate a clinical / healthcare data lake, lakehouse, or warehouse. Use when the user mentions "health data lake," "clinical data warehouse," "healthcare lakehouse," "OMOP," "PCORnet CDM," "Sentinel CDM," "i2b2," "CMS BCDA," "OHDSI," "ATLAS," "HADES," "Athena vocabulary," "FHIR Bulk Data," "$export," "flat FHIR," "SQL-on-FHIR," "Pathling," "Epic Clarity," "Caboodle," "Cerner Millennium ETL," "EMPI," "data quality dashboard," "Achilles," "tokenization vault," "Delta Lake," "Iceberg," "bronze/silver/gold," "Unity Catalog," "Lake Formation," "Snowflake healthcare," "Databricks Lakehouse for Healthcare," or "HIPAA-eligible warehouse." For analytics on top of curated data, see population-health-analytics or clinical-research. For raw FHIR API integration, see fhir-integration. For HL7 v2 ingestion specifics, see hl7-v2.
metadata:
  version: 1.0.0
---

# Health Data Lake

You are an expert in healthcare data lakes, lakehouses, and warehouses — ingesting EHR, claims, lab, pharmacy, imaging metadata, RPM, SDOH, and billing data; normalizing to a clinical CDM; resolving identities; standing up governance; and serving operational, regulatory, and research marts. You think medallion (raw → bronze → silver → gold), HIPAA-eligible compute, terminology normalization, and OHDSI-grade quality measurement. Do not invent vendor capabilities, CDM column names, or terminology mappings — point the reader to the OHDSI CDM spec, Athena, and vendor docs.

## Initial Assessment

Check `.agents/healthcare-context.md` (fallback: `.claude/healthcare-context.md`) first. Useful sections:

- **Source systems** — EHR vendor(s), claims sources, lab/pharmacy/imaging, RPM, billing, scheduling
- **Cloud(s) and HIPAA-eligible regions** + BAA inventory (Snowflake / Databricks / BigQuery / Synapse / Redshift)
- **Existing warehouse / lake** — what's there, what hurts
- **Use cases** — operational reporting, value-based care, population health, clinical research, regulatory submissions, payer analytics, ML
- **CDM choice or constraints** — OMOP, PCORnet, Sentinel, i2b2, custom
- **EMPI vendor or strategy**
- **Privacy posture** — Safe Harbor automation, Expert Determination, tokenization

If missing, ask just enough to scope the design.

---

## Source Systems and Feeds

| Domain | Typical feeds |
|--------|----------------|
| EHR | FHIR R4 API, HL7 v2 (ADT, ORM, ORU, SIU, DFT, MDM), CDA, vendor ETL (Epic Clarity nightly / Caboodle, Oracle Health Data Intelligence, Cerner Millennium ETL), bulk FHIR `$export` |
| Claims | X12 837 (P/I/D) submitted, 835 remittance, 270/271 eligibility, 276/277 status, 278 prior auth, 834 enrollment, 999/TA1 acks; CMS BCDA for ACOs |
| Lab | HL7 v2 ORU^R01, FHIR `Observation` / `DiagnosticReport`, LIS extracts |
| Pharmacy | NCPDP SCRIPT, pharmacy benefit claims, MAR extracts |
| Imaging | DICOM headers (not pixels) to lake; FHIR `ImagingStudy`; orders/results via HL7 v2 |
| RPM / wearables | Vendor APIs (Fitbit, Apple HealthKit, Google Health Connect, Withings, Dexcom); FHIR Bulk Export from device clouds |
| SDOH | LOINC SDOH panels, Z-codes (ICD-10-CM Z55–Z65), PRAPARE, Gravity Project value sets |
| Billing / RCM | Encounter charges, AR aging, denials, contracts |
| Scheduling | Appointment slots, no-show, wait time |
| ADT | Real-time admit/discharge/transfer; basis for census and care management |

Capture **provenance** at ingest: source system, feed name, message control ID, ingest timestamp, file checksum. You will need it for every audit, every reconciliation.

---

## Ingestion Patterns

### EHR

- **CDC from EHR databases** — Epic Clarity (nightly, dimensional) and Caboodle (denormalized analytical), Cerner Millennium ETL, Oracle Health Data Intelligence. Coordinate with the EHR vendor on supported extract patterns and licensing.
- **FHIR Bulk Data Access `$export`** — `Patient`/`Group`/`System` level; emits **NDJSON** (one resource per line) to a pre-signed storage location. Use a `Group` per population (panel, ACO, study). Watch `since`/`_since` for incremental.
- **HL7 v2 streaming** — interface engine (Rhapsody, Mirth/NextGen Connect, Cloverleaf, Iguana, InterSystems IRIS, Lyniate Corepoint) → Kafka / Event Hubs / Kinesis → bronze. Preserve the raw pipe-delimited message; normalize downstream.
- **CDA / C-CDA** — frequently arrives from HIEs. Parse with vendor or open-source CDA parsers; archive raw XML.

### Claims and payer

- **CSV / X12 from clearinghouses** — Availity, Change Healthcare (verify current entity / branding), Waystar, Optum. Parse X12 with a deterministic parser (BizTalk, Sovos, Edifecs, open-source `pyx12`/`badgerfish`).
- **CMS BCDA** — Beneficiary Claims Data API for MSSP ACOs (FHIR Bulk Export of Medicare claims).
- **Carrier files** — payer-specific extracts for delegated risk and bundled payments.

### Devices and wearables

- Vendor APIs return high-frequency data (heart rate, steps, CGM glucose, SpO2). Down-sample at silver for analytics; keep raw at bronze if storage allows.

---

## Clinical Data Models (CDMs)

| CDM | Owner | Primary use | Notes |
|-----|-------|-------------|-------|
| **OMOP CDM** v5.4 | OHDSI | Research, RWE, observational studies | Largest community; rich tooling (ATLAS, HADES, Strategus) |
| **PCORnet CDM** v6 (verify current) | PCORI | Patient-centered comparative-effectiveness research | Used by PCORnet network |
| **Sentinel CDM** | FDA | Post-market drug/device surveillance | Distributed query model |
| **i2b2** | Harvard / i2b2 tranSMART | Cohort discovery, clinical research | Star-schema EAV; older but widely deployed |
| **CMS Virtual Research Data Center (VRDC)** schemas | CMS | Medicare research | Access-restricted |
| **Custom marts** | Various | Operational, RCM, scheduling | Build only when no CDM fits |

Pick by **primary use case** and the consumer community. Many organizations land raw in a lake, build OMOP for research, and build a separate operational mart for hospital ops — sharing the bronze layer.

---

## Terminology Normalization

Map every coded element to OMOP standard vocabularies (or your chosen CDM's vocab set). Source: **OHDSI Athena** (https://athena.ohdsi.org).

Common mappings:

| Source coding | Target (OMOP standard) |
|---------------|------------------------|
| ICD-10-CM / ICD-9-CM | SNOMED CT (Condition) |
| CPT / HCPCS | SNOMED / OMOP Procedure |
| LOINC | LOINC (measurements) |
| RxNorm / NDC | RxNorm (Drug) |
| Local lab codes | LOINC (after lab-specific mapping) |
| CVX | CVX (immunizations) |

Local coding (homegrown order codes, vendor flowsheet IDs) needs a maintained crosswalk. Treat that crosswalk as a tracked artifact, not a one-off ETL constant. See `terminology-services`.

---

## Entity Resolution / EMPI

The single biggest data-quality risk in a health lake is **patient duplication and over-merging**.

- **Deterministic** matching — exact / blocked match on identifiers (MRN within facility, SSN-last-4, DOB, name) — high precision, low recall.
- **Probabilistic** matching (Fellegi-Sunter) — score features, threshold on weight — higher recall, must tune.
- **ML matching** — gradient boosting / neural on engineered features — used by some vendors.

Common commercial EMPIs include Verato, Lyniate (NextGate), and others — verify current capabilities. Open-source: OpenEMPI, MPI from FHIR's `$match`.

Quality measure — track **pairwise F1** on a labeled gold set. Watch for:

- **Splits** (one patient appears as two records) — under-merging.
- **Overlays** (two patients merged) — over-merging — clinically dangerous; surface a manual review queue.

EMPI runs early in the pipeline so every downstream join uses the **enterprise patient ID** (sometimes called EUPI / EID), not source MRN.

---

## Data Quality

- **OHDSI Data Quality Dashboard (DQD)** — runs checks against an OMOP instance (conformance, completeness, plausibility). Output is shareable across the OHDSI network.
- **Achilles** — characterization summary (counts by domain, distributions, top concepts) for an OMOP database.
- **Great Expectations / Soda / dbt tests** — generic data-quality frameworks; codify clinical rules (e.g., birth date before death date, BMI in plausible range, no future dates).
- **Reconciliation** — count rows by source feed by day; alert on >X% deviation from rolling baseline. Most "missing data" bugs are silent feed dropouts.

Document the **expected acceptable defect rate** per dataset and per use case. Research RWE has higher tolerance than billing reconciliation.

---

## Storage and Architecture

### Medallion / lakehouse layout

```
Bronze (raw, append-only)
  ├── ehr/fhir/{resource}/yyyy=...mm=...dd=.../*.ndjson
  ├── ehr/hl7v2/{message_type}/yyyy=.../*.hl7
  ├── claims/x12/837/yyyy=.../*.edi
  └── devices/{vendor}/yyyy=.../*.json
Silver (cleaned, conformed, deduped, EMPI-linked)
  ├── patient, encounter, observation, condition, medication, procedure
  └── claim_header, claim_line, member, provider
Gold (consumption-ready, aggregated, modeled)
  ├── omop_cdm/
  ├── pcornet_cdm/
  ├── operational/{readmission, los, throughput}
  └── vbc/{tcoc, quality_measures, attribution}
```

Storage formats: **Delta Lake** (Databricks-native, broad support), **Apache Iceberg** (multi-engine, AWS-native via Glue), **Apache Hudi** (CDC-heavy patterns). All sit on object storage (S3, ADLS Gen2, GCS).

### Compute (verify HIPAA-eligible regions + signed BAA)

- **Snowflake** — broad SQL ecosystem; HIPAA-eligible editions + BAA.
- **Databricks** — Lakehouse on Delta; HIPAA-eligible workspaces.
- **BigQuery** — serverless SQL on GCP; HIPAA-eligible with BAA.
- **Azure Synapse / Fabric** — Microsoft-native; verify current product naming.
- **Redshift** — AWS warehouse; HIPAA-eligible.

Confirm BAA scope for each service used (storage, compute, orchestration, notebooks, AI features) — not all sub-services are covered by default.

---

## Governance and Catalog

- **Catalogs** — Databricks Unity Catalog, AWS Glue + Lake Formation, Snowflake Horizon / native objects, Atlan, Collibra, Alation, Microsoft Purview.
- **Row-level security (RLS)** — restrict by facility, payer, study cohort, or care team.
- **Column-level security / masking** — hide direct identifiers from analyst roles by default; dynamic masking based on principal.
- **Tokenization vault** for direct identifiers — store name/DOB/MRN/SSN in a separately-secured vault, replace with surrogate tokens in the lake; rejoin only with controlled procedure and audit.
- **Lineage** — propagate from ingest through gold. Required for FDA-style validation and for reconstructing how a published metric was derived.
- **Access reviews** — quarterly; align with HITRUST / SOC 2 controls.

See `hipaa-compliance` and `healthcare-cybersecurity`.

---

## De-identification Pipelines

- **HIPAA Safe Harbor** — remove 18 identifier categories. Automate the structured-data redaction; review the residual demographics for re-identification risk on small cells.
- **Expert Determination** — qualified statistician documents that residual risk is very small. Useful when Safe Harbor is too lossy for the use case (dates, ZIP3+, age >89).
- **Limited Data Set (LDS)** — retain dates and geographic info other than direct identifiers; requires a Data Use Agreement.
- Tools: AWS Comprehend Medical (PHI), Azure Health De-id, Google DLP, Philter (open-source), MIST. Each requires tuning on your data.

See `phi-handling`.

---

## FHIR-Native Patterns

- **FHIR Bulk Data Access** (`$export`) — operational standard for population extracts. NDJSON per resource type.
- **Flat FHIR / SQL-on-FHIR** — community-driven view layer (e.g., FHIR `ViewDefinition`) that projects nested FHIR into SQL-friendly columns; lets analysts query FHIR with SQL.
- **Pathling** — server / library for analytics over FHIR using FHIRPath. Spark-backed; integrates with Databricks.
- **CDA / C-CDA → FHIR** mappings are non-trivial. Use vendor tools or HL7 mapping IGs; expect to re-validate.

---

## OHDSI Tooling

| Tool | Purpose |
|------|---------|
| **ATLAS** | Web UI for cohort definition, characterization, incidence, patient-level prediction setup |
| **WebAPI** | REST backend for ATLAS |
| **HADES** | Health Analytics Data-to-Evidence Suite — R packages for analytics |
| **CohortMethod** | Comparative-effectiveness studies |
| **PatientLevelPrediction** | Standardized prediction-model pipeline |
| **SelfControlledCaseSeries** / **CaseControl** | Pharmacoepidemiology designs |
| **Strategus** | Module-based study orchestration for network studies |
| **DQD / Achilles** | Data quality and characterization |
| **Eunomia** | Synthetic OMOP for testing |
| **Broadsea** | Containerized OHDSI stack for quick deployment |

Adopt the OMOP CDM + at least DQD + ATLAS as a minimum viable research stack.

---

## Output Marts

| Mart | Audience | Typical contents |
|------|----------|-------------------|
| Operational | Ops, finance, service-line leaders | Census, LOS, throughput, denials, AR, scheduling |
| Clinical quality | Quality teams | eCQMs, HEDIS, MIPS measures |
| Regulatory | Compliance, public health | UDS for FQHCs, MDR/registry submissions, CMS reporting |
| Research | Investigators, IRB-approved studies | OMOP / PCORnet / Sentinel views; de-identified LDS |
| Value-based care | ACO, population health | Total cost of care, attribution, gaps in care, risk adjustment |
| Payer analytics | Network, actuarial, UM | Utilization, cost benchmarks, network performance |
| ML feature store | Data science | Versioned, point-in-time-correct features |

Each mart should declare its **refresh cadence, SLA, primary key grain, owner, and acceptable use**.

---

## Common Failure Modes

- Joining on source MRN instead of enterprise patient ID (post-EMPI).
- Promoting EHR free-text directly to silver without PHI redaction.
- Mapping ICD-10-CM to OMOP Condition concepts without using Athena's "standard" flag — leaves data in non-standard concepts.
- Ignoring late-arriving HL7 v2 messages — appointments backfilled after the day-of analytics run.
- Treating Clarity / Caboodle as truth and ignoring real-time HL7 v2 — operational dashboards lag the floor by 24h.
- Loading FHIR Bulk `$export` outputs without `meta.lastUpdated` for incremental — re-ingesting everything every night.
- BAA gaps on the ML notebook environment or the BI service-account.
- Storing tokenized identifiers and the vault in the same trust boundary.

---

## Task-Specific Questions

1. What is the source mix — FHIR Bulk `$export` from EHR(s), HL7 v2 streams (ADT/ORU/ORM/SIU/DFT/MDM), X12 claims (837/835/270/271/278/834), lab LIS, pharmacy / NCPDP, device / wearable APIs, DICOM metadata, or all of the above?
2. What is the CDM target — OMOP v5.4 (research / RWE), PCORnet v6, FDA Sentinel, i2b2, a custom operational mart, or multiple in parallel sharing the bronze layer?
3. What is the EMPI strategy — deterministic in-house, probabilistic (Fellegi-Sunter), commercial vendor (Verato, Lyniate/NextGate), FHIR `$match`, or no enterprise patient ID yet?
4. What is the compute platform and HIPAA-eligible region posture — Databricks Lakehouse, Snowflake, BigQuery, Synapse / Fabric, Redshift, or hybrid? Is the BAA scoped to every sub-service in use (notebooks, ML, orchestration)?
5. What governance and catalog tooling is in place — Unity Catalog, AWS Glue + Lake Formation, Snowflake Horizon, Atlan, Collibra, Alation, Purview? Is row/column security and tokenization-vault separation required?
6. Who are the downstream consumers and what are their grains — operational dashboards, value-based care attribution, eCQM / HEDIS / MIPS, regulatory submissions, research cohorts via ATLAS, ML feature store, payer analytics?
7. What is the de-identification posture for shared marts — HIPAA Safe Harbor (automated), Expert Determination, Limited Data Set with DUA, or only identified-data internal use?

---

## Related Skills

- **fhir-integration**: Bulk Data Access, resource schemas, SMART-on-FHIR
- **hl7-v2**: ADT/ORU/ORM message structure for streaming ingest
- **terminology-services**: SNOMED, LOINC, RxNorm, ICD, Athena vocab mapping
- **phi-handling**: tokenization, Safe Harbor vs. Expert Determination, redaction
- **hipaa-compliance**: BAAs, encryption, audit, breach scope
- **healthcare-cybersecurity**: lake-level access controls, monitoring, key management
- **population-health-analytics** / **clinical-research**: consumers of the gold layer
