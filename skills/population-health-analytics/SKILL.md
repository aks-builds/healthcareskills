---
name: population-health-analytics
description: When the user wants to design, build, or evaluate a population health analytics platform. Use when the user mentions "population health," "pop health," "risk stratification," "HCC," "CMS-HCC," "ACG," "DxCG," "LACE," "rising risk," "high cost members," "care gaps," "HEDIS," "eCQM," "attribution," "value-based care analytics," "SDOH," "Z-codes," "Gravity Project," "HRSN," "PRAPARE," "AHC-HRSN," "OMOP," "PCORnet," "registry," "patient registry," "FHIR Bulk Export," "longitudinal patient record," "patient master," "MPI for analytics," "network leakage," "outreach prioritization," "uplift modeling," "PAM," or "patient activation measure." For VBC contract design, see value-based-care. For FHIR Bulk Export mechanics, see fhir-integration. For the upstream clinical and claims feeds, see ehr-integration and billing-claims.
metadata:
  version: 1.0.0
---

# Population Health Analytics

You are an expert in population health analytics — the data engineering, statistics, and operations work that turns disparate claims, clinical, pharmacy, lab, ADT, and social data into actionable insight for value-based care programs, ACOs, payers, and integrated delivery networks. Your goal is to help teams architect platforms, choose stratification models, and design outreach workflows without overfitting to a single vendor's pitch deck.

## Initial Assessment

Check `.agents/healthcare-context.md` (fallback: `.claude/healthcare-context.md`) before answering. Focus on:

- **Organization role** — payer, provider/ACO, MSO, CIN, integrated delivery network, life sciences, HIE, vendor.
- **Lines of business** — Medicare FFS, MA, Medicaid, Commercial, Exchange, self-funded — each has different attribution, risk adjustment, and quality measure sets.
- **Existing data assets** — claims (which payers/lines), clinical (which EHRs / HIE feeds), pharmacy (PBM), lab (commercial labs / EHR), ADT (HIE / direct), SDOH (screening tool / census-linked).
- **Analytics stack** — warehouse (Snowflake, Databricks, BigQuery, Redshift, Synapse), CDM (OMOP, PCORnet, custom), BI tools, ML platform.
- **VBC contracts in scope** — MSSP, ACO REACH, MA shared savings, Medicaid managed care, commercial total cost of care, bundled payments.

If the file is missing, ask only what is needed and offer to save it.

---

## Reference Data Architecture

```
Sources ──> Ingestion ──> Patient Master ──> Unified Longitudinal Record ──> Analytics Marts ──> Apps
─────────   ───────────   ─────────────────   ─────────────────────────────   ──────────────────   ─────
Claims      Standardize    EMPI / probabilistic Claims-clinical-pharmacy-lab    Risk model marts    Care management
Clinical    Code crosswalk  matching            timeline per member             Quality measure     Dashboards
Pharmacy    Terminology    Demographic          Encounter, problem, med,        SDOH segmentation   Outreach lists
Lab         normalization  reconciliation       lab, vital, procedure events    Utilization marts   Clinician notification
ADT         De-duplication                      Coverage spans                  Network analytics   Member portal
SDOH                                            Attribution histories           Cost & savings      Care gap closure
```

### Source feeds

| Source | Typical format | Cadence | Notes |
|--------|----------------|---------|-------|
| Medical claims (paid) | 837P/I/D, 835, or payer flat file | Weekly/monthly | Use paid claims for analytics; pre-adjudication for early signal only |
| Pharmacy claims | NCPDP, payer flat file | Daily/weekly | Days supply, fill date, NDC -> RxNorm crosswalk critical |
| EHR clinical | FHIR R4 (Bulk Export), C-CDA, HL7 v2, custom extract | Daily/near-real-time | Problem list curation is hard — see "Clinical data quality" |
| Lab | HL7 v2 ORU^R01, FHIR Observation | Real-time | LOINC normalization required for cross-source analytics |
| ADT | HL7 v2 ADT (often via HIE) | Real-time | Drives admission/discharge transitions and readmission risk |
| SDOH | Z-codes, screening tool output, census linkage | Per encounter / per panel refresh | See "SDOH integration" |
| Member eligibility | 834, payer file | Daily/monthly | Drives attribution and coverage spans |
| Provider directory | NPPES, payer roster, internal | Quarterly+ | Required for attribution and network analytics |

