# HCC Risk Adjustment

Reference for engineers building risk-adjustment modules and HCC capture workflows. Describes CMS-HCC mechanics at a conceptual level, RAF calculation, MEAT documentation criteria, and RADV audit posture. Does **not** enumerate specific HCC IDs, ICD-to-HCC mappings, coefficient values, or model-year blends — those are CMS-published artifacts that must be loaded from the current CMS Risk Adjustment release.

> Verify HCC IDs, ICD-to-HCC mappings, coefficients, and model-version blend percentages against the **current CMS Risk Adjustment release** for the applicable payment year. Do not hard-code any of these values from memory.

## Contents

- What HCC risk adjustment is
- CMS-HCC model versions and v28 phase-in
- Other risk-adjustment models
- HCC mechanics
- RAF calculation concept
- MEAT criteria for documentation
- HCC capture workflow
- RADV (Risk Adjustment Data Validation)
- Engineering patterns
- Common pitfalls

---

## What HCC Risk Adjustment Is

HCC risk adjustment translates a member's documented diagnoses (and demographics) into a **Risk Adjustment Factor (RAF)** that adjusts payments to plans and providers caring for that member.

The economic intent: plans and providers caring for sicker populations receive higher payments; plans and providers with healthier populations receive lower payments. Without risk adjustment, capitation systems would incentivize cherry-picking and adverse selection.

The core model in US Medicare is **CMS-HCC** — Hierarchical Condition Categories maintained by CMS for Medicare Advantage and several CMS Innovation Center models.

---

## CMS-HCC Model Versions and v28 Phase-In

CMS updates the CMS-HCC model periodically. Major recent versions include:

- **v22** — earlier model version (historical reference).
- **v24** — model version used as a primary or blend basis through the v28 transition.
- **v28** — current major model version being phased in. v28 changed:
  - The HCC list (some conditions consolidated, some moved hierarchies)
  - The ICD-10-CM-to-HCC mapping (some codes no longer map to an HCC; some maps changed)
  - The coefficient values

CMS phases v28 in over multiple payment years, blending v24 and v28 RAFs by published percentages.

> Verify the current model version, the blend percentages for the applicable payment year, and any further updates against the **CMS Rate Announcement** and the **CMS Risk Adjustment release**.

> Do **not** assume the blend or the current version — read the current rate announcement.

---

## Other Risk-Adjustment Models

Different lines of business use different models. Verify the correct model for each population:

- **CMS-HCC** — Medicare Advantage, ACO REACH, and several CMMI models.
- **HHS-HCC** — ACA individual and small-group commercial risk adjustment.
- **RxHCC** — Medicare Part D drug risk adjustment.
- **CDPS / CDPS+Rx** — Medicaid risk-adjustment in many states.
- **Commercial proprietary** — DxCG, Johns Hopkins ACG, Verisk Sightlines, and others used by commercial payers and risk-bearing entities.

Each has its own coefficient set, model years, and operational rules.

> Verify the applicable model and its current release for the population in scope.

---

## HCC Mechanics

### Diagnosis sources

ICD-10-CM diagnoses on **accepted encounter types** (inpatient, outpatient, and professional under CMS encounter-data filtering rules) within a **payment-year measurement period** are eligible to contribute to RAF.

> Verify the current encounter-type filter (which place-of-service / bill-type / provider-type combinations contribute) and the measurement-period window against the current CMS Risk Adjustment guidance.

### Annual recapture

In CMS-HCC, **HCCs do not carry over year-to-year**. Each condition that should risk-adjust must be **assessed and documented** annually within the measurement period to be re-captured for the following payment year.

This drives a substantial operational workflow: Annual Wellness Visits (AWV), chart reviews, suspect-condition outreach, provider engagement, and clinician documentation prompts.

### Hierarchy

HCCs are organized into **hierarchies** within a clinical "family" (e.g., diabetes). When a member has codes mapping to multiple HCCs in the same family, the **more severe** HCC trumps the less severe — only the most severe HCC in the family contributes to RAF.

> Verify current hierarchy structure in the CMS model files.

### Interactions

Some HCC and demographic combinations carry **interaction coefficients** in addition to the individual coefficients. The CMS model software computes these interactions; engineers should not reimplement from memory.

---

## RAF Calculation Concept

At a conceptual level, RAF is roughly:

```
RAF = demographic_coefficient
    + sum(disease_coefficients for captured HCCs after hierarchy)
    + sum(interaction_coefficients for triggered interactions)
    [× normalization_factor]
    [× model_version_blend if phase-in]
```

In MA capitation, monthly payment is roughly:

```
PMPM = base_rate × RAF × geographic_factor × adjustments
```

> Verify the exact formula, normalization factor, blend percentages, and adjustment structure against the current CMS Rate Announcement. The conceptual formula above is for orientation only.

> Do **not** compute RAF in production code without running the **CMS-published model software** or a validated re-implementation against the **CMS-published denominator / coefficient files**.

---

## MEAT Criteria for Documentation

For an HCC-relevant diagnosis to be **defensible at audit**, the encounter documentation should evidence **MEAT** at the encounter level:

- **M**onitored — signs, symptoms, disease progression
- **E**valuated — test results, response to treatment, exam findings
- **A**ssessed / Addressed — discussion, ordering, counseling, referral
- **T**reated — medications, therapies, procedures

Pulling a diagnosis from the **problem list alone** without encounter-level MEAT evidence is the canonical RADV failure pattern.

### Equivalent frameworks

