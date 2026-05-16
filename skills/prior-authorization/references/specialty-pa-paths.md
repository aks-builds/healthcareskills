# Specialty PA Paths

Reference for engineers building prior authorization workflows that touch specialty drug, imaging, surgical, DME, and other PA-heavy service categories. Describes the typical intermediaries, channels, and clinical-criteria patterns at a high level. Does not enumerate specific payer rules, drug-by-drug PA criteria, or RBM contract terms — those change frequently.

> Verify current intermediary relationships, channels, and clinical criteria with the specific payer/PBM/UMO/RBM before building integrations.

## Contents

- Why specialty PA is structurally different
- Specialty pharmacy paths
- High-cost imaging paths
- Surgical PA paths
- DME paths
- Behavioral health paths
- Post-acute paths
- Genetic / molecular testing paths
- Engineering implications

---

## Why Specialty PA Is Structurally Different

Most healthcare PA volume concentrates in a handful of service categories where:

- The cost or risk per unit of service is high enough to justify per-case review.
- Clinical criteria are well-defined (often supported by guideline-derived rule sets).
- Payers delegate to specialized **utilization management organizations (UMOs)** or **benefit managers** rather than running the review in-house.

The delegation pattern means engineers building PA software must integrate with the **delegated entity**, not just the contracted payer. The same payer may delegate different service lines to different vendors.

---

## Specialty Pharmacy Paths

Specialty pharmacy covers high-cost biologics, oncology agents, gene/cell therapy, GLP-1s, and other complex therapies — distinct from retail Rx for chronic conditions.

### Intermediaries

- **Surescripts** — operates a switch network connecting EHRs, e-prescribing modules, pharmacies, and PBMs. ePA capabilities ride on top of e-prescribing.
- **CoverMyMeds** (Optum / RelayHealth heritage) — historically the dominant ePA intermediary; abstracts per-PBM ePA quirks.
- Both are commonly used; many EHRs support one or both.

### Channels

- **NCPDP SCRIPT ePA** — `PARequest`, `PAResponse`, `PAAppealRequest`, `PACancelRequest` transactions integrated with e-prescribing.
- **PBM web portals** — manual submission for cases that fall outside ePA-supported question sets.
- **Specialty pharmacy hubs** (manufacturer-sponsored hubs and dispensing specialty pharmacies) — often handle PA on the patient's behalf as part of patient-onboarding services.

### Typical workflow

1. Prescriber initiates an Rx in the EHR.
2. E-prescribing module performs **real-time formulary and benefit (F&B) check** against the member's PBM — surfaces formulary tier and PA requirement.
3. If PA required, the system sends a `PARequest` (via Surescripts or CoverMyMeds) with the payer-specific question set.
4. PBM returns approval, denial, or additional-info request.
5. Approval information flows back to the EHR and to the dispensing pharmacy.

### Engineering notes

- **Pharmacy ePA is faster and cheaper than medical PA** for drugs that can be billed under the pharmacy benefit — don't route every drug through medical PA.
- The line between **pharmacy benefit** (NCPDP) and **medical benefit** (X12 / FHIR) drug billing depends on the drug, site of administration, and plan design. Some specialty drugs are dual-benefit and need workflow logic to route correctly.
- **Patient-onboarding hubs** (manufacturer-sponsored) often have their own PA workflows that operate alongside the prescriber's; coordination is required to avoid duplicate submissions.

> Verify PBM-specific ePA support, hub relationships, and benefit routing per drug with the payer / PBM / manufacturer.

---

## High-Cost Imaging Paths

High-cost imaging (MRI, CT, PET, nuclear cardiology) is almost universally PA-required for commercial and MA, and the review is typically **delegated to a Radiology Benefit Manager (RBM)**.

### Common RBMs

- **eviCore** (Cigna / Evernorth)
- **Carelon Medical Benefits Management** (Anthem heritage; formerly AIM Specialty Health)
- **AIM Specialty Health** (historical brand; rolled into Carelon)
- Payer-internal UM teams for payers that handle imaging in-house

### Channels

- **RBM web portals** — dominant channel.
- **RBM-specific APIs** — increasingly available for high-volume submitters.
- **X12 278** — supported by some RBMs.
- **Da Vinci CRD/DTR/PAS** — emerging at some RBMs, particularly for impacted-payer scope under CMS-0057-F.

### Clinical criteria

- **ACR Appropriateness Criteria** — American College of Radiology guidance that many RBM rule sets reference.
- **NCCN** — for oncology imaging.
- **RBM-proprietary** — typically derived from published guidelines + payer-specific rules.

### Engineering notes

- The RBM is often **the same across multiple payers** — once integrated, the integration is reusable.
- The RBM authorization decision may or may not flow back to the payer automatically — verify the payer's claim-side validation method.
- **Concurrent / contrast-vs-non-contrast** changes mid-procedure can void the PA — workflow must handle escalation.

> Verify per-payer RBM delegation, per-RBM channels, and current clinical criteria with each entity.

---

## Surgical PA Paths

Surgical PA covers orthopedic (joint replacement, spine, sports medicine), bariatric, cardiac, ENT, ophthalmology, and other surgical service lines.

### Channels

- Payer or UMO web portals — dominant.
- Fax forms — still common for many surgical specialties.
- X12 278 — supported by some payers.
- Da Vinci PAS — emerging for impacted-payer scope.

### Clinical criteria

- **InterQual** (Change Healthcare / Optum) — widely licensed UM criteria sets.
- **MCG** (Hearst) — alternative widely-licensed UM criteria sets.
- **Payer-proprietary medical policies** — typically published online by the payer.
- **Specialty society guidelines** (AAOS, NASS, ASMBS, etc.) — referenced by payer policies.

