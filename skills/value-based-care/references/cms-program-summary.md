# CMS Value-Based Care Program Summary

Reference for engineers building VBC analytics platforms. Summarizes the major CMS Innovation Center (CMMI) and traditional CMS VBP programs by population, model type, attribution method, quality measure set, and payment mechanism — at a high level. Does **not** enumerate specific savings rates, benchmark formulas, performance-year parameters, or effective dates — those change with each rulemaking cycle.

> Verify all program parameters against the **current CMS program guide** (MSSP page, ACO REACH page, Innovation Center page, QPP page) and the **latest final rule / participation agreement** before encoding rules. Program rules change every performance year.

## Contents

- Programs overview table
- MSSP (Medicare Shared Savings Program)
- ACO REACH
- PCF (Primary Care First)
- AHEAD
- CKCC / KCC
- BPCI-A
- CJR
- EOM (Enhancing Oncology Model)
- MA Star Ratings and MA risk
- Commercial VBC and Medicaid VBP
- Engineering implications
- Cross-program coordination

---

## Programs Overview Table

| Program | Population | Model type | Attribution | Risk position | Quality |
|---|---|---|---|---|---|
| **MSSP** | Medicare FFS | Shared savings (Basic A–E + Enhanced — verify current tracks) | Retrospective claims-based plurality (verify current methodology) + voluntary alignment | Upside (BASIC A) → two-sided (BASIC D/E, Enhanced) | MSSP quality measure set |
| **ACO REACH** | Medicare FFS | Capitation (Professional or Global) — two-sided | Prospective alignment + voluntary alignment | Two-sided | ACO REACH quality measures + health equity benchmark adjustment |
| **PCF** | Medicare FFS | Population-based primary care payment + performance | Voluntary participant practice with attributed beneficiaries | Performance-based adjustment | PCF performance measure set |
| **AHEAD** | Medicare FFS + multi-payer | State-led all-payer TCOC | State-defined | State-defined | State-defined |
| **CKCC / KCC** | ESRD / late-stage CKD Medicare FFS | Capitation or shared savings | Disease-specific attribution | Varies by option | Kidney-care-specific measures |
| **BPCI-A** | Medicare FFS clinical episodes | Retrospective episode bundle | Episode-trigger driven (anchor stay or anchor procedure) | Two-sided on episode TCOC | Quality measures tied to episode |
| **CJR** | Medicare FFS lower-extremity joint replacement | Retrospective episode bundle | MSA-defined and procedure-trigger | Two-sided | LEJR-specific quality measures |
| **EOM** | Oncology Medicare FFS | Episode-based + MEOS PBPM | Episode-trigger | Two-sided | Oncology-specific measures |
| **MA Star Ratings** | Medicare Advantage | Capitation + bonus payments | MA enrollment | Quality-driven bonus | HEDIS + CAHPS + HOS + operational |

> Verify each row against the current CMS program guide. The "track structure" of MSSP, the participation options in ACO REACH, the active scope of BPCI-A and CJR, and the current scope of EOM all change.

---

## MSSP (Medicare Shared Savings Program)

### What it is

The permanent CMS shared-savings program for **Medicare FFS** providers organized as Accountable Care Organizations (ACOs).

### Population

Medicare FFS beneficiaries attributed to participating ACOs.

### Tracks

MSSP has multiple tracks with differing risk levels. Historically these have included **Basic** (Levels A through E, glide path from upside-only to two-sided) and **Enhanced** (highest risk and reward).

> Verify the current track structure, eligibility, and glide path against the current MSSP final rule. CMS has restructured tracks multiple times.

### Attribution

Historically retrospective claims-based plurality (the PCP who delivered the plurality of qualifying primary-care visits over a look-back) plus voluntary alignment.

> Verify current attribution methodology including the qualifying-visit code list and look-back window.

### Quality

CMS-published MSSP quality measure set, increasingly delivered via APP (APM Performance Pathway) for some tracks.

> Verify current MSSP quality measure set and submission pathway.

### Risk adjustment

CMS-HCC (current model version — verify v24/v28 phase-in).

### Benchmark

Historical + regional blend; rebased on rolling cycles. Methodology is detailed in the MSSP rule.

> Verify the current MSSP benchmark methodology — it has been restructured multiple times.

---

## ACO REACH

### What it is

CMS Innovation Center capitation-based ACO model focused on **Medicare FFS** beneficiaries. Successor to the Global and Professional Direct Contracting (GPDC) model.

### Population

Medicare FFS beneficiaries aligned to a REACH ACO.

### Participation options

Historically **Professional** and **Global** options with differing risk and capitation arrangements; standard, new-entrant, and high-needs ACO types.

> Verify current participation options, type structure, and operational status of ACO REACH against the current CMMI program page. Innovation Center models have defined performance-year windows.

### Attribution

Prospective alignment (with prospective + voluntary alignment options).

> Verify current attribution rules.

### Quality

ACO REACH quality measure set with health-equity-related elements.

### Risk adjustment

CMS-HCC with model coefficients caps and adjustments specific to REACH.

> Verify current ACO REACH risk-adjustment rules — caps and methodology have been modified.

### Benchmark

Includes a **health equity benchmark adjustment** to address underserved populations. Specific formulation has evolved.

> Verify current benchmark methodology and equity adjustment.

---

## PCF (Primary Care First)

### What it is

CMMI primary-care-focused model with population-based payments and performance-based adjustments for participating primary care practices.

### Population

Medicare FFS beneficiaries attributed to participating practices, with multi-payer alignment encouraged where state participation exists.

### Payment

Population-based payment per attributed beneficiary plus performance-based adjustment.

### Quality