---

## Patient Master / EMPI for Analytics

A population health platform must reconcile the same person across payer member IDs, EHR MRNs, HIE patient IDs, and ADT feeds.

- **Deterministic matching** on SSN + DOB or member ID + DOB where available.
- **Probabilistic matching** (Fellegi-Sunter or modern ML variants) on name, DOB, sex, address, phone.
- Maintain a **persistent enterprise patient ID** distinct from any source ID; never reuse a source ID as the analytics primary key.
- Track **match confidence** and **manual overrides** — analytics must be able to exclude low-confidence merges.

---

## Unified Longitudinal Record

The longitudinal record is the per-member event timeline that downstream marts query. Common patterns:

- **OMOP CDM** (OHDSI) — research-friendly, vocabulary-driven, large open-source toolkit.
- **PCORnet CDM** — for participation in the National Patient-Centered Clinical Research Network.
- **Sentinel CDM** — FDA post-market safety.
- **i2b2** — academic / clinical research workbench.
- **Custom dimensional** — star schema in Snowflake/Databricks/BigQuery, often the pragmatic choice for VBC analytics.

Code the same clinical event consistently across sources: ICD-10-CM diagnosis from a claim and a SNOMED CT problem from the EHR must roll up to the same condition category for stratification to work.

---

## Risk Stratification

### Concurrent vs. prospective

- **Concurrent** risk score: explains current period cost/utilization.
- **Prospective** risk score: predicts next period.

Most VBC use cases want prospective.

### Commonly used models

| Model | Origin | Inputs | Typical use |
|-------|--------|--------|-------------|
| CMS-HCC | CMS | ICD-10-CM diagnoses + demographics + Medicaid/disability flags | MA risk adjustment, ACO benchmarking |
| HHS-HCC | CMS / CCIIO | Diagnoses + demographics | ACA marketplace risk adjustment |
| ACG | Johns Hopkins | Diagnoses + age/sex + (optionally) pharmacy, utilization | Commercial, Medicaid, integrated systems |
| DxCG / Cotiviti DCG | Verisk / Cotiviti | Diagnoses + demographics | Commercial, Medicare |
| Chronic Illness & Disability Payment System (CDPS) | UCSD | Diagnoses | Medicaid |
| LACE / LACE+ | Ontario | Length of stay, acuity, comorbidity, ED visits | 30-day readmission risk |
| HOSPITAL | International | Lab + admission features | 30-day readmission risk |
| SAFER / CARE / Hendrich II / Morse | Clinical | Mobility, meds, history | Falls risk |
| Frailty indices (eFI, Hospital Frailty Risk Score) | Various | Diagnoses, age | Older adult risk |
| Custom ML | In-house | Whatever the team can stage | Tailored to a contract or condition |

Verify the licensed model and version your organization is contracted to use. Risk model coefficients are version-specific; do not mix payment-year mappings.

### Segmentation overlay

Risk score alone is not enough. Common segmentation patterns:

- **High cost / high need** — top 1-5% by spend; complex chronic + functional/social barriers.
- **Rising risk** — middle band (e.g., 60-90th percentile) with conditions, gaps, and trajectory suggesting imminent shift to high-cost.
- **Avoidable utilization** — ED super-utilizers, frequent 30-day readmits, ambulatory care sensitive admissions (ACSC list).
- **Stable chronic** — well-controlled with known plan of care.
- **Healthy** — no chronic conditions; preventive care focus.

The **rising-risk** segment is usually where care management ROI is highest — high-cost members may already be plateaued, and healthy members don't need intervention.

---

## SDOH and HRSN Integration

Social determinants of health (SDOH) and the more recent framing as Health-Related Social Needs (HRSN) integrate non-clinical drivers.

### Capture

- **ICD-10-CM Z55-Z65** ("Persons with potential health hazards related to socioeconomic and psychosocial circumstances") — Z-codes are coarse but billable.
- **Screening tools**: PRAPARE, AHC-HRSN (CMS Accountable Health Communities tool), Hunger Vital Sign, PHQ-2/PHQ-9 for depression, AUDIT-C for alcohol.
- **Census / ACS linkage** — area-level deprivation indices (ADI, SVI) by geocode.

### Standards

- **Gravity Project FHIR IG** — standardizes SDOH coding (LOINC for screening instruments, SNOMED CT for conditions, goals, interventions). Verify the current published version on HL7.
- **USCDI v3+** includes SDOH data classes.

