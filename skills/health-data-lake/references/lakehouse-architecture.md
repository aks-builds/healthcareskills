# Lakehouse Architecture for Healthcare

Reference for the medallion (bronze / silver / gold) lakehouse pattern applied to healthcare data, with tradeoffs across storage formats (Delta Lake, Apache Iceberg, Apache Hudi) and governance layers (Unity Catalog, AWS Lake Formation, Atlan).

## Contents

- The Medallion Pattern
- Bronze (Raw)
- Silver (Cleaned, Conformed)
- Gold (Consumable Marts)
- Storage Format Tradeoffs (Delta / Iceberg / Hudi)
- Governance Tooling (Unity Catalog / Lake Formation / Atlan)
- Compute Platforms (HIPAA-eligible)
- Healthcare-Specific Concerns
- Reference Topology
- Common Pitfalls

---

## The Medallion Pattern

Three layers, each with a defined role and SLA. Data flows one way (raw to consumer); each layer is rebuildable from the previous one given enough time.

```
Source systems
   ↓ (ingest, no transformation)
Bronze — raw, append-only, source-formatted, with provenance
   ↓ (clean, conform, EMPI-link, deduplicate, normalize)
Silver — cleaned, conformed, EMPI-linked, terminology-normalized
   ↓ (model, aggregate, denormalize, materialize)
Gold — consumption-ready marts: CDMs, operational, VBC, ML feature store
   ↓
Consumers — BI tools, ML pipelines, registry submissions, research, partners
```

Why this structure:

- **Bronze** preserves raw history; you can reprocess silver and gold without re-pulling from the source system.
- **Silver** centralizes the expensive cross-source logic (EMPI, terminology) so every gold mart benefits from one canonical version.
- **Gold** can be use-case-specific without re-litigating identity and terminology each time.

---

## Bronze (Raw)

### Purpose

Land everything, change nothing.

### Characteristics

- Append-only.
- Stored in source format (NDJSON for FHIR, pipe-delimited for HL7 v2, EDI for X12, JSON for device feeds, CDA XML).
- Partitioned by source, feed type, and date.
- Carries **provenance metadata**: source system, feed name, file name or message control ID, ingest timestamp, file checksum, optional batch identifier.
- Encrypted at rest. Access restricted to ETL and audit roles.

### Layout (example)

```
bronze/
  ehr/
    fhir/
      epic-prod/
        Patient/yyyy=...mm=...dd=.../Patient.ndjson
        Condition/yyyy=...mm=...dd=.../Condition_0.ndjson
      cerner-community/
        ...
    hl7v2/
      epic-prod/
        ADT/yyyy=...mm=...dd=.../*.hl7
        ORU/yyyy=...mm=...dd=.../*.hl7
    cda/
      hie-statewide/
        yyyy=...mm=...dd=.../*.xml
  claims/
    x12/
      payer-A/
        837P/yyyy=.../*.edi
        835/yyyy=.../*.edi
      cms-bcda/
        EOB/yyyy=.../*.ndjson
  devices/
    fitbit/yyyy=.../*.json
    dexcom/yyyy=.../*.json
  _manifest/
    epic-prod-fhir-bulk/yyyy=.../manifest.json
    payer-A-837P/yyyy=.../manifest.json
```

### Retention

Long. Bronze is the audit-and-reconciliation layer; "we lost it from bronze" is rarely acceptable. Many institutions retain bronze for years, possibly tiered to colder storage as it ages.

---

## Silver (Cleaned, Conformed)

### Purpose

Produce a single, canonical, cross-source clinical view that downstream marts can rely on.

### Activities

1. **Parse** raw formats into structured tables (NDJSON → flat columns, HL7 v2 segments → typed fields, X12 → claim header / line, CDA → structured sections).
2. **EMPI**: resolve every record to an enterprise patient ID (EUPI / EID). Carry the source MRN(s) as attributes for traceability but do not join on them downstream.
3. **Deduplicate**: same encounter or message arriving twice from different feeds.
4. **Conform**: column names, types, units, timezones across sources.
5. **Terminology normalize**: source codes (ICD-10-CM, CPT, local lab codes, NDC) mapped to standards (SNOMED, RxNorm, LOINC), with the source code preserved.
6. **Validate**: clinical sanity (birth before death, BMI in plausible range, dates not in the future), referential integrity (encounter references valid patient, observation references valid encounter).
7. **Tombstones**: apply deletions / retractions from upstream Bulk Export `deleted` lists, ADT cancellations, claim voids.

