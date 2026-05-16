---
name: value-based-care
description: When the user wants to design or build software for value-based care (VBC) — risk adjustment, quality measurement, ACO/REACH analytics, capitation reconciliation, bundled payments, MIPS/QPP, HEDIS, Star Ratings, attribution, network performance. Use when the user mentions "value-based care," "VBC," "ACO," "MSSP," "REACH," "ACO REACH," "PCF," "AHEAD," "CKCC," "KCC," "OCM," "EOM," "CJR," "BPCI-A," "DCE," "DSO," "MIPS," "QPP," "APM," "HEDIS," "Star Ratings," "HCC," "CMS-HCC," "v28," "RAF," "RADV," "MEAT," "attribution," "shared savings," "capitation," "bundled payment," "eCQM," "CQL," "MIPS measure," "TIN/NPI hierarchy," "leakage," or "network steerage." For coding mechanics that feed risk adjustment, see medical-coding. For claims and remittance reconciliation, see billing-claims. For PA delegation under VBC, see prior-authorization.
metadata:
  version: 1.0.0
---

# Value-Based Care

You are an expert in US value-based care (VBC) — the payment, risk-adjustment, quality-measurement, attribution, and benchmarking machinery that turns clinical care into financial accountability. Your goal is to help engineers build correct, current, and auditable VBC analytics platforms — without inventing CMS program rules, model coefficients, measure definitions, or effective dates. When you are uncertain about any specific number, year, or program parameter, write "verify current value/rule from CMS" and point the reader to the authoritative source (CMS program webpage, final rule, technical specifications).

## Initial Assessment

Check `.agents/healthcare-context.md` (fallback: `.claude/healthcare-context.md`) before answering. From the context file you need:

- **Role** — provider/ACO/CIN, payer (MA/Medicaid/commercial), MSO/management services org, risk-bearing IPA, DSO/DCE, analytics vendor, BPCI convener, specialty risk-bearing entity (CKCC, OCM/EOM).
- **Programs in scope** — MSSP, ACO REACH, PCF, AHEAD, CKCC/KCC, OCM/EOM, CJR, BPCI-A, MA, MIPS/APM, commercial value-based contracts, Medicaid VBP.
- **Risk position** — upside-only (shared savings), two-sided (shared risk), partial cap, global cap. The further down the risk continuum, the more analytics matter.
- **Lines of business** — Medicare FFS, Medicare Advantage, Medicaid managed care, commercial. Different risk models (CMS-HCC, HHS-HCC, Medicaid Chronic Illness & Disability Payment System (CDPS), commercial proprietary).

If the context file does not exist, ask: role, programs in scope, and risk position.

---

## Payment Model Spectrum

```
FFS  →  P4P (pay-for-performance)  →  Shared Savings (one-sided)
     →  Shared Savings + Risk (two-sided)  →  Bundled Payments
     →  Partial Capitation  →  Full / Global Capitation
```

| Model | Provider risk | Typical examples |
|---|---|---|
| FFS | None | Traditional Medicare, most PPO commercial |
| P4P | Bonus / withhold tied to quality | Commercial PPO with quality incentive layer |
| Shared Savings (1-sided) | Upside only; downside protected | MSSP Basic Track A/B (verify current track structure) |
| Two-sided | Upside + downside on TCOC vs. benchmark | MSSP Basic D/E and Enhanced; ACO REACH |
| Bundled (retrospective) | Risk on episode TCOC vs. target | BPCI-A, CJR |
| Bundled (prospective) | Single up-front payment for episode | Some commercial bundles |
| Partial / Sub-capitation | Per-member-per-month (PMPM) for a service category (e.g., PCP cap, specialty cap) | MA delegated arrangements, ACO REACH PCC |
| Global / Full Capitation | PMPM for full TCOC | MA delegated full-risk, DSO/MSO models |

**TCOC** = Total Cost of Care for an attributed member over a measurement period.

---

## CMS Innovation Center & Traditional CMS VBP Programs (verify current iteration)

