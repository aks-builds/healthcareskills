# Dose and Range Checking

Dose-range checking is the safety check that compares an ordered dose against the maximum (and sometimes minimum) appropriate dose for the patient, given age, weight, renal function, hepatic function, and indication. It is one of the highest-value CPOE checks — and one of the most fatiguing when poorly tuned.

This reference is intentionally generic. Specific dose limits must come from the institution's verified formulary and a current authoritative drug-knowledge source (FDB, Medi-Span, Multum, Lexicomp). Never invent numeric limits.

## Why Dose Checking Matters

Dose errors are a large share of preventable medication harm. The errors most commonly intercepted by dose checking:

- Ten-fold overdose from misplaced decimal or unit confusion (mg vs. mcg).
- Renal-overdose from a dose appropriate for normal kidneys given to a patient with CKD.
- Hepatic-overdose for drugs cleared by the liver.
- Weight-based pediatric overdose from adult-dose default.
- Cumulative dose exceedance for drugs with a lifetime maximum (e.g., anthracyclines).

## Inputs to a Dose Check

A complete dose check considers:

- The ordered drug (RxCUI at SCD level, or ingredient + strength + form).
- The ordered dose (numeric + unit), frequency, route, and duration.
- Patient age.
- Patient weight (or BSA for chemo).
- Renal function (most often eGFR; some drugs need creatinine clearance via Cockcroft-Gault).
- Hepatic function (Child-Pugh class or LFT pattern).
- Indication (some drugs have different max doses by indication).
- Comorbidities (e.g., metformin contraindicated below some eGFR).
- Cumulative prior exposure (for lifetime-max drugs).

If any input is missing, the check is incomplete — surface that to the clinician rather than silently passing.

## Pediatric Considerations

Pediatric dosing is **fundamentally different** from adult dosing.

- Most pediatric doses are weight-based (mg/kg/dose or mg/kg/day).
- The maximum is the lower of (mg/kg cap, absolute adult dose).
- Neonatal and infant dosing often differs from older-pediatric.
- Premature neonates have additional adjustments.

Do not run a pediatric order through adult dose logic. The CPOE must select the correct age-banded rule set automatically based on the patient's date of birth and weight.

## Renal Dose Adjustment

For renally-eliminated drugs, the maximum dose, frequency, or both depend on renal function:

- Compute eGFR (or CrCl for drugs whose published dosing references it).
- Look up the dose adjustment for the patient's renal band per the institution's verified source.
- Recommend the adjustment passively if the ordered dose exceeds the band's recommendation.
- Hard-stop on contraindications (e.g., some drugs are contraindicated below a threshold).

Renal function is dynamic — recompute when fresh labs arrive and re-evaluate active orders. A dose that was right at admission may be wrong on hospital day 3 if AKI develops.

## Hepatic Dose Adjustment

For hepatically-cleared drugs:

- Look up the dose adjustment per Child-Pugh class (A/B/C) or LFT-based criteria per the institution's source.
- Some drugs are contraindicated in severe hepatic impairment.
- Hepatic function inputs are less standardized than renal — confirm what the institution uses.

## High-Alert Medications

The ISMP High-Alert Medications list identifies drug classes that cause disproportionate harm when used in error. For these classes, dose checking should be tighter and pair with additional safeguards:

| Class | Typical extra safeguards |
|-------|--------------------------|
| Anticoagulants | INR / aPTT / anti-Xa monitoring orders, weight-based dosing for LMWH, renal adjustment |
| Insulin | Glucose-monitoring orders, basal/bolus structure, sliding-scale guardrails |
| Opioids | MME ceiling per institution, naloxone availability, respiratory monitoring |
| Chemotherapy | BSA-based, cumulative lifetime caps, two-prescriber verification, protocol-based ordering |
| Neuromuscular blockers | Restricted ordering, monitoring required |
| Concentrated electrolytes | Restricted access, double-check |

> Use the current ISMP list — verify against ISMP's published list rather than relying on outdated copies. ISMP updates the list periodically.

## Drug-Knowledge Sources

Authoritative dose-range data must come from a maintained, current source. The major options:

| Source | Type | Notes |
|--------|------|-------|
| First Databank (FDB MedKnowledge) | Commercial | Widely embedded in US EHRs |
| Wolters Kluwer Medi-Span | Commercial | Widely embedded in US EHRs and pharmacies |
| Cerner Multum | Commercial | Embedded in some EHRs |
| Lexicomp (Wolters Kluwer) | Commercial | Reference + dosing data |
| Truven Micromedex | Commercial | Reference + dosing data |
| ISMP guidance | Reference content | Augments — does not replace — the structured data sources |

Pick **one** authoritative source per installation. Multiple sources in parallel produce conflicting alerts and confusing thresholds.

> Verify current licensing terms, content currency, and EHR integration support with the vendor. Pricing and integrations evolve.

## How Tightly to Tune

Untuned dose checking generates massive override rates. Most installations need iteration:

- Severe exceedance (e.g., >50% above max) -> interruptive, hard-stop.
- Moderate exceedance (e.g., 10-50% above) -> interruptive but acknowledgeable.
- Mild exceedance (<10% above) -> passive recommendation in the order-sign sidebar.
- Below recommended minimum -> usually a passive nudge; many clinicians intentionally start low.

Tune per service line. ICU sometimes legitimately dose above the routine ceiling; surgical floors do not.

## Patterns That Reduce Fatigue

- **Filter by exceedance magnitude**, not "any dose above any reference."
- **Suppress repeated alerts** for the same order in the same encounter.
- **Tier the alert** — pharmacy-only for some checks, prescriber for others.
- **Show the math** — what eGFR, what weight, what cited dose ceiling. Clinicians override less when they can see the basis (and this supports the Cures Act independent-review prong for CDS classification).
- **Pair with an order-set default** that pre-fills the correct dose so the alert rarely fires.

## Common Pitfalls

- Treating "any dose above the median" as an alert. Real harm is at the tails.
- Using a single adult ceiling for pediatric and neonatal patients.
- Not re-checking dose when renal function changes during the stay.
- Hardcoding ceilings instead of pulling from a maintained source.
- Letting multiple sources fire conflicting alerts.
- Counting an override that the prescriber documented with a valid reason as the same as a click-through override. Track reasoned overrides separately.

## Measurement

A healthy dose-check program measures:

| Metric | Direction |
|--------|-----------|
| Fire rate per order | Tune downward as low-value rules are retired |
| Accept rate (clinician changes the order in response) | Stable or up |
| Override-with-reason rate | High (clinicians documenting why) |
| Click-through-override rate | Low |
| Adverse events linked to overridden dose alerts | Investigated and used to re-tune |

> All numeric dose limits, classification thresholds, and clinical examples must be verified against the institution's authoritative drug-knowledge source and current ISMP high-alert list. This reference is structural, not clinical content.