PCF performance measures focused on primary care quality and continuity.

> Verify PCF current scope, performance year, and measure set.

---

## AHEAD

### What it is

States Advancing All-Payer Health Equity Approaches and Development (AHEAD) — state-led all-payer total-cost-of-care model.

### Population

All payers within participating states (Medicare FFS + Medicaid + commercial).

### Mechanism

States set state-level cost growth targets and align participating payers and providers.

> Verify current participating states, target structure, and operational status.

---

## CKCC / KCC

### What it is

Kidney Care Choices — capitation and shared-savings options for providers caring for ESRD and late-stage CKD beneficiaries.

### Population

Medicare FFS beneficiaries with ESRD or specified stages of CKD.

### Options

CKCC (CKD Contracting Entities) with multiple risk options.

> Verify current CKCC options, risk structure, and operational status.

---

## BPCI-A (Bundled Payments for Care Improvement Advanced)

### What it is

CMMI episode-based bundled payment model — providers take retrospective risk on the total cost of a clinical episode from anchor through the post-anchor window.

### Population

Medicare FFS beneficiaries who trigger a participating clinical episode.

### Mechanism

Retrospective episode reconciliation against a target price. Multiple clinical episode types.

> Verify current model year, participant scope, clinical episode list, and target-price methodology.

---

## CJR (Comprehensive Care for Joint Replacement)

### What it is

Mandatory episode-based bundle for **lower extremity joint replacement (LEJR)** in selected metropolitan statistical areas (MSAs).

### Population

Medicare FFS beneficiaries undergoing LEJR in participating MSAs.

> Verify current participating MSA list and operational status.

---

## EOM (Enhancing Oncology Model)

### What it is

CMMI oncology episode model, successor to the Oncology Care Model (OCM, which ended). Episode-based payment for oncology care with monthly enhanced oncology services (MEOS) per-beneficiary-per-month payments.

### Population

Medicare FFS beneficiaries undergoing chemotherapy for specified cancer types.

> Verify current EOM scope, performance year, and cancer-type inclusion list.

---

## MA Star Ratings and MA Risk

### What it is

Medicare Advantage organizations are paid a **risk-adjusted capitation** for each enrolled member. **Star Ratings** drive **bonus payments** and **rebate retention** that fund supplemental benefits.

### Mechanism

- Capitation = base rate × RAF × geographic factor × adjustments.
- Star Rating = composite weighted score across HEDIS, CAHPS, HOS, and operational measures.
- Bonus and rebate percentages depend on Star Rating tier.

### Risk adjustment

CMS-HCC (with phase-in of v28). Encounter-data-based; RxHCC for Part D.

### Quality

HEDIS + CAHPS + HOS + operational measures (call center, complaints, appeals, etc.).

> Verify current Star Rating measure list, weights, cut points, and Tukey outlier methodology — these change annually.

---

## Commercial VBC and Medicaid VBP

### Commercial VBC

Commercial VBC contracts use the same building blocks (attribution, benchmark, risk adjustment, quality) but rules are **contract-by-contract**. There is no single national methodology. Engineers must read each contract's:

- Attribution method and look-back
- Benchmark methodology
- Risk adjustment model (often commercial proprietary or HHS-HCC for ACA-aligned)
- Quality measure set (often HEDIS-aligned)
- Settlement timing and reconciliation rules

### Medicaid VBP

State Medicaid VBP arrangements are state-by-state, often built on top of Medicaid managed care contracts. Risk adjustment may use **CDPS / CDPS+Rx** or a proprietary model. Quality often uses **Medicaid Adult and Child Core Sets** and state-specific measures.

> Verify each commercial / Medicaid VBP contract's specifics with the contracting parties.

---

## Engineering Implications

### Multi-program platform

A VBC platform serving an organization participating in multiple programs (e.g., MSSP + commercial ACOs + MA delegated risk) must support **per-program** attribution, risk adjustment, quality, and benchmark engines. Sharing infrastructure where possible while pinning per-program rules.

### Program parameter versioning

Every program parameter (track structure, savings rate, benchmark formula, quality measure list, RA model version) is **performance-year-specific**. Store program parameters as effective-dated reference data; never hard-code.

### Exact benchmark replication

For programs where the platform projects performance against benchmark (most ACO programs), the benchmark math must be replicated **exactly** as CMS computes it, not approximately. Even small drift produces unreliable projections.

### Encounter-data submission

Programs with risk adjustment (MA, ACO REACH, REACH-style models) require encounter data submission per CMS rules. Verify current submission system (RAPS is retired; EDPS is the current Encounter Data Processing System for MA — verify current name and timelines).

### Audit reproducibility

Every quality numerator, attribution decision, RAF calculation, and TCOC computation must be reproducible from raw source data. CMS, RADV, and contract audits demand this.

---

## Cross-Program Coordination

### Overlap rules

CMS sets **overlap rules** between programs (e.g., MSSP vs. ACO REACH vs. BPCI-A for the same beneficiary or the same episode). The overlap rules change with each rule cycle.

> Verify current overlap rules against the applicable program guidance before encoding precedence logic.

### QPP / MIPS interaction

Clinicians participating sufficiently in **Advanced APMs** (e.g., qualifying ACO REACH or MSSP tracks) may earn the **Qualifying APM Participant (QP)** designation, exempting them from MIPS for that performance year.

> Verify current Advanced APM list and QP thresholds.

### Dual-eligible considerations

Dual-eligible (Medicare + Medicaid) and partial-dual beneficiaries have specific inclusion/exclusion rules across programs. Hospice election, ESRD coverage period, and skilled nursing facility status also affect program inclusion.

> Verify current dual-eligible and exclusion rules per program.