### Schema Style

Two common patterns:

- **Wide clinical tables**: `silver.patient`, `silver.encounter`, `silver.observation`, `silver.condition`, `silver.medication`, `silver.procedure`, `silver.claim_header`, `silver.claim_line`, etc. Each table has columns sufficient for most downstream uses.
- **Per-source then conformed**: `silver.epic.encounter`, `silver.cerner.encounter`, plus `silver.conformed.encounter` (union with source tag). Useful when source-specific lineage matters.

EMPI-linked enterprise patient ID is the primary join key throughout silver.

### Refresh Cadence

Mixed. Real-time (HL7 v2 streaming) for operational signals; batch (FHIR Bulk Export, Clarity nightly, X12 daily) for research-grade data. Document the cadence per silver table.

---

## Gold (Consumable Marts)

### Purpose

Use-case-specific tables that consumers query directly. Each gold mart has a declared owner, refresh cadence, primary-key grain, acceptable use, and SLA.

### Typical Marts

| Mart | Audience | Examples |
|------|----------|----------|
| OMOP CDM | Research, RWE | `person`, `visit_occurrence`, `condition_occurrence`, etc. |
| PCORnet CDM | PCORI network research | PCORnet-specified tables |
| Operational | Ops, finance | Census, LOS, throughput, denials, AR aging |
| Clinical quality | Quality team | eCQM, HEDIS, MIPS measure tables |
| Value-based care | ACO, population health | Attribution, total cost of care, gaps in care |
| Payer analytics | Network, actuarial | Utilization, cost benchmarks |
| Regulatory | Compliance, public health | UDS, registry submissions |
| ML feature store | Data science | Versioned, point-in-time-correct features |

### Modeling Choices

- Denormalized star or one-big-table for BI tools.
- Normalized CDMs (OMOP, PCORnet) for research tooling.
- Slowly-changing dimensions (SCD-2) for entities where history matters (provider affiliation, payer plan).
- Point-in-time-correct snapshots for ML feature stores so models do not leak future information.

### Refresh and SLA

Each mart declares its target freshness. Operational marts may refresh every 5–15 minutes from HL7 v2 streams; research marts may refresh nightly or weekly. Document and monitor against SLA.

---

## Storage Format Tradeoffs

Three open table formats dominate. All sit on object storage (S3, ADLS Gen2, GCS, others). All support ACID transactions, schema evolution, and time travel; they differ in maturity, engine support, and CDC characteristics.

### Delta Lake

- **Origin**: Databricks; open-sourced.
- **Strengths**: native in Databricks; broad engine support (Spark, Trino, Presto, Athena, BigQuery via federation, Snowflake via external tables); strong streaming integration; mature MERGE INTO; Delta Sharing for cross-organization sharing; Unity Catalog tight integration.
- **Watch**: historically Databricks-leaning; many engines now read Delta well, but Databricks is the most native.

### Apache Iceberg

- **Origin**: Netflix; Apache project.
- **Strengths**: multi-engine first-class (Snowflake, BigQuery, AWS Glue / Athena, Trino, Spark, Flink); hidden partitioning; strong support for very large tables and schema evolution; vendor-neutral governance via the table format spec; rapidly becoming the cross-engine open standard.
- **Watch**: streaming and CDC patterns are catching up to Delta; ecosystem maturity has improved fast but varies by engine.

### Apache Hudi

- **Origin**: Uber.
- **Strengths**: change-data-capture-heavy patterns; record-level upserts and deletes (copy-on-write and merge-on-read storage types); good for ingest from operational source-of-truth databases.
- **Watch**: smaller community than Delta and Iceberg; ecosystem narrower.

### How to Choose

| Constraint | Pick |
|------------|------|
| Already in Databricks | Delta Lake |
| Multi-engine open lakehouse (Snowflake + Athena + Spark) | Iceberg |
| CDC-heavy ingest from operational stores | Hudi (or Delta with carefully-built CDC) |
| Cross-organization sharing | Delta Sharing if Delta; Iceberg REST catalog if Iceberg |
| Long-term vendor neutrality | Iceberg has the broadest open-engine support; Delta is open-source but Databricks-aligned |