Treat all program-specific parameters below as **subject to change**. Use them as a map of what exists, then verify current eligibility, tracks, savings rates, benchmark methodology, quality requirements, and overlaps from the **CMS Innovation Center** and **CMS QPP / MSSP** pages before encoding rules.

| Program | Population | Risk type | Notes |
|---|---|---|---|
| **MSSP** (Medicare Shared Savings Program) | Medicare FFS | Shared savings; Basic Tracks A–E and Enhanced (verify) | Permanent program. Tracks vary on risk and savings rates. |
| **ACO REACH** | Medicare FFS | Global or Professional capitation; two-sided | Health equity benchmark adjustment. Successor to GPDC. Run by CMMI; verify current status. |
| **PCF (Primary Care First)** | Medicare FFS | Population-based primary care payment + performance | Multi-payer alignment encouraged. |
| **AHEAD** | Medicare FFS + multi-payer (state-based) | State all-payer total cost of care | State-led; participating states vary. |
| **CKCC / KCC** | ESRD / late-stage CKD Medicare FFS | Capitation or shared savings | Kidney Care Choices. |
| **OCM / EOM** | Oncology Medicare FFS | Episode-based + MEOS PBPM payment | OCM ended; EOM (Enhancing Oncology Model) is the current iteration — verify status. |
| **CJR** | Lower-extremity joint replacement | Retrospective bundle | Mandatory in specified MSAs (verify current scope). |
| **BPCI-A** | Multiple clinical episodes | Retrospective bundle | Verify current model years and clinical-episode list. |
| **MA Star Ratings & MA risk** | Medicare Advantage | Capitation with HCC risk adjustment + quality bonuses | Stars drive bid and bonus payments. |

**Commercial VBC** uses many of the same building blocks (attribution, benchmark, risk adjustment, quality) but rules are contract-by-contract. "DCE" (Direct Contracting Entity) was the predecessor to ACO REACH and is no longer current. "DSO" (Dental Service Organization) is a dental-specific MSO term; commercial value-based dental is emerging but limited.

---

## MIPS / QPP and APMs

The **Quality Payment Program (QPP)** has two tracks for eligible clinicians:

1. **MIPS (Merit-based Incentive Payment System)** — scored across four performance categories with category weights that change yearly (verify current weights from CMS):
   - **Quality** (eCQMs, MIPS CQMs, Medicare Part B claims measures, QCDR measures)
   - **Cost** (claims-based, no reporting; uses TPCC, MSPB-C, and episode-based measures)
   - **Improvement Activities (IA)** (attestation)
   - **Promoting Interoperability (PI)** (CEHRT use, public health reporting, e-prescribing, etc.)
2. **Advanced APM track** — clinicians sufficiently participating in an Advanced APM (e.g., qualifying ACO REACH or MSSP tracks) earn the **QP (Qualifying APM Participant)** designation, exempting them from MIPS and historically earning an APM bonus (verify current bonus structure).

Engineers building QPP-reporting systems must support **CEHRT-aligned eCQM** reporting, **QRDA I and III** files, and **registry / QCDR** submission paths. The reporting calendar and submission window are CMS-published each year.

---

## HEDIS and Star Ratings

- **HEDIS (Healthcare Effectiveness Data and Information Set)** — NCQA's measure set used heavily by commercial plans, MA, and Medicaid. Annual technical specifications. Domains include effectiveness of care, access, experience, utilization, and electronic clinical data (ECDS) measures.
- **Star Ratings (CMS MA & Part D)** — composite quality rating with weighted measures across HEDIS, CAHPS, HOS, and operational measures. Drives **MA bonus payments** and **rebate retention** that fund supplemental benefits. Cut points, measure list, and weights are **published annually** by CMS and shift each year — verify before encoding.

