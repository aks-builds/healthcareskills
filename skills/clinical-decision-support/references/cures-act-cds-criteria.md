# Cures Act CDS Exemption — Four-Prong Test

The 21st Century Cures Act (2016) created a statutory carve-out so that certain clinical decision support software is **not** a "device" under the Federal Food, Drug, and Cosmetic Act and therefore not subject to FDA medical-device regulation as Software as a Medical Device (SaMD). FDA has subsequently issued guidance refining the criteria.

To qualify for the exemption, the CDS function must meet **all four** of the prongs below. Failing any one prong means the software is a regulated device and must go through the appropriate FDA pathway (510(k), De Novo, PMA, or other applicable route).

> The wording below is a working paraphrase. Verify the exact criteria against the current FDA guidance "Clinical Decision Support Software" (most recent final version) and the Cures Act statute itself. The four-prong test has been refined since 2016 and may be revised again.

## The Four Prongs

### Prong 1 — Source data

The CDS is **not** intended to acquire, process, or analyze:
- A medical image, OR
- A signal from an in vitro diagnostic device, OR
- A pattern or signal from a physiological monitor (e.g., raw waveform data).

In practice, this means the CDS operates on **EHR data** (codified problems, labs, meds, structured observations, demographics) — not raw signals or images.

### Prong 2 — Information displayed

The CDS is intended to display, analyze, or print medical information about a patient or other medical information such as clinical guidelines, peer-reviewed studies, or institutional protocols.

### Prong 3 — Purpose

The CDS is intended for the purpose of supporting or providing recommendations to a **health care professional** about the prevention, diagnosis, or treatment of a disease or condition.

(Patient-facing decision support is treated differently — patient-facing software is generally not eligible for this exemption.)

### Prong 4 — Independent review

The CDS is intended to enable the health care professional to **independently review the basis for the recommendation** so the professional does not rely primarily on the software's recommendation to make a clinical decision.

In practice, this means the clinician must be able to inspect:
- The inputs the CDS used
- The logic or reasoning that produced the output
- The evidence base (guideline, study, or rule) that justifies the recommendation

And must have the time and information necessary to actually do that review.

## Worked Examples — On the Exempt Side

| Function | Why it qualifies |
|----------|------------------|
| Flagging an outpatient diabetic patient as overdue for HbA1c, with a card that shows the active diabetes Condition, the date of the last HbA1c, and the cited guideline | Prong 1: EHR data, not signals. Prong 2: displays guideline + patient data. Prong 3: supports clinician decision (order vs not order). Prong 4: clinician can see the inputs (diabetes problem, last lab date) and the source guideline. |
| An order set for adult sepsis bundle that pre-selects labs, fluids, and broad-spectrum antibiotic options with the supporting evidence | EHR data, supports clinician, evidence visible per item, clinician can de-select and edit. |
| A risk score for 30-day readmission computed from claims and EHR fields, with the contributing factors and a citation of the validated model | EHR data, clinician-facing, clinician can see contributing factors (the drivers). Assumes the drivers are interpretable — if the model is a black box and the clinician cannot meaningfully review the basis, prong 4 fails. |
| A renal-dose-adjustment passive recommendation card showing the calculated eGFR, the dose threshold for the ordered drug, and the published guideline | All four prongs satisfied; clinician can verify. |

## Worked Examples — On the Regulated Side

| Function | Why it does NOT qualify |
|----------|-------------------------|
| An algorithm that flags possible stroke on a CT angiogram image | Prong 1 fails — analyzes a medical image. SaMD. |
| ECG arrhythmia detection from raw waveform data | Prong 1 fails — analyzes a physiological-monitor signal. SaMD. |
| A "black box" deep-learning sepsis predictor whose contributing factors are not exposed to clinicians, used in the ED where there is no time to review the basis | Prong 4 fails — independent review is not feasible. SaMD. |
| A continuous glucose monitor app that interprets sensor data and recommends insulin dose | Prong 1 fails — analyzes IVD signal. SaMD. |
| Patient-facing app that recommends whether to seek emergency care | Prong 3 fails — recommendation is to a patient, not a health care professional. May require regulation depending on indication. |
| A CDS that is so heavily relied on in a time-critical setting (e.g., trauma resuscitation) that the clinician cannot meaningfully review the basis | Prong 4 fails by use context. SaMD. |

## Design Implications

If you want your CDS to remain exempt, **build for prong 4**. This is where most modern AI/ML-driven CDS gets stuck.

Concrete design practices that help satisfy prong 4:

- Show the inputs the CDS used (every prefetch resource, every value).
- Show the logic in a form a clinician can read. For rule-based CDS, that is the rule. For models, that is calibrated probability + top contributing factors + the validated reference.
- Cite the evidence base on every card via the `source` field.
- Reserve modal / hard-stop alerts for true safety events. A modal that the clinician cannot click past does not enable independent review.
- Pace the workflow so the clinician has time to actually review.

If any of these are infeasible for the use case, the tool is likely a SaMD — go to the **fda-samd** skill and plan a regulatory submission.

## Related Regulations

- **ONC HTI-1 (2024)** — Imposes transparency requirements on **Predictive Decision Support Interventions (DSIs)** in ONC-certified EHRs. Even CDS that qualifies for the Cures Act exemption may be subject to HTI-1 source-attribute reporting (training data, validation, fairness measures, etc.) when delivered through a certified EHR. Verify current HTI-1 requirements against the official rule.
- **State telehealth and scope-of-practice laws** — May constrain who the CDS can recommend to and what disclaimers are required.
- **FTC and state consumer-protection law** — Apply to marketing claims about CDS, exempt or not.

## Sanity-Check Questions Before Claiming Exemption

1. Does the CDS ever process a medical image, IVD signal, or physiological waveform? If yes — not exempt.
2. Is the user a health care professional? If patient-facing — likely not exempt.
3. Can the clinician see the inputs, the reasoning, and the cited evidence on screen? If no — likely not exempt.
4. Is the clinical setting one where the clinician has time and information to independently review? If no — likely not exempt.
5. Has counsel reviewed the design against the current FDA CDS guidance? If no — pause before deploying.

> The four-prong test and FDA's interpretive guidance evolve. Confirm against the current FDA "Clinical Decision Support Software" final guidance and consult regulatory counsel for any borderline use case.
