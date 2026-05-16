# 21st Century Cures Act §3060 — CDS Exemption Criteria

Section 3060 of the 21st Century Cures Act amended the FD&C Act so that certain **clinical decision support** software is not regulated as a medical device. FDA's 2022 final guidance "Clinical Decision Support Software" interprets the statutory criteria. This reference walks through the four criteria, with examples on each side of the line.

**This summarizes the statute and the 2022 final guidance. Verify against the current FDA guidance document and any subsequent FDA clarifications before relying on any specific conclusion.**

## The four criteria — all must be met

The software is **not** a device, and is therefore exempt from FDA premarket regulation, if **all four** of the following are true.

> ### Criterion 1
> The software is **not intended** to acquire, process, or analyze:
> - a **medical image**, or
> - a **signal from an in vitro diagnostic device**, or
> - a **pattern or signal from a signal acquisition system**.

> ### Criterion 2
> The software is intended to **display, analyze, or print medical information** about a patient, or **other medical information** such as peer-reviewed clinical studies and clinical practice guidelines.

> ### Criterion 3
> The software is intended to **support or provide recommendations** to a health care professional about prevention, diagnosis, or treatment of a disease or condition.

> ### Criterion 4
> The software is intended to enable the **health care professional to independently review** the basis for the recommendation, so the HCP does not rely primarily on the software to make a clinical decision.

If any one of the four fails, the software is **not** exempt and **is** a device. It then has to be analyzed under standard device pathways.

## Criterion 1 — Image / IVD / signal-acquisition

**On the "exempt" side:**

- A tool that surfaces the radiologist's existing report alongside the imaging order. Display only; no analysis of the image itself.
- A care-coordination dashboard that lists prior imaging studies by date and modality.
- A system that imports lab results values from a connected EHR (the IVD analyzed the sample; this tool is just displaying numeric values).

**On the "device" side:**

- Software that analyzes a chest X-ray and flags pneumonia → analyzes a medical image.
- Software that interprets an ECG waveform and predicts AFib → analyzes a signal from a signal-acquisition system.
- Software that processes a CGM trace and predicts hypoglycemia → analyzes a signal from a continuous-glucose-monitor signal-acquisition system.
- Software that analyzes audio waveform for cough patterns → analyzes a signal from a signal acquisition system.
- Software that ingests a raw IVD instrument signal (vs. a reported numeric result) and produces a different interpretation → analyzes a signal from an IVD.

Note: a system that displays a numeric **result** that an IVD instrument produced is generally not analyzing the IVD signal itself — but check the specifics of what your software is doing.

## Criterion 2 — Medical information

This criterion is usually satisfied by any software in the clinical-CDS space. The question is whether the software displays / analyzes / prints medical information about a patient or other medical information.

- A drug-drug interaction checker displays interaction information — satisfies this criterion.
- A guideline-summarization tool displays peer-reviewed evidence — satisfies this criterion.

Criterion 2 is rarely the failure point. It tends to be a pass-through in the analysis.

## Criterion 3 — Recommendations to a health care professional

The criterion requires the intended user to be a **health care professional** and the output to be a recommendation about **prevention, diagnosis, or treatment**.

**On the "exempt" side:**

- A diabetes-care dashboard for a physician with treatment recommendations under guideline thresholds.
- A care-gap surfacing tool for primary-care clinicians.

**On the "device" side:**

- A **patient-facing** symptom-checker that tells the patient whether to seek care — the user is not an HCP. Fails criterion 3.
- A **patient-facing** chatbot that recommends home remedies — fails criterion 3.
- A direct-to-consumer fitness app that recommends exercise based on a heart-rate sensor — fails criterion 3 (and possibly criterion 1).

A clinician-facing tool can fail this criterion if the recommendation is not about prevention, diagnosis, or treatment (e.g., a pure billing-coding suggestion is not a clinical recommendation, which is fine for billing tools but means they're not what this criterion was about — they have their own analysis).

## Criterion 4 — Independent review of the basis

This is the criterion where many ML-enabled CDS systems fail.

The software must be **intended to enable** the HCP to **independently review** the basis for the recommendation, such that the HCP does not rely primarily on the software.

FDA's interpretation focuses on **whether the HCP can reasonably understand**:

1. The input data the software used.
2. The relevant clinical and supporting data, such as peer-reviewed studies or guidelines.
3. The basis for the recommendation, including the rationale or algorithmic logic at a level sufficient for the HCP to assess.

**On the "exempt" side:**

- A rule-based CDS that says "Recommend ACE inhibitor for HTN patient with eGFR > 60" and shows the rule, the BP reading, and the eGFR.
- A guideline-citing recommendation that links to the source guideline.
- A score that shows the contributing factors and their weights with citations.

**On the "device" side:**

- A black-box ML model that outputs a single risk number with no explanation. The HCP cannot independently review what drove the prediction.
- A model with SHAP values that are uninterpretable in the clinical context (and FDA has noted post-hoc explanations are not necessarily sufficient).
- A **time-critical** alert where the HCP must act in seconds — the HCP cannot meaningfully review even if the basis is shown.
- A model whose basis is "we trained on 50,000 patients and got AUROC 0.85" — that is not a basis the HCP can independently review for a specific recommendation.

ML implementations often need to either: (a) provide enough transparency that an HCP can independently assess each recommendation, or (b) accept that they are not §3060-exempt and proceed through device pathways.

## Putting it together — common patterns

| Pattern | Criterion-by-criterion | Verdict |
|---------|------------------------|---------|
| Drug-drug interaction alert with citation | All pass | Likely exempt |
| Rule-based sepsis criteria checklist for clinicians | All pass | Likely exempt |
| ML inpatient deterioration score, clinician-facing, with feature attributions and time to review | 1 ok, 2 ok, 3 ok, 4 may or may not pass depending on review time and transparency | Borderline; conservative approach is to treat as device |
| ML sepsis early-warning real-time alert in EHR | 4 fails in practice (time-critical) | Device |
| ML chest-X-ray pneumonia detector | 1 fails | Device |
| Patient-facing symptom checker | 3 fails | Device |
| Black-box risk score for inpatients | 4 fails | Device |
| Clinician dashboard surfacing labs and rules with citations | All pass | Likely exempt |
| ECG-AFib interpretation | 1 fails | Device |

## When a CDS exemption analysis says "exempt"

- Document the analysis. Keep the rationale, the inputs to each criterion, the date, and the version of the software.
- Re-do the analysis when the software changes — adding a new feature, changing the output, broadening the user population can each shift the answer.
- Maintain the transparency that supports criterion 4: source citations, explanation panels, time for the HCP to review.

## When the analysis says "device"

- Hand off to the regulatory team.
- Walk the IMDRF matrix to anchor risk.
- Identify pathway: 510(k), De Novo, PMA.
- Stand up the quality system.
- Plan PCCP scope if ML-enabled.
- See `gmlp-principles.md` and `pccp-template.md`.

## Verify-current sources

- 21st Century Cures Act §3060 statutory text (FD&C Act §520(o))
- FDA final guidance "Clinical Decision Support Software" (September 2022)
- Any subsequent FDA clarifications or sub-regulatory guidance
- IMDRF SaMD risk categorization framework