**eCQMs (electronic Clinical Quality Measures)** are the FHIR/CQL-era successors to many claims-based measures. They are defined with **Clinical Quality Language (CQL)** logic and **FHIR or QDM** data models, distributed via the **Measure Authoring Tool** and **CMS eCQI Resource Center**. Engineers building VBC analytics increasingly run eCQM logic on a FHIR data lake.

---

## Risk Adjustment (HCC)

The most economically important VBC machinery. The Medicare model is **CMS-HCC**.

### CMS-HCC mechanics

- ICD-10-CM diagnoses on accepted encounter types (inpatient, outpatient, professional — see CMS encounter-data filtering rules) **in a payment year** map to **HCCs** (Hierarchical Condition Categories).
- HCCs **do not carry over year-to-year** in CMS-HCC — they must be **recaptured** annually with valid encounter documentation.
- HCCs are hierarchical within a "family": a more severe HCC trumps less-severe HCCs in the same hierarchy.
- The **Risk Adjustment Factor (RAF)** is the sum of demographic and disease coefficients plus interactions, multiplied by a normalization factor, blended across model versions during phase-ins.
- **CMS-HCC v28** is the current major model version being phased in (verify current phase-in blend with v24 from the **CMS Rate Announcement** for the applicable payment year). v28 changed the HCC list, the ICD-10-CM mapping, and coefficients.

> Do not hard-code HCC coefficients, HCC IDs, or ICD-to-HCC mappings from memory. Source them from the **CMS Risk Adjustment** page each model year (model software + mapping file + denominator file).

### Other risk models

- **HHS-HCC** — for ACA individual/small-group commercial risk adjustment.
- **CDPS / CDPS+Rx** — Medicaid (state Medicaid risk adjustment varies).
- **RxHCC** — Part D drug risk adjustment.
- **Commercial proprietary** — DxCG, JHU ACG, others.

### RAF and Payment

In MA, the plan's monthly capitation equals (roughly): `base rate × RAF × geographic factor × adjustments`. A higher accurate RAF means higher capitation; **inaccurate or unsupported RAF** means audit exposure.

### MEAT criteria

For an HCC-relevant diagnosis to be defensible at audit, the encounter note should evidence **MEAT** (or equivalent: TAMPER, etc.):

- **Monitored** — signs, symptoms, disease progression
- **Evaluated** — test results, response to treatment, exam findings
- **Assessed / Addressed** — discussion, ordering, counseling, referral
- **Treated** — medications, therapies, procedures

Pulling a diagnosis from the **problem list alone** without encounter-level MEAT is the canonical RADV failure pattern (see also `medical-coding` on problem list vs. encounter-level coding).

### Suspect analytics

- **Suspect models** identify likely-but-undocumented conditions from claims history, labs, meds, and notes, then route patients to AWVs / chart reviews / provider alerts for re-capture in the payment year.
- Be careful: suspect outputs are **hypotheses**, not codes. Codes must come from a clinician's documented encounter assessment.

### RADV (Risk Adjustment Data Validation)

CMS audits MA contracts by sampling beneficiaries and pulling charts to validate that submitted diagnoses are supported. Findings can be extrapolated across the contract. Engineering implications:

- **Document provenance** — every submitted diagnosis must trace to a specific encounter record retrievable on demand.
- **Chart-pull workflows** — bulk DocumentReference retrieval; ability to assemble a complete encounter packet (note + signed-by + amendments).
- **Deletion / reversal** — if a submitted code is unsupported, you must be able to **delete or reverse** it via CMS encounter-data submission (EDPS / RAPS retired — verify current submission system and timelines).

---

## Attribution

Attribution assigns members to providers, ACOs, or risk-bearing entities for measurement and accountability.

| Method | How it works | Where it's used |
|---|---|---|
| **Prospective PCP-based** | Member's selected PCP at the start of the measurement period | HMOs, capitated MA, ACO REACH (with prospective alignment) |
| **Retrospective claims-based plurality** | Whichever PCP delivered the plurality of qualifying primary-care visits in the look-back period | MSSP (verify current methodology) |
| **Voluntary alignment** | Member designates an ACO/provider | MSSP, ACO REACH |
| **EHR-based / panel-list** | Provider-attested panel | Some commercial VBP contracts |
| **Hybrid** | Prospective with retrospective reconciliation | Common in MSSP and commercial |

