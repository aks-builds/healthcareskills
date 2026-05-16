# US Medical Code Set Overview

Reference for engineers building software that touches US medical coding systems. Each entry covers purpose, owner, licensing posture, and update cadence. Every value here is at high level — verify current revision against the authoritative source before encoding.

> Verify current revision against the maintainer's published release notes each cycle. Do not freeze tables to a single year in production code.

## Contents

- ICD-10-CM (diagnoses)
- ICD-10-PCS (inpatient procedures)
- CPT (Current Procedural Terminology)
- HCPCS Level II
- CDT (dental)
- NDC (drug product identifier)
- MS-DRG (inpatient grouping)
- APC (outpatient grouping)
- Cross-cutting concerns
- Where to source the data

---

## ICD-10-CM (Diagnoses)

**Purpose**: classify diagnoses, signs/symptoms, and factors influencing health on US claims (all settings).

**Maintainer**: National Center for Health Statistics (NCHS, part of CDC) for content; CMS co-maintains.

**License posture**: public domain in the US.

**Update cadence**: annual major release effective **Oct 1**; mid-year addenda/errata may publish in **April** (verify current cadence with CDC/CMS).

**Structure**: 3-7 character alphanumeric. 7th-character extensions used in injury, OB, and external-cause chapters. Placeholder `X` fills positions when a 7th character is required but no 4th-6th character applies.

**Engineering notes**:
- Validity is **date-of-service-dependent**. Codes are added, revised, and deleted each fiscal year.
- Claims/X12 transmit codes **without** the decimal; documentation typically shows the decimal.
- Excludes1 (mutually exclusive) vs. Excludes2 (both can be coded) is a coding-rule, not a syntactic property of the code itself.
- The **ICD-10-CM Official Guidelines for Coding and Reporting** are published annually by CDC/CMS and update each fiscal year.

> Verify current revision against the CDC NCHS / CMS ICD-10 release.

---

## ICD-10-PCS (Inpatient Procedures)

**Purpose**: classify inpatient procedures on US 837I claims.

**Maintainer**: CMS.

**License posture**: public domain in the US.

**Update cadence**: annual release effective **Oct 1**.

**Structure**: 7-character alphanumeric — Section, Body System, Root Operation, Body Part, Approach, Device, Qualifier. Multi-axial composition rather than lookup.

**Engineering notes**:
- Used **only** on inpatient hospital claims. Outpatient procedures use CPT/HCPCS.
- Autocoders must compose codes from structured op-report fields, not phrase lookup.
- Annual changes affect a small percentage of codes but the affected stays can be high-volume.

> Verify current revision against the CMS ICD-10-PCS release.

---

## CPT (Current Procedural Terminology)

**Purpose**: procedures and services on US outpatient and professional claims.

**Maintainer**: American Medical Association (AMA).

**License posture**: **licensed**. Products that display, derive, or distribute CPT codes need an AMA license. Treat as licensed content in the build/release pipeline.

**Update cadence**: annual release effective **Jan 1**, with **quarterly errata** and **PLA (Proprietary Laboratory Analyses) codes** released quarterly.

**Structure**:
- **Category I** — 5-digit numeric standard procedures and services.
- **Category II** — `XXXXF` optional performance-measurement tracking codes (do not bill alone).
- **Category III** — `XXXXT` emerging-tech tracking codes.

**Engineering notes**:
- License the data; do not scrape.
- Versioning: separate the annual release from quarterly errata in your reference data.
- Modifiers (numeric, 2-character) attach to CPT codes and affect payment.

> Verify current revision against the AMA CPT release and licensing terms.

---

## HCPCS Level II

**Purpose**: supplies, DME, drugs administered other than oral, ambulance, orthotics/prosthetics, and other items not in CPT.

**Maintainer**: CMS.

**License posture**: public domain in the US.

**Update cadence**: **quarterly** updates.

**Structure**: 1 letter + 4 digits. Common letter ranges:
- **A** transport, medical/surgical supplies
- **B** enteral/parenteral
- **C** hospital outpatient
- **E** DME
- **G** temporary procedures / professional services
- **J** drugs administered other than oral (J-codes)
- **K** DMERC temporary
- **L** orthotic/prosthetic
- **Q** temporary
- **S** commercial temporary
- **T** Medicaid

**Engineering notes**:
- **J-codes** are the billable unit for separately payable Part B drugs. Claims often carry both the NDC (product identity) and the J-code with units that differ — compute the J-code unit from administered milligrams.
- Modifiers are typically alphanumeric (e.g., RT/LT, XE/XS/XP/XU, GT/95 telehealth historical).

> Verify current revision against the CMS HCPCS quarterly release.

---

## CDT (Current Dental Terminology)

**Purpose**: dental procedures on 837D dental claims.

**Maintainer**: American Dental Association (ADA).