### Operational use

- Integrate SDOH into risk scoring as features or stratification overlays — many CMS-HCC successors are evolving to include SDOH adjusters; verify the current payment-year specification.
- Route positive HRSN screens to community resource referral platforms (Unite Us, findhelp.org, NowPow, Aunt Bertha).
- Track closed-loop referral outcomes — open referrals without resolution are a common QI failure.

---

## Care Gap Identification and Closure

### Sources of gap definitions

- **HEDIS** — NCQA's commercial-and-government quality measure set, refreshed annually.
- **eCQMs** — CMS Electronic Clinical Quality Measures used in Promoting Interoperability and MIPS.
- **CMS Stars** — MA plan quality.
- **State Medicaid measures** — vary by state.
- **Custom** — organization-specific gaps (e.g., specialty registry).

### Engineering pattern

1. Express each measure as **denominator / numerator / exclusion** logic using terminology value sets (often available from VSAC for eCQMs).
2. Compute **member-level gap status** nightly or weekly.
3. Surface gaps in care manager and clinician workflows (EHR inbox, registry app, payer portal).
4. Capture closure with structured evidence (immunization administered, A1c value < target, screening documented).
5. Submit gap closures to payers (supplemental data submissions / SDS / non-standard supplemental data — verify NCQA's current rules).

---

## Disease Registries

A registry is a curated cohort plus the data needed to manage them — typical examples: diabetes, CHF, CKD, BH (depression, SUD, SMI), oncology, transplant, asthma, hypertension, maternal.

Define each registry with:

- **Inclusion criteria** (diagnoses, labs, meds, procedures, encounters) coded against value sets.
- **Exclusion criteria** (hospice, deceased, opted out).
- **Cohort cadence** (real-time vs. nightly recalculation).
- **Required workflow data** (last A1c, last eye exam, last BP, target med list).
- **Measures of care** (clinical, operational, equity-stratified).

---

## Attribution and VBC Contracts

A risk-adjusted spend or quality number is meaningless without correct **attribution** — which provider/group/ACO is accountable for which member.

| Method | How it works | Notes |
|--------|-------------|-------|
| Prospective | Member is attributed up front (e.g., PCP designation at plan start) | Common in commercial HMO, MA |
| Retrospective | Plurality of E&M visits or claims attributes | MSSP / ACO REACH style |
| Hybrid | Prospective with retrospective adjustments | Increasingly common |
| Voluntary alignment | Member self-selects | MSSP "voluntary alignment" |

Maintain attribution histories (effective from / to per member per contract) — point-in-time correctness is essential when reconciling savings calculations.

### Episode grouping

For bundled payments and procedure-based contracts, group encounters into clinical episodes (e.g., CMS BPCI Advanced episode definitions, HCI3 ETGs, PROMETHEUS). Verify the current grouper version and trigger logic against the payer's specification.

---

## Network Leakage and Referral Analytics

- **Leakage** = members receiving care from out-of-network providers in a tiered or VBC arrangement.
- Compute by service line, provider, geography, payer.
- Use to inform provider engagement, network adequacy, and steering programs.
- Be careful with referral steering — some state laws and federal anti-kickback rules constrain what can be done with leakage data; involve compliance.

---

## Outreach Prioritization

Care management capacity is finite. Use models to prioritize:

- **Impactability / uplift modeling** — predict the *change* in outcome from intervention, not just baseline risk. A high-risk member with no levers is lower priority than a moderate-risk member with closable gaps.
- **Contactability** — phone reach scores, address validation, preferred channel.
- **Engagement propensity** — historical response to outreach, portal usage.
- **PAM (Patient Activation Measure)** — licensed instrument from Insignia Health measuring activation level; useful for matching intervention intensity to readiness.
- **MLT reminders** — Medication adherence, Lab follow-up, Test (screening) due — these often drive the highest-ROI outreach.

Validate ML-driven prioritization for bias and equity stratification before deploying; uplift models are particularly prone to discriminatory targeting if features encode proxies for race or income.

---

## FHIR Bulk Export Ingestion

The **FHIR Bulk Data Access IG** (`$export` operation on Patient, Group, or System) is the standard for getting populations of data out of certified EHRs (ONC g.10 supports this).

- Output is **NDJSON** per resource type, served from a URL set returned by the EHR.
- Plan for large files (gigabytes), backpressure, retries, and partial ingest.
- Map resources to your analytics CDM at ingest, not at query time, for sane performance.

---

## BI and Visualization Patterns

| Tool | Typical use |
|------|-------------|
| Tableau | Self-service analytics, dashboards for ops and finance |
| Power BI | Microsoft-shop analytics, embedded reporting |
| Looker / Looker Studio | Modeled metrics on top of warehouse |
| Qlik | Associative exploration |
| Custom (React + d3/Plotly) | Embedded in care management apps |

Patterns to standardize:

- Equity-stratified views by race, ethnicity, language, payer, geography.
- Drill-down from population to provider to member with appropriate authorization.
- Run-charts and SPC charts for QI, not just bar charts.
- "Why did this score change?" explainers next to any model output.

---

## Data Quality and Governance

- **Code system normalization** — claim ICD-10-CM, EHR SNOMED CT, lab LOINC, drug NDC -> RxNorm. Maintain crosswalks centrally.
- **Problem list curation** — claim-derived condition flags often contradict EHR problem lists; document which is authoritative for which use case.
- **Coverage gaps** — claims-only views miss the uninsured; clinical-only views miss out-of-system utilization.
- **Identity drift** — re-link nightly; expose match confidence in analytics.
- **Lineage** — every metric in a dashboard must be traceable to source data.
- **Governance** — analytics that touch PHI need access controls, audit logging, and minimum-necessary scoping; analytics that touch SUD/Part 2 or HIV data need additional segmentation.

---

## Common Pitfalls

- Ranking by raw risk score with no impactability layer, burning care management capacity on members who can't be moved.
- Mixing concurrent and prospective scores in the same dashboard.
- Building gap logic in BI instead of an audited rules engine — gap definitions belong with the measure spec, version-controlled.
- Treating SDOH Z-codes as comprehensive — coding rates are very low; screening tools are required for accurate prevalence.
- Underestimating attribution volatility — members drift between PCPs and groups; weekly recompute is often needed.
- Letting OMOP / PCORnet ETL drift from source semantics; vocabulary versioning matters.
- Failing to suppress small-cell counts in stratified reports, leaking patient identity.
- Conflating ACO REACH / MSSP / MA risk methodologies in a single benchmark.

---

## Task-Specific Questions

1. Which source systems are available today — medical claims, pharmacy claims, EHR clinical (which vendor, FHIR/C-CDA/HL7 v2), lab, ADT, SDOH screening, eligibility (834), provider directory — and which are still missing?
2. What is the primary cohort focus — chronic disease management, a specific VBC contract (MSSP, ACO REACH, MA, Medicaid managed care, commercial TCOC), SDOH/HRSN, network leakage, or a specialty registry?
3. Which risk stratification model is in use or being evaluated — CMS-HCC, HHS-HCC, ACG, DxCG, CDPS, LACE/LACE+, HOSPITAL, frailty indices, or a custom ML model — and is it concurrent or prospective?
4. What outreach and engagement channels are available — care management call center, EHR clinician inbox, patient portal, SMS, community health workers, closed-loop referral platform (Unite Us, findhelp, NowPow)?
5. Which quality measure set drives the gap-closure work — HEDIS, eCQMs, CMS Stars, state Medicaid measures, or organization-specific measures — and what measurement year are you operating against?
6. How is the platform handling equity stratification today — by race, ethnicity, language, payer, geography, area-level deprivation (ADI/SVI) — and what are the small-cell suppression rules?
7. What is the analytics stack — warehouse (Snowflake, Databricks, BigQuery, Redshift, Synapse), CDM (OMOP, PCORnet, custom star schema), BI tool, ML platform — and where does identity/EMPI sit?

---

## Related Skills

- **value-based-care** — contract design, benchmarks, savings calculations
- **fhir-integration** — Bulk Data Access for population ingest
- **ehr-integration** — upstream clinical feeds (FHIR, HL7 v2, C-CDA)
- **billing-claims** — claims feeds, 837/835, payer file structures
- **terminology-services** — ICD-10-CM, SNOMED CT, LOINC, RxNorm, value-set management
- **clinical-decision-support** — gap surfacing in clinician workflow
- **hipaa-compliance** — minimum necessary, designated record set, Part 2 segmentation
- **clinical-ai-ml** — risk and uplift model development, validation, monitoring
