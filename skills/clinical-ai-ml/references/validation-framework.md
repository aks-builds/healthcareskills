# Clinical ML Validation Framework

This reference is a checklist for validating a clinical machine-learning model before it sees a clinician in a prospective setting. The order is important: cohort first, splits second, metrics third, fairness fourth, and only then deployment readiness.

## 1. Cohort definition

### Define in clinical terms first

- Express inclusion and exclusion criteria in plain clinical language with a clinician co-author, before any SQL is written.
- Each criterion should answer: who, when (with respect to the index event), and on what data substrate.
- Distinguish target cohort (the population the model is for) from training cohort (the data you actually use).

### Pick the index event

- The **index event** is the moment in time at which the model fires (admission, ED arrival, lab order, visit, discharge, etc.).
- All features must be derivable from data with `effective_time <= index_time` AND data that would realistically be in the production system at `index_time`.
- The label is determined from data **after** the index event within a documented horizon (e.g., 30-day readmission, sepsis within 24h).

### Watch out for

- **Immortal time bias** — patients can't be in the cohort before they were observable in the system.
- **Selection bias** — the cohort definition can favor sicker or healthier patients in ways that distort generalization.
- **Survivorship bias** — patients who die before the index event are excluded, which can mask true risk.

### Reproducibility

- Cohort definitions live as SQL or OMOP `cohort_definition` records under version control.
- A re-run of the cohort definition against the same data snapshot should produce the same row set.

## 2. Splits

### Random vs. temporal vs. site / patient

| Split type | When to use | Watch out for |
|------------|-------------|---------------|
| **Random** (encounter-level) | Almost never for clinical models | Patient appears in train + test; massive leakage |
| **Random** (patient-level) | Acceptable for cross-sectional tasks | Time-correlation still leaks |
| **Temporal** | Default for most clinical prediction | Hold out the latest period as test; use earlier for train; use middle for validation |
| **Site-level** | Multi-site models | Hold out an entire site as test; tests true generalization across operations |
| **External validation** | Before any production deployment | Hold out a different organization's data — the gold standard |

### Rule of thumb

- For a model that will be deployed at site A: train and validate on earlier data from site A, test on later data from site A (temporal), then validate externally on site B before going live.

### Splits and class balance

- Do not oversample / undersample the test set. Keep the test set at the natural prevalence the model will see in production.
- Up- or down-sampling during training is fine if you understand the calibration implications (recalibrate to natural prevalence).

## 3. Metrics

### Discrimination

- **AUROC** — easy to read; inflates on imbalanced labels; useful for comparing models on the same data. Not useful for deployment-threshold selection.
- **AUPRC** — more honest on rare events. Always report alongside AUROC for prevalence < 10%.

### Calibration

- A model can have high AUROC and miscalibrated probabilities. Clinicians distrust mismatched probabilities.
- **Reliability diagram** — predicted vs. observed risk by decile.
- **Brier score** — overall quality of probabilistic predictions.
- **Expected calibration error (ECE)** — mean miscalibration across bins.
- **Hosmer-Lemeshow** — goodness-of-fit test; used carefully on large samples.
- Re-calibrate per deployment site (Platt scaling, isotonic regression).

### Clinical utility

- **NNT / NNB** — number needed to treat / benefit at a chosen alert threshold.
- **Decision-curve analysis (DCA)** — net benefit across a range of threshold probabilities. Often the most clinician-useful single chart.
- **PPV at clinical threshold** — out of every 100 alerts at threshold T, how many are true positives? Clinicians work from PPV.
- **Alert rate at clinical threshold** — operational planning input.

## 4. Fairness audit

### Subgroup definitions

Audit across subgroups defined by clinically and socially relevant attributes:

- Race, ethnicity, sex, age band, primary language, payer, geography (urban / rural), insurance status, disability status where captured.
- Define the subgroups in a prospective audit charter, not after looking at the data.

### Subgroup metrics

- **Subgroup AUC / AUPRC** — discrimination by group.
- **Subgroup calibration** — reliability diagram and ECE within each group.
- **Equalized odds** — equal true-positive and false-positive rates across groups.
- **Predictive parity** — equal PPV across groups.
- **Demographic parity** — equal positive-prediction rate; often inappropriate clinically but include knowingly when relevant.

### The trade-off

- You cannot satisfy all fairness criteria simultaneously when base rates differ across groups (the impossibility theorem). Pick the criterion that fits your use case and document the trade-off.

### Operationalize

- Treat a substantial subgroup gap as a stop-ship until either the model is corrected or a documented, signed-off rationale exists.
- The fairness audit is repeated in monitoring (drift can be subgroup-specific).

## 5. Calibration on deployment

Even after a good development calibration, deployment in a new setting often requires recalibration:

- Site-specific case mix differs.
- Coding practices differ.
- Operational workflow differs (the "what was charted at the index event" composition).

Plan a **silent / shadow deployment** to gather production-equivalent data and recalibrate before clinicians see the score.

## 6. Cross-validation hygiene

If you cross-validate:

- Fold by **patient** (or by **site**), never by **encounter**.
- Apply preprocessing (imputation, normalization, feature selection) **within each fold**, not before splitting. Otherwise the test data leaks into preprocessing.
- For temporal data, use **rolling-origin** cross-validation rather than k-fold.

## 7. Sanity checks

Before you trust an AUROC, run sanity checks:

- **Permutation test** — shuffle labels; AUROC should drop to ~0.5. If it doesn't, you have leakage somewhere (e.g., the data split itself is leaky).
- **Date-only baseline** — a model trained only on date features. If it's competitive with your model, you may have temporal leakage.
- **Single-feature baseline** — fit a logistic regression on the top three SHAP-by-mean-magnitude features. If it matches your complex model, complexity wasn't earning its keep.
- **Future leakage check** — for each top feature, compute the timestamp distribution relative to index. Flag features with timestamps suspiciously close to the label.

## 8. Documentation

Every model that proceeds past validation produces:

- **Model card** (see `monitoring-checklist.md` for the template).
- **Validation report** with all metrics above, split definitions, calibration plots, fairness analysis.
- **Cohort definition** SQL / OMOP definition under version control.
- **Feature dictionary** — name, definition, source, time window, expected range, missingness handling.
- **Pre-registered evaluation plan** — what you were going to measure, before you measured it.

## 9. Validation gates

Recommended gates before deployment:

| Gate | Criterion |
|------|-----------|
| Cohort sign-off | Clinical lead approves the cohort definition |
| Leakage audit | Permutation test passes; date-only baseline does not approach model AUROC; feature-time audit clean |
| Internal validation | Temporal hold-out AUROC and AUPRC meet pre-registered thresholds |
| External validation | At least one external site or organization replicated key metrics within tolerance |
| Calibration | ECE < pre-registered threshold; reliability diagram acceptable across deciles |
| Fairness | Subgroup AUC / calibration / chosen parity metric within tolerance, or documented rationale |
| Silent deployment | Production-equivalent performance matches retrospective within tolerance |
| Clinical sign-off | Clinical lead signs the validation report and the model card |
| Regulatory sign-off | If SaMD, regulatory affairs signs off; otherwise, governance committee |

Each gate fails closed. If a gate is skipped, document why and who approved it.

## Verify-current sources

- Methods papers on prediction modeling (TRIPOD-AI, CONSORT-AI, SPIRIT-AI extensions)
- IMDRF guidance on ML-enabled medical devices
- FDA / Health Canada / MHRA Good Machine Learning Practice principles
- Decision-curve analysis methodology (Vickers et al.)
- Calibration / reliability-diagram methodology (Niculescu-Mizil & Caruana)