### Engineering notes

- Surgical PA documentation is often extensive (imaging, conservative-treatment trial, BMI history, specialist consults). Plan attachment workflow.
- **Bilateral procedures** require careful PA coding (units, modifiers RT/LT/50) to avoid mismatch with the eventual claim.
- **Multi-step procedures** (e.g., spinal fusion with multiple levels, staged reconstruction) may require multiple PAs.

> Verify per-payer surgical PA channels and clinical criteria.

---

## DME Paths

DME (durable medical equipment) PA covers power mobility, CPAP/BiPAP, oxygen, infusion pumps, hospital beds, and similar items.

### Channels

- **DME MAC portals and fax** for Medicare FFS.
- Payer or DME-vendor portals for commercial / MA.
- X12 278 for some payers.
- CMS-mandated DME PA pilot programs have specific submission paths — verify current programs.

### Clinical criteria

- **Medicare DME LCDs and policy articles** — detailed coverage criteria with required documentation elements.
- **Payer-proprietary medical policies**.

### Engineering notes

- **Detailed Written Order (DWO)** and **face-to-face encounter documentation** are required for many DME items under Medicare — workflow must capture and attach.
- **Refill / replacement** has separate rules from initial provision.
- **Continued-use documentation** is often required for ongoing rental items.

> Verify current Medicare DME LCDs and payer-specific DME criteria.

---

## Behavioral Health Paths

Behavioral health PA covers inpatient psych, residential, IOP/PHP, ABA, TMS, ECT, neuropsych testing, and other services.

### Channels

- Payer or behavioral-health-carve-out UMO portals (e.g., Optum Behavioral Health, Magellan, Carelon Behavioral Health).
- Fax.
- Concurrent review for inpatient typically by phone or portal.

### Clinical criteria

- **InterQual Behavioral Health**
- **MCG Behavioral Health Care**
- **ASAM** for substance use disorder
- **LOCUS / CALOCUS** for level-of-care
- Payer-proprietary

### Engineering notes

- **Continued-stay / concurrent review** for inpatient and residential is its own state machine — daily or every-few-days review with shifting clinical criteria.
- **Parity laws** (federal MHPAEA and state parity statutes) constrain how behavioral health UM can differ from medical-surgical UM — denial documentation must hold up to parity audit.
- Behavioral health carve-outs are common — the payer's name on the card may not be who reviews behavioral PA.

> Verify carve-out delegations and parity-relevant operational requirements per payer and per state.

---

## Post-Acute Paths

Post-acute PA covers skilled nursing facility (SNF), home health agency (HHA), inpatient rehab facility (IRF), and long-term acute care hospital (LTACH) admissions and concurrent days.

### Channels

- Payer or post-acute UMO portals; phone for urgent admissions.
- **CMS-mandated post-acute PA programs** (e.g., HHA / IRF / LTACH demonstrations) — verify current scope.

### Clinical criteria

- **CMS Conditions of Participation and coverage criteria** for the specific post-acute setting.
- **InterQual / MCG post-acute** criteria sets.
- Payer-proprietary.

### Engineering notes

- **Discharge planning workflow** must integrate with PA — same-day SNF admission needs same-day PA capability.
- **Concurrent review** for inpatient rehab is intensive — daily review with cliff edges if criteria not met.
- **MA two-midnight and observation policies** intersect with post-acute PA — verify current Medicare and MA guidance.

> Verify current CMS post-acute coverage rules and payer-specific PA programs.

---

## Genetic / Molecular Testing Paths

Genetic and molecular testing PA covers oncology biomarker panels, pharmacogenomics, hereditary cancer testing, prenatal screening, and similar tests.

### Channels

- **MolDX** (Palmetto / Noridian) — Medicare contractor lab-benefit-management program for molecular diagnostics; coverage policy and registration of new test codes.
- Payer benefit-management programs (e.g., Avalon Healthcare Solutions, Concert Genetics, eviCore Lab).
- Payer-direct portals.

### Clinical criteria

- **MolDX LCDs and articles** for Medicare.
- **NCCN biomarker guidelines** for oncology.
- Payer-proprietary policies.

### Engineering notes

- **CPT PLA (Proprietary Laboratory Analyses) codes** and **HCPCS lab codes** evolve rapidly — keep code mappings current.
- **Z-codes / DEX Z-codes** are used in MolDX to identify specific tests; verify current use.
- Documentation requirements (family history, prior testing, clinical indication) are extensive.

> Verify current MolDX coverage, payer benefit-management programs, and code mappings.

---

## Engineering Implications

### Multi-channel orchestration

A real PA system handles all of the above categories in parallel. Build a channel router keyed on **payer × service category × member plan** that selects the correct submission path (intermediary, channel, format).

### Clinical-criteria abstraction

Different criteria sets (InterQual, MCG, ACR, NCCN, MolDX, payer-proprietary) have different shapes. Build a **criteria adapter layer** that normalizes the question-set + evidence requirements to a common internal representation.

### Decision-capture consistency

Regardless of channel, the decision returned must be normalized to the same structured authorization object (number, valid range, codes, units, conditions). See `da-vinci-pa-stack.md` for the structure used in PAS.

### Aging and TAT

Each category has its own typical TAT. Build aging buckets per category and surface breach risk to the PA team early.

### Coordination with claims

The PA number lands on the claim. Mismatch between PA-authorized codes/units/providers and billed codes/units/providers is a major denial source. See `billing-claims` for the claim-side validation.

> Verify all intermediary, channel, and clinical-criteria specifics with the relevant entity before building integrations.