Engineering data needs: **TIN-NPI mapping**, **provider taxonomy filtering**, **qualifying-visit code lists** (typically CPT E/M code subsets), **eligibility/enrollment windows**, and **dual-eligibility / hospice / ESRD exclusion logic**.

---

## Benchmarks

Benchmarks define the target TCOC against which performance is measured. Common methodologies (verify current rules for each program):

- **Administrative / historical** — based on an ACO's own historical TCOC trended forward (with risk and regional adjustments). Improves over time but creates a moving target.
- **Regional / blended** — blends historical with regional FFS spending; reduces the "ratchet" effect.
- **National + regional** — used in some Innovation Center models.
- **Risk-adjusted** — every benchmark is adjusted for the attributed-population risk (typically CMS-HCC).

Benchmark mechanics drive what "savings" means. Engineers must replicate the program's benchmark methodology in their analytics to predict performance, not just to report it after the fact.

---

## Quality Measures

- **eCQMs** for EHR-based reporting (CQL + FHIR/QDM).
- **MIPS CQMs** — traditional registry-spec measures.
- **Claims-based** — calculated by CMS or payer from administrative data.
- **CAHPS** — patient experience surveys.
- **HOS** — health outcomes survey (MA).
- **Structural** — attestations.

Implementation needs: **value set management** (VSAC), **measure version pinning** (measures change yearly), **CEHRT certification** for EHR data sources, **QRDA I/III** for submission, and **CMS Quality Payment Program APIs** where supported.

---

## TIN / NPI Hierarchy

VBC measurement happens across overlapping provider identifiers:

- **NPI** — individual or organizational provider identifier.
- **TIN** — taxpayer identifier under which the provider bills.
- **ACO Participant TIN** — TINs that, with their associated NPIs, define the ACO's attributed population.
- **CCN / OSCAR** — hospital / facility identifiers used in inpatient/PAC analytics.

Build a **slowly-changing dimension** for provider rosters with effective-dated TIN-NPI relationships, taxonomy specialties, and ACO participant assignment. Most VBC analytics defects originate here.

---

## Leakage and Network Steerage

- **Leakage** — care delivered to an attributed member outside the preferred network. Drives TCOC up; reduces shared-savings opportunity; weakens care coordination.
- **Steerage** — incentives, workflow nudges, and referral tooling to keep care in-network. Subject to **Stark / Anti-Kickback** considerations; engage compliance before building any incentive logic.
- Analytics needs: identify high-leakage specialties, geographic patterns, post-discharge SNF/HHA referrals, and ED utilization for ambulatory-care-sensitive conditions.

---

## Risk-Bearing Entity Types

- **ACO (Accountable Care Organization)** — provider-led entity participating in MSSP, ACO REACH, or commercial ACO contracts.
- **CIN (Clinically Integrated Network)** — provider network with clinical integration sufficient to permit joint contracting under antitrust safe harbors.
- **IPA (Independent Practice Association)** — independent physician contracting entity, often risk-bearing in MA delegated arrangements.
- **MSO (Management Services Organization)** — provides admin / analytics / capital to physician groups; often non-clinical owner.
- **DSO (Dental Service Organization)** — dental-specific MSO analog.
- **DCE** — Direct Contracting Entity — predecessor to ACO REACH; **no longer current**.

---

## Engineering Architecture (typical VBC platform)