For most new healthcare lakehouse builds, the decision narrows to **Delta Lake (if Databricks-centric)** or **Iceberg (if multi-engine / cross-cloud)**.

---

## Governance Tooling

Pick a governance and catalog layer that aligns with your compute platform.

### Databricks Unity Catalog

- **Scope**: catalog, schema, table, view, volume; row- and column-level security; dynamic masking; lineage; audit logging; Delta Sharing.
- **Best fit**: Databricks-centric lakehouses on Delta. Native and tight.
- **Strengths**: integrated lineage across notebooks, jobs, and SQL; cross-workspace sharing.
- **Watch**: most powerful when everything runs through Databricks; non-Databricks engines have limited reach.

### AWS Lake Formation

- **Scope**: fine-grained access control on Glue Catalog tables across Athena, Redshift Spectrum, EMR, and (to varying degrees) other AWS-resident engines.
- **Best fit**: AWS-centric lakes built on S3 + Glue Catalog, especially with Iceberg.
- **Strengths**: integration with AWS IAM, LF-Tags for tag-based access, row- and column-level filters.
- **Watch**: cross-engine consistency outside the AWS analytic stack requires care; non-AWS engines see Lake Formation through varying mechanisms.

### Snowflake Native Objects

- **Scope**: roles, future grants, masking policies, row access policies, tag-based policies, lineage (Account Usage).
- **Best fit**: Snowflake-centric warehouses (and Snowflake's Iceberg-table support for lakehouse-style external tables).
- **Strengths**: very mature SQL-native governance.
- **Watch**: governance lives inside Snowflake; lakehouse data accessed by other engines needs separate governance.

### Atlan / Collibra / Alation / Microsoft Purview

- **Scope**: enterprise data catalogs, business glossary, data-product registries, stewardship workflow, lineage aggregation across compute platforms.
- **Best fit**: enterprises with multiple compute platforms (Snowflake + Databricks + BigQuery + on-prem) where one unified business-facing catalog is needed.
- **Strengths**: cross-platform stewardship; business glossary; PHI / PII tagging; data-product discovery.
- **Watch**: these are catalog layers, not access-control engines. Enforcement still happens in the compute platform (Unity Catalog / Lake Formation / Snowflake roles).

### Practical Combination

A common production combination in healthcare:

- **Compute-native** access enforcement (Unity Catalog **or** Lake Formation **or** Snowflake roles), enforcing PHI scoping at query time.
- **Enterprise catalog** (Atlan / Collibra / Purview) aggregating metadata, business glossary, lineage, and PHI tags across platforms.
- **Tokenization vault** stored separately from the analytic environment for direct identifiers, with controlled re-identification procedures.

---

## Compute Platforms (HIPAA-Eligible)

All require a signed Business Associate Agreement (BAA) with the cloud provider **and** confirmation that the specific sub-services in use are in scope.

- **Databricks**: Lakehouse on Delta; HIPAA-eligible workspaces on AWS, Azure, and GCP. Confirm the specific workspace tier and ML / serving features are in BAA scope.
- **Snowflake**: HIPAA-eligible editions (Business Critical and above on AWS / Azure / GCP) with BAA. Confirm Snowflake-managed services (Snowpark ML, Cortex, AI features) are in scope.
- **BigQuery**: GCP HIPAA-eligible with BAA. Confirm BigQuery ML and adjacent AI features are in scope.
- **Azure Synapse / Microsoft Fabric**: Azure HIPAA-eligible services with BAA. Verify current product naming and scoping.
- **Redshift / EMR / Athena / Glue**: AWS HIPAA-eligible with BAA.

For **every** managed service in use (notebooks, ML, AI features, monitoring, data catalogs, third-party connectors), confirm BAA scope. Default BAAs from cloud providers do not always cover every preview or AI-adjacent feature; gaps here are a frequent finding in HITRUST and SOC 2 audits.

---

## Healthcare-Specific Concerns

### PHI Boundary

The lake holds PHI. Treat every layer as PHI by default; require explicit, documented de-identification to reduce that classification.

- Direct identifiers (name, MRN, SSN, full DOB, address) stored in a **tokenization vault** separate from the analytic environment. Surrogate tokens stand in for them in the lake. Re-identification is a controlled procedure with audit.
- Limited Data Sets and Safe-Harbor / Expert-Determination de-identified outputs live in separate schemas / catalogs / accounts with their own access policies.

### Lineage

End-to-end lineage from source feed to gold metric is required for:

- FDA-style validation if any output influences clinical decisions (SaMD).
- Reproducing a published quality measure or regulatory submission.
- Investigating a data-quality incident.

Use the compute platform's native lineage (Unity Catalog, Snowflake Account Usage) augmented by an enterprise catalog where multiple platforms are involved.

### Audit

HIPAA, HITRUST, and state regulations require audit logs of:

- Who queried what data, when, from where.
- Privileged operations (grants, ETL service-account activity, governance changes).
- Re-identification events.

Retain audit logs per regulatory retention requirement (often 6+ years in US healthcare; verify per jurisdiction).

### Encryption

At rest and in transit; customer-managed keys (CMK / BYOK) where institutional policy requires it. Tokenization vault keys segregated from analytic environment keys.

### Reconciliation

Healthcare data feeds drop silently. Row-count and per-source / per-day reconciliation against a rolling baseline catches outages that file-count checks miss.

---

## Reference Topology

A widely-recognizable healthcare lakehouse topology, simplified:

```
Source systems
  Epic (Clarity nightly, Caboodle, FHIR Bulk Export, HL7 v2 streams)
  Cerner (Millennium ETL, HL7 v2 streams)
  Lab LIS (HL7 v2 ORU)
  Pharmacy (NCPDP)
  Claims (X12 837/835 from clearinghouses, BCDA for Medicare)
  Devices / wearables (vendor APIs)
  HIE (CDA / C-CDA, FHIR)

         ↓ ingestion (Kafka / Event Hubs / Kinesis for streams;
                       scheduled jobs for batch)

Bronze (object storage: S3 / ADLS Gen2 / GCS)
  Raw NDJSON / HL7 / X12 / CDA / JSON
  Manifests and provenance
  Encrypted, append-only

         ↓ batch + streaming transforms

Silver (Delta or Iceberg tables, managed via Unity Catalog or Glue + Lake Formation)
  Conformed clinical tables, EMPI-linked
  Terminology-normalized via OHDSI Athena
  Real-time updates from HL7 v2 streams
  Tombstones applied from upstream deletions

         ↓ mart-specific transformations

Gold
  omop_cdm.* (research)
  pcornet_cdm.* (network)
  ops.* (operational dashboards)
  quality.* (eCQM / HEDIS / MIPS)
  vbc.* (attribution, TCOC)
  ml_features.* (point-in-time-correct features)

         ↓

Consumers
  BI (Power BI, Tableau, Looker, Sigma)
  Research notebooks (Databricks, JupyterHub)
  ATLAS / OHDSI tooling
  Registry submissions
  Partner data exchange (Delta Sharing, Iceberg REST, FHIR Bulk)
  Patient apps via FHIR services on top of gold
```

Tokenization vault and PHI direct identifiers live in a separate, more-restricted environment, accessed only through controlled re-identification procedures.

---

## Common Pitfalls

- **Mixing layers.** Skipping silver and consuming bronze directly in a BI tool creates undocumented downstream logic and breaks lineage.
- **One-size-fits-all gold.** Trying to make a single OMOP gold mart serve operations, quality, VBC, and research stretches it past its design.
- **EMPI applied at gold instead of silver.** Every mart re-implements identity resolution slightly differently; downstream counts disagree.
- **Storing tokenized identifiers and the vault in the same trust boundary.** Defeats the purpose of tokenization.
- **BAA scope gaps on adjacent services.** ML notebooks, AI features, third-party connectors, BI service accounts — each must be in scope.
- **No reconciliation alerts.** File-count checks miss within-file row-count drops.
- **Refresh-cadence drift.** Operational dashboards lag the floor by a day because they read research-grade marts instead of real-time HL7 v2.
- **No SCD-2 where history matters.** Provider affiliation, payer plan, attribution all change; current-row-only modeling rewrites history.
- **Schema drift unhandled.** New optional FHIR elements, new HL7 v2 segments, new X12 versions — silver-layer schemas must evolve with versioning and tests.
- **No documented re-identification procedure.** Re-identification will be requested (legitimate investigation, recall, regulatory query). Without a procedure, it happens informally and breaks audit posture.
