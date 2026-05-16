# Care Gap Catalogs

Reference for the major care-gap measure sets used in population health: HEDIS, eCQMs, MIPS, CMS Stars, and state Medicaid. Definitions, denominator/numerator/exclusion logic, and value sets change each measurement year — **always verify against the current measurement year specification published by NCQA, CMS, or the applicable state Medicaid program before implementing gap logic.**

## Contents

- HEDIS (NCQA)
- eCQMs (CMS)
- MIPS Quality
- CMS Star Ratings (Medicare Advantage)
- State Medicaid quality
- Custom organizational measures
- Measure categories
- Engineering pattern
- Verification expectations

---

## HEDIS (NCQA)

**Healthcare Effectiveness Data and Information Set** is NCQA's quality measure set, used by commercial health plans, Medicare Advantage, Medicaid managed care, and exchange plans.

- Refreshed annually in the **HEDIS Volume 2 Technical Specifications** (typically published in July for the following measurement year, with errata released throughout).
- Approximately 90+ measures across domains (verify current count).
- Used directly for plan accreditation (NCQA), CMS Star Ratings, and many state Medicaid quality programs.
- Supplemental data submission (SDS) and non-standard supplemental data (NSSD) rules govern how supplemental clinical data is accepted; verify current NCQA SDS rules.

Common HEDIS measure families (verify current measure list and acronyms):

- Cardiovascular (e.g., controlling blood pressure, statin therapy for cardiovascular disease).
- Diabetes (e.g., hemoglobin A1c control, eye exam, kidney evaluation).
- Cancer screening (e.g., breast, colorectal, cervical).
- Behavioral health (e.g., follow-up after ED visit for mental illness, antidepressant medication management).
- Pediatric (e.g., well-child visits, childhood immunization status, adolescent immunization status).
- Women's health (e.g., prenatal and postpartum care, contraceptive care).
- Medication safety (e.g., use of high-risk medications in older adults).
- Utilization (e.g., ambulatory care, hospital readmissions).

**HEDIS measures have specific licensing terms.** Implementations must reference the current NCQA Volume 2 spec; do not hard-code value sets from older measurement years.

---

## eCQMs (CMS)

**Electronic Clinical Quality Measures** are CMS-defined quality measures expressed in CQL (Clinical Quality Language) with value sets on VSAC, designed for direct computation from certified EHR data.

- Used in **Promoting Interoperability** (hospital and clinician), **MIPS Quality**, **Medicare Shared Savings Program (MSSP)**, **Medicaid Promoting Interoperability**, and other CMS programs.
- Each measure has a CMS ID (e.g., CMS122), a measure ID, a version, and a CQL specification published on the CMS Measures Inventory Tool (MIT) and eCQI Resource Center.
- **Value sets are published on VSAC** with measure-year-specific stewardship.
- Measure specifications change annually; the **eCQM Annual Update Pre-Publication Document** lists changes.

Examples (verify current spec and measure number):

- Diabetes hemoglobin A1c poor control.
- Controlling high blood pressure.
- Closing the referral loop.
- Preventive care and screening (e.g., colorectal cancer screening, breast cancer screening).
- Depression screening and follow-up.

Eligible clinicians and eligible hospitals report eCQMs through CMS reporting platforms; QRDA I (patient-level) and QRDA III (aggregate) are the standard submission formats.

---

## MIPS Quality

**Merit-based Incentive Payment System** Quality category (for eligible clinicians under the Quality Payment Program).

- A subset of MIPS Quality measures are eCQMs; others are claims-based or registry-based.
- Verify the current MIPS measure inventory and reporting requirements with CMS for each performance year.
- MIPS PI category (Promoting Interoperability) is separate from MIPS Quality but shares some measure overlap.

---

## CMS Star Ratings (Medicare Advantage)

CMS Star Ratings drive MA plan payment and consumer plan choice. Quality measures used in Stars include many HEDIS measures, CAHPS survey measures, HOS survey measures, and administrative measures.

- The **Star Ratings Technical Notes** (CMS, annual) specify measure weights, cut points, and computation rules for each measurement year.
- Cut points are determined retrospectively (Tukey outlier deletion and other rules); verify current methodology.
- Measure weights change periodically (e.g., patient experience weight changes).

---

## State Medicaid quality

State Medicaid managed care quality programs vary by state but commonly include:

- **CMS Adult Core Set** and **CMS Child Core Set** measures.
- HEDIS measures (most states require submission of a defined HEDIS measure subset).
- State-specific measures (e.g., specific maternal health, behavioral health, SDOH measures).

Verify the current State Medicaid Quality Strategy and managed care contract requirements for each state in scope. State 1115 waivers may also impose specific measure requirements.

---

## Custom organizational measures

Organizations often layer custom measures on top of standardized sets:

- Specialty registry measures (e.g., oncology, cardiology, behavioral health).
- Internal quality improvement measures.
- ACO-internal performance measures.
- Custom equity-stratified measures.

Treat custom measures with the same rigor as standardized measures — denominator / numerator / exclusion logic, value sets, version control, and audited rules engine.

---

## Measure categories

Independent of measure set, gaps fall into categories:

| Category | Examples |
|----------|----------|
| Preventive screening | Mammography, colorectal cancer screening, cervical cancer screening, depression screening |
| Immunization | Childhood, adolescent, adult, influenza, pneumococcal |
| Chronic disease control | A1c control, BP control, LDL where measured |
| Medication management | Statin adherence, high-risk medications in older adults, MTM completion |
| Care transitions | Follow-up after hospitalization for mental illness, post-discharge follow-up |
| Behavioral health | Depression remission, follow-up after ED visit for substance use |
| Maternal / pediatric | Prenatal and postpartum care, well-child visits |
| Equity stratification | Any measure stratified by race, ethnicity, language, payer, geography |

---

## Engineering pattern

A correct gap-closure pipeline:

1. **Express each measure** as denominator / numerator / exclusion logic, ideally using the published CQL (for eCQMs) or NCQA HEDIS spec.
2. **Bind value sets** to current VSAC (eCQMs) or NCQA Volume 2 (HEDIS) for the measurement year.
3. **Compute member-level gap status** nightly or weekly using an **audited rules engine** — not BI dashboard ad-hoc logic.
4. **Surface gaps** in clinician workflows (EHR inbox), care manager registries, and member-facing nudges.
5. **Capture closure** with structured evidence (immunization administered, A1c result, screening documented).
6. **Submit supplemental data** to payers per their SDS / NSSD rules for measures where the payer accepts supplemental clinical data.
7. **Audit and version-control** measure definitions; record measurement year for every computed result.

Common engineering pitfalls:

- Hard-coding value sets from an older measurement year; gaps disappear or appear spuriously when codes change.
- Building gap logic in BI rather than a dedicated rules engine; non-auditable, brittle.
- Conflating concurrent and look-back computation windows.
- Failing to apply exclusion criteria correctly (denominator exceptions).
- Missing supplemental data submission deadlines; gaps closed but never credited.

---

## Verification expectations

Before each measurement year cycle:

1. Pull the current HEDIS Volume 2 Technical Specifications from NCQA.
2. Pull current eCQM specifications from the CMS eCQI Resource Center and value sets from VSAC.
3. Pull the current MIPS measure inventory from CMS.
4. Pull the current CMS Star Ratings Technical Notes.
5. Pull current State Medicaid quality measure requirements for each state in scope.
6. Verify supplemental data submission (SDS) rules with each payer.
7. Refresh value sets in the rules engine and run regression tests against the prior measurement year.

Always operate against current measurement-year specifications. Measure definitions, denominators, exclusions, and value sets change every year.