Some organizations use **TAMPER** (Treatment, Assessment, Monitor / Medicate, Plan, Evaluate, Referral) or similar mnemonics. The substantive requirement is the same: the encounter note must support that the condition was clinically addressed at this encounter.

> Verify current CMS RADV guidance and any sub-regulatory updates on documentation standards.

---

## HCC Capture Workflow

A typical HCC capture workflow integrates with both clinical and analytics systems:

1. **Population identification** — eligible members for the payment-year measurement period (enrollment, model eligibility).
2. **Suspect analytics** — identify likely-but-undocumented conditions from claims history, labs, meds, prior diagnoses, and (optionally) NLP on notes. These are **hypotheses**, not codes.
3. **Outreach prioritization** — schedule AWVs, prioritize chart reviews, queue provider alerts for upcoming visits.
4. **Pre-visit prep** — present suspect conditions and prior captured HCCs to the clinician with supporting evidence.
5. **Clinician assessment and documentation** — at the encounter, the clinician evaluates each condition and documents with MEAT.
6. **Encounter-level coding** — coding (CAC + human coder) assigns ICD-10-CM codes that reflect the documented assessment. See `medical-coding` for problem-list vs. encounter-level coding mechanics.
7. **Submission** — encounter data submitted to CMS via the current submission system. RAPS is retired; EDPS (Encounter Data Processing System) is the current path for MA — verify current submission system and timelines.
8. **Provider feedback** — capture rate, gap-close rate, RAF impact reporting back to clinicians.
9. **Audit-ready storage** — every submitted diagnosis traceable to a specific encounter record with documentation evidence.

### Suspect analytics caveats

- Suspect outputs are hypotheses. Codes must come from a clinician's documented encounter assessment.
- Aggressive suspect models can drive over-capture and audit exposure.
- Monitor **bias** — disparities in suspect-flag rates and capture rates by member demographics are a compliance and reputational risk.

---

## RADV (Risk Adjustment Data Validation)

CMS audits MA contracts (and may audit equivalent models) by **sampling beneficiaries** and pulling charts to validate that submitted diagnoses are supported. Findings can be **extrapolated** across the contract, producing large recoveries.

### Engineering implications

- **Document provenance** — every submitted diagnosis must trace to a specific encounter record retrievable on demand.
- **Chart-pull workflows** — bulk DocumentReference retrieval; ability to assemble a complete encounter packet (note + signed-by + amendments + supporting documents).
- **Deletion / reversal** — if a submitted code is unsupported, the plan must be able to **delete or reverse** it via CMS encounter-data submission.
- **Sampling-aware monitoring** — track high-RAF members and high-impact HCCs for proactive internal audit.

### Internal audit (pre-RADV)

Plans typically run their own internal HCC validation programs:

- **Concurrent chart audit** for high-risk diagnoses (e.g., major depression, neoplasms, chronic conditions with frequent miscoding).
- **Retrospective chart reviews** — clinician chart reviews to add or delete codes based on documentation. RADV-style review is now expected as a routine plan operation.
- **Coder QA** — sample-based reviews of CAC and coder performance.

> Verify current RADV methodology, extrapolation policy, and any updates to RADV scope against current CMS guidance.

---

## Engineering Patterns

### Source model files from CMS each year

For each payment year, download from the CMS Risk Adjustment page:

- The **model software** (binary or open-source re-implementation reference)
- The **mapping file** (ICD-10-CM → HCC by model version)
- The **denominator / coefficient file**
- The **factors file** (demographic coefficients, interaction coefficients, normalization factor)

Pin all of these by payment year. Do not blend across model years without applying the CMS-published blend percentages.

### Run CMS model software where possible

Re-implementing the CMS-HCC model from memory invites silent drift. Where possible, run the CMS-published software (or a re-implementation validated against it) and feed it claims and demographic data.

### Track per-member RAF history

Maintain an RAF time series per member with:

- Captured HCCs and the encounter sources that contributed to each
- Demographic factors at the time
- Model version used
- Normalization and blend at the time
- Reconciliation against the payer's submitted RAF (where the platform is provider-side) or against CMS-paid RAF (where payer-side)

### Provenance and audit

Every diagnosis used in RAF must link to:

- The encounter (date, provider, place of service)
- The clinical document (note, op report, etc.)
- The coder or autocoder decision
- The submission record
- Any subsequent reversal or amendment

Store these as an immutable audit log.

### Reproducibility

Given the same raw source data, the platform must reproduce the same RAF. Avoid non-determinism in suspect models, attribution windows, and date-of-service filtering.

---

## Common Pitfalls

- **Hard-coding HCC coefficients or ICD-to-HCC mappings from memory.** Source from the current CMS Risk Adjustment release.
- **Using a stale model version** (e.g., v22 or v24 after v28 phase-in has progressed past the applicable blend percentage).
- **Treating problem-list codes as billable evidence for HCC.** Codes must come from encounter-level documented assessment.
- **Skipping MEAT verification** — letting suspect-condition outputs flow to submission without clinician documentation.
- **Failing to delete or reverse** unsupported codes — leaves audit exposure on the books.
- **Submitting through retired pathways** — RAPS is retired; verify the current encounter-data submission system.
- **Ignoring bias monitoring** — disparities in suspect flagging or capture rates by member demographics expose the program to compliance and reputational risk.
- **Treating RAF as a single value** rather than a time series with payment-year and submission-version dimensions.

> Verify against the current CMS Risk Adjustment guidance, current CMS-HCC release, and current RADV methodology before relying on any specific mechanic above.