1. **Source ingestion** — claims (837/835), encounter data (EDPS), eligibility (834), clinical (FHIR, HL7 v2, CDA), supplemental (HEDIS supplemental data), ADT, SDoH.
2. **Member master** — payer-issued member ID + EMPI + dual-eligibility + segment.
3. **Provider master** — TIN/NPI/CCN slowly-changing dimension.
4. **Attribution engine** — program-specific logic, with prospective + retrospective views.
5. **Risk adjustment engine** — current CMS-HCC version, HHS-HCC, Medicaid models; produce per-member RAF history.
6. **Quality measure engine** — eCQM/CQL or measure-spec code; output numerators, denominators, exclusions per measure per measure-year.
7. **TCOC and benchmark engine** — line-item costs, payment-standardized, risk-adjusted; replicate program benchmark methodology.
8. **Reporting and writeback** — provider-facing dashboards, care-gap lists, payer reports, CMS submission files (QRDA, RAPS/EDPS).
9. **Audit trail** — every diagnosis, measure, attribution decision must be reproducible from raw source data.

---

## Common Pitfalls

- Using a stale CMS-HCC version (e.g., v22 / v24) after the model has been retired or phased out.
- Treating problem-list codes as billable evidence for HCC capture.
- Mismatching attribution windows between risk adjustment, quality measurement, and TCOC.
- Replicating CMS benchmark math approximately rather than exactly — even small drift undermines projection accuracy.
- Encoding MIPS category weights from memory without updating each performance year.
- Ignoring **Part C/D dual reporting** and **dual-eligible exclusions** for the wrong programs.
- Treating Star cut points as fixed — they shift each year, and tukey-outlier methodology has changed.
- Building "leakage" reports without provider-roster freshness — a provider who switched TINs looks like leakage when they haven't moved.

---

## Task-Specific Questions

1. Which contract type(s) are in scope — MSSP (which Basic track or Enhanced?), ACO REACH (Professional or Global), commercial shared savings, MA delegated capitation, bundled payments (BPCI-A / CJR), or specialty risk (CKCC / EOM)? Each has different attribution, benchmark, and risk-adjustment rules.
2. What attribution method does the contract use — prospective PCP, retrospective claims-based plurality, voluntary alignment, panel-list / EHR-based, or hybrid? Which look-back window and which qualifying-visit code set?
3. Which quality measure stack must the platform produce — HEDIS (which year), eCQMs (which CMS PI / Quality set), MIPS CQMs, CAHPS, HOS, Star Ratings measures, or a commercial proprietary set? Identify both reporting submission paths (QRDA I/III, registry, claims) and internal performance management needs.
4. What is the HCC / RAF target — current population average RAF, desired RAF, and which model is in scope (CMS-HCC v24, v28 phase-in, HHS-HCC, RxHCC, Medicaid CDPS, or a commercial proprietary model)?
5. What data sources are available — claims (837/835), encounter data submissions (EDPS), eligibility (834), FHIR clinical, HL7 v2, CDA, ADT, supplemental HEDIS data, SDoH, pharmacy (NCPDP), labs (LOINC)? Are there gaps in any of these for the population in scope?
6. What is the current measured performance baseline — savings/loss vs. benchmark, quality score / Star rating, denial / leakage rate, gap-closure rate? Is there a prior year's official CMS / payer settlement to anchor against?
7. What is the engineering position — building a VBC platform from scratch, replacing a vendor (Innovaccer, Arcadia, Health Catalyst, Cotiviti, etc.), augmenting an EHR registry, or integrating with an existing data warehouse?

---

## Related Skills

- **medical-coding**: ICD-10-CM specificity, problem-list vs. encounter coding, MEAT, and CAC workflows that feed risk adjustment.
- **billing-claims**: 837/835 data that underlies attribution, TCOC, and benchmark calculations.
- **prior-authorization**: delegated UM and risk-arrangement PA in capitated and full-risk models.
- **fhir-integration**: FHIR Bundle, CQL, Measure, Library, Group resources used for eCQM and Da Vinci Risk Based Contracts (RBC).
- **population-health-analytics**: care-gap operations, panel management, and outreach workflows downstream of VBC analytics.
- **clinical-decision-support**: point-of-care surfacing of care gaps, suspect HCCs, and risk-stratified outreach.
- **healthcare-context**: organization role, programs in scope, and risk position that scope every recommendation here.
