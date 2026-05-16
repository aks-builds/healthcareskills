# Predetermined Change Control Plan (PCCP) — Generic Outline

A **Predetermined Change Control Plan (PCCP)** is a mechanism FDA permits manufacturers to include in a marketing submission for an ML-enabled medical device. It lets the manufacturer pre-specify the changes intended after clearance, without requiring a new premarket submission for each change.

FDA finalized PCCP guidance in 2024. This reference is a generic structural outline. **Verify all content and structure against the current FDA PCCP final guidance and any subsequent clarifications before drafting a PCCP for an actual submission.**

This is engineering / product context, not a regulatory submission template. Final PCCPs are prepared in coordination with regulatory affairs.

## Three required elements

A PCCP under FDA guidance must contain three elements:

1. **Description of Modifications** — the specific, planned changes the manufacturer intends to make.
2. **Modification Protocol** — the V&V and documentation procedures used to implement each planned change.
3. **Impact Assessment** — an analysis of the benefits and risks of the planned modifications, including the controls.

Anything **outside** the PCCP still requires a new submission.

## 1. Description of Modifications

A clear, specific description of what changes are planned. Vague descriptions are routinely rejected.

### Subsections (suggested)

- **Modification 1 — Periodic retraining**
  - What is being retrained (the full model, a head, a calibration layer)
  - Why (drift, new data, new sites)
  - Cadence (e.g., every 12 months, or threshold-triggered)
  - Boundary conditions (e.g., only on data from existing sites; only with prevalence within +/-X% of training)

- **Modification 2 — New data sources**
  - Which new sources (e.g., additional health system, additional EHR vendor)
  - How they are characterized (data dictionary, terminology mapping, quality checks)
  - Boundary conditions

- **Modification 3 — New patient population**
  - Which expanded population (e.g., extending from adult inpatient to adult ED)
  - Inclusion / exclusion criteria changes
  - Boundary conditions

- **Modification 4 — Threshold tuning**
  - What threshold is tunable
  - Within what range
  - Operational triggers for tuning

- **Modification 5 — Recalibration**
  - Recalibration method (Platt, isotonic, etc.)
  - Per-site recalibration scope

- **Modification 6 — Bug fixes / safety updates**
  - Scope of fixes covered without resubmission
  - Boundary (any fix that changes intended use is out of PCCP)

Be conservative in scope. PCCP scope drift is a common review issue.

## 2. Modification Protocol

The procedures for implementing each modification. These must be specific enough that FDA can trust the change will be implemented safely.

### Components (suggested)

#### A. Data management plan

- Data ingestion procedures for new sources / periods.
- Data-quality acceptance criteria (missingness, coverage, terminology mapping completeness).
- Cohort definition update procedure.
- Training / validation / test split discipline (temporal, site-level, patient-level).
- De-identification, security, access controls during data movement.

#### B. Re-training procedure

- Training pipeline reference (code repository, container image, hyperparameters).
- Acceptance criteria for the retrained model (performance, calibration, fairness — all pre-registered thresholds).
- Comparison to the predecessor model (non-inferiority on key metrics, no regression on subgroups).
- Sign-off gate for promotion to production.

#### C. V&V procedure

- Internal validation on held-out data.
- External validation requirements (if any).
- Calibration analysis on the new data distribution.
- Fairness audit on audited subgroups.
- Documentation requirements: validation report, model card update.

#### D. Performance evaluation criteria

- Discrimination thresholds (AUROC / AUPRC).
- Calibration thresholds (ECE).
- Fairness thresholds (subgroup metric tolerances).
- Clinical-utility thresholds (PPV at threshold, decision-curve net benefit).
- Pre-registered acceptance criteria — not retrospective fits.

#### E. Update implementation procedure

- Software change-control records (linked to QMS).
- Code review, testing, release procedures.
- Deployment procedure (staging → silent / shadow → restricted prospective → full).
- Rollback procedure.
- Notification of changes to users / customers (labeling, release notes).

#### F. Monitoring and surveillance

- Post-modification monitoring of drift, performance, calibration, fairness, volume.
- Triggers for early evaluation of the modification.
- Triggers for rolling back the modification.

## 3. Impact Assessment

The analysis of benefits and risks of each planned modification.

### Components (suggested)

- **Benefits** — why the modification is being planned (sustained performance, expanded population, etc.).
- **Risks** — clinical risk, performance regression risk, fairness regression risk, operational risk.
- **Comparison to no modification** — the counterfactual cost of not making the change.
- **Controls** — how each risk is mitigated by the Modification Protocol.
- **Residual risk** — the risk that remains after controls.
- **Risk-acceptability** — per ISO 14971 framework.

The Impact Assessment should reference the Modification Protocol's controls and explain how each risk is addressed.

## What is NOT in scope of a PCCP

Anything that changes the **intended use** of the device is out of PCCP scope and requires a new submission. Examples:

- Changing from clinician-facing to patient-facing.
- Expanding from one clinical indication to another.
- Adding a new sensor / data type that wasn't anticipated.
- Architectural changes outside the modification descriptions.

Be conservative — listing a modification in the PCCP commits the manufacturer to executing it as described. Listing too little means more submissions later; listing too much locks in promises that may not fit how the model actually evolves.

## Cross-references with other FDA frameworks

- **GMLP** — the 10 GMLP principles structure how the PCCP is built (see `gmlp-principles.md`).
- **ISO 14971** — risk management; the Impact Assessment maps to risk-file content.
- **IEC 62304** — software lifecycle; the Modification Protocol's V&V procedures sit inside the software lifecycle.
- **Cybersecurity guidance** — modifications may have cybersecurity implications; address in the protocol.
- **Transparency principles** — modifications should be communicated to users in labeling / release notes.

## Engineering implications

The PCCP shapes the training, validation, and deployment pipeline. The pipeline must be implementable as code + procedures — not aspirations.

- Reproducible training runs with versioned code, data, hyperparameters.
- Automated V&V gates that enforce the pre-registered thresholds.
- Versioned monitoring metrics with documented thresholds.
- Audit trail per training run, per validation run, per deployment.
- Sign-off workflow that captures who approved each modification.
- Customer-visible change communication path.

## Cadence

- The PCCP itself does not have an expiration in current guidance — verify current.
- Each planned modification has its own implementation cadence (e.g., 12-month retrain).
- Periodic FDA reporting on modifications implemented is expected; verify the current required form and timing.

## Verify-current sources

- FDA final guidance "Predetermined Change Control Plans for Artificial Intelligence-Enabled Device Software Functions" (2024)
- FDA guidance on PCCPs more broadly across device types
- FDA / Health Canada / MHRA Good Machine Learning Practice principles
- FDA / Health Canada / MHRA Transparency for Machine Learning-Enabled Medical Devices: Guiding Principles
- ISO 14971 — Application of risk management to medical devices
- IEC 62304 — Medical device software lifecycle processes