**License posture**: **licensed** by ADA.

**Update cadence**: annual release.

**Structure**: `Dxxxx`.

**Engineering notes**:
- License the data from ADA for any product that displays or distributes CDT.
- Used in dental claim assembly; coordinates with payer dental fee schedules.

> Verify current revision against the ADA CDT release.

---

## NDC (National Drug Code)

**Purpose**: identify manufactured drug products at the labeler-product-package level.

**Maintainer**: US Food and Drug Administration (FDA).

**License posture**: public.

**Update cadence**: continuous (manufacturers list and de-list products on an ongoing basis).

**Structure**: three segments — labeler (4-5 digits), product (3-4 digits), package (1-2 digits). Source is 10 digits across the three segments; claims typically transmit an **11-digit normalized form** (`5-4-2`).

**Engineering notes**:
- The same drug at the same strength may have hundreds of NDCs across manufacturers, repackagers, and package sizes.
- **RxNorm** normalizes across NDCs to a clinical drug concept (ingredient + strength + dose form).
- Claims-side: store the raw NDC and the normalized 11-digit form; preserve original packaging info for J-code unit calculations and audit.

> Verify current data via the FDA National Drug Code Directory.

---

## MS-DRG (Medicare Severity DRG)

**Purpose**: group inpatient stays into severity-adjusted payment categories under the Medicare Inpatient Prospective Payment System (IPPS).

**Maintainer**: CMS.

**License posture**: public methodology; the **grouper software** (3M / Solventum, Optum) is licensed.

**Update cadence**: annual release with the **IPPS final rule**, effective **Oct 1**.

**Engineering notes**:
- Groups on PDx, SDx, procedures (ICD-10-PCS), discharge disposition, sex, age, and presence of MCCs/CCs.
- **AP-DRG** and **APR-DRG** (3M) are alternative DRG systems used by some state Medicaid programs and commercial payers — do not assume MS-DRG everywhere.

> Verify current revision against the CMS IPPS final rule and grouper version notes.

---

## APC (Ambulatory Payment Classification)

**Purpose**: group hospital outpatient claim lines for payment under the Medicare Outpatient Prospective Payment System (OPPS).

**Maintainer**: CMS.

**License posture**: public.

**Update cadence**: **quarterly** with the OPPS Addendum B; annual final rule.

**Engineering notes**:
- **Composite APCs** bundle related services; **Comprehensive APCs (C-APCs)** pay a single rate for a primary procedure plus all adjunctive same-date services.
- **Status indicators** on each HCPCS/CPT line determine payment behavior — pricing engines must consult Addendum B for current values.

> Verify current revision against the quarterly OPPS Addendum B.

---

## Cross-Cutting Concerns

### Date-of-service validity

Every code system above changes over time. Pricing, scrubbing, and analytics must apply the **version effective on the date of service**, not the version current at the time of processing.

### Decimal vs. no-decimal

ICD-10-CM stores in code form with implied position; X12 claims transmit **without** decimal; documentation typically shows it. Choose one canonical form internally and translate at the boundary.

### Modifiers

CPT modifiers are numeric (e.g., `25`, `59`, `76`, `77`, `91`); HCPCS modifiers are alphanumeric (e.g., `RT`, `LT`, `GT`, `95`, `XE`, `XS`, `XP`, `XU`). Modifier rules change — do not hard-code semantics from memory; build a configurable modifier table sourced from current CPT/HCPCS data each year.

> Verify current modifier definitions and payer-specific rules against AMA CPT, CMS HCPCS, and payer companion guides.

### Terminology services alongside billing codes

- **SNOMED CT** — EHR-side problem-list and structured-data terminology.
- **LOINC** — laboratory and clinical observations.
- **RxNorm** — clinical-drug concept normalization across NDCs.

CAC systems must reconcile clinical terminology with billing codes; the mapping is many-to-many and not lossless.

---

## Where to Source the Data

| Code system | Primary source | Notes |
|---|---|---|
| ICD-10-CM | CDC NCHS / CMS | Public domain; annual + errata |
| ICD-10-PCS | CMS | Public domain; annual |
| CPT | AMA | License required |
| HCPCS Level II | CMS | Public domain; quarterly |
| CDT | ADA | License required |
| NDC | FDA NDC Directory | Public; continuous |
| MS-DRG (methodology) | CMS IPPS Final Rule | Annual |
| MS-DRG grouper | 3M/Solventum, Optum | License required |
| APC | CMS OPPS Addendum B | Public; quarterly |
| NCCI PTP/MUE | CMS NCCI | Public; quarterly |
| LCD/NCD | CMS Medicare Coverage Database | Public; rolling |

> Build a release-management process that pulls each source on its native cadence and stamps the reference data with effective dates. Treat any update as a release event requiring re-validation against your gold set.
