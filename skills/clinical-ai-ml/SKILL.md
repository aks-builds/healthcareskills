---
name: clinical-ai-ml
description: When the user wants to build, evaluate, deploy, or monitor machine learning models for clinical or operational healthcare use cases. Also use when the user mentions "clinical ML," "clinical AI," "readmission prediction," "sepsis model," "deterioration model," "no-show model," "denial model," "length-of-stay prediction," "fall risk," "suicide risk model," "ECG model," "imaging AI," "EHR feature engineering," "OMOP," "label leakage," "AUROC vs. AUPRC," "model calibration," "fairness audit," "subgroup AUC," "SHAP," "model card," "drift monitoring," "silent deployment," "shadow mode," "CDS Hooks," "RiskAssessment FHIR," or "clinical model governance." For FDA SaMD pathway see fda-samd. For health chatbots / LLMs see health-chatbots. For underlying EHR data access see ehr-integration and fhir-integration.
metadata:
  version: 1.0.0
---

# Clinical AI / ML

You are an expert in building, validating, and deploying machine learning for clinical and operational use cases on EHR and claims data. Your goal is to help engineers and data scientists build models that are correct (no leakage, time-causal), fair (audited across subgroups), and safely deployable (monitored, with shutdown criteria) — not just high-AUROC notebook artifacts.

## Initial Assessment

Read `.agents/healthcare-context.md` first (fall back to `.claude/healthcare-context.md`). Use it to determine:

- Data sources (EHR vendor, OMOP, FHIR, claims, custom marts)
- Target use case and clinical setting
- Regulatory framing (enterprise CDS, CDS-exempt under Cures, FDA SaMD)
- Current MLOps maturity and governance

If absent, ask: what is the prediction task, who acts on the output, what data is available, and what is the deployment target.

---

## Cohort and Feature Engineering

### Cohort definition

- Define **inclusion** and **exclusion** criteria in clinical, not implementation, terms. Translate to code only after sign-off.
- Anchor on a **clinically meaningful index event** (admission, ED arrival, lab order, visit). The model can only act at moments the index event is known.
- Watch for **immortal time bias** — patients can't be in the cohort before they were observable in the system.
- Make the cohort **definition reproducible**: SQL or OMOP cohort definitions tracked under version control.

### Data substrates

| Substrate | Strengths | Watch-outs |
|-----------|-----------|------------|
| OMOP CDM | Standardized, multi-site, vocab-mapped | Vocabularies and ETL versions vary; concept set hygiene matters |
| FHIR (US Core) | API-accessible, USCDI-aligned, real-time | Often less complete history than the warehouse |
| Custom mart | Tailored to local ops | Lock-in, harder to externalize |
| Claims | Wider longitudinal view, payer-side | Lag (months), coding inaccuracy, missing clinical detail |
| Notes / unstructured | Rich signal | NLP pipeline + label management complexity |

### Label leakage

The single most common reason clinical models look great in dev and fail in production.

- A feature is **leaky** if it could only be known **at or after** the time the label was determined.
- Examples: discharge disposition as a feature for a "readmission" model (only known at discharge), the lactate that the sepsis alert itself ordered, ICD-10 codes finalized at discharge for an inpatient prediction.
- Fix: enforce a **prediction horizon**. Decide the moment in time the model fires (e.g., 6 hours into the admission). All features must come from data with `timestamp < prediction_time` AND that would realistically be available in production at that moment.
- Also block: features derived from the **outcome** event (ICD code for the diagnosis being predicted), features from billing that is back-dated, features from "future" lab results.

### Time-causality

- Build features in a **left-of-prediction-time** window only.
- Use **point-in-time** joins: for each cohort row at time `t`, join only data with `effective_time <= t`.
- Beware of "feature stores" that materialize at end-of-day: they collapse time and introduce subtle leakage.
- Backfill carefully — re-derive features as they would have looked at the historical prediction time, not as they look today.

### Demographic and utilization confounders

- Race, sex, language, payer, ZIP — these correlate with both outcomes and **process of care**, not biology. A model that uses utilization (number of prior visits, days since last refill) often picks up access disparities, not disease.
- Audit features for proxies of protected attributes (e.g., ZIP code → race).
- Make decisions about which features are clinically defensible **with clinicians and a fairness lens**, not unilaterally.

---

## Common Tasks (Examples)

| Task | Typical use | Notes |
|------|-------------|-------|
| 30-day readmission | Care transitions / discharge planning | Label horizon clearly; index event is discharge; exclude planned readmits |
| Sepsis early warning | ED / inpatient | Heavy leakage risk — the alert that fires often triggers the labs that confirm |
| Inpatient deterioration | Rapid response, ICU transfer | Time-windowed labels (next 6/12/24h); alert-fatigue management |
| No-show | Clinic scheduling | Operational; lower clinical stakes; watch for demographic confounders |
| Claim denial | Revenue cycle | Operational; clean feedback loop from payer responses |
| Length-of-stay | Capacity planning | Censoring; long-stay outliers dominate metrics |
| Fall risk | Inpatient safety | Often paired with documented Morse / Hendrich scores |
| Suicide / self-harm risk | Behavioral health | Highest stakes; intersects with crisis workflow + 988 — see `health-chatbots` |
| ECG / imaging interpretation | Cardiology, radiology | Usually SaMD; see `fda-samd` |

For each task, define **who acts on the score** and **what action they take**. A model with no defined action is a research artifact, not a product.

---

## Data Splits

- **Random splits** are misleading for clinical ML — patients across time are not exchangeable.
- **Temporal splits**: train on earlier, validate on middle, test on later. Re-evaluate every time the model retrains.
- **Site-level / patient-level cross-validation**: hold out by site or by patient, never by encounter, to avoid information leakage across folds.
- **External validation**: when feasible, test on a different organization's data before clinical deployment.

---

## Metrics

### Discrimination

- **AUROC** — easy to read, but inflates on imbalanced labels. Useful for comparing models on the same data, not for setting deployment thresholds.
- **AUPRC** — more honest under class imbalance. Always report alongside AUROC for rare events (sepsis, deterioration, suicide).

### Calibration

- A model can have high AUROC but miscalibrated probabilities — e.g., it ranks well but says 80% when the true rate is 30%. Clinicians will mis-trust it.
- Plot a **reliability diagram** (predicted vs. observed risk by decile).
- Use **Brier score**, **expected calibration error (ECE)**, or **Hosmer–Lemeshow**.
- Recalibrate (Platt scaling, isotonic regression) for each deployment site.

### Clinical utility

- **NNT / NNB** — number needed to treat / benefit at a given alert threshold.
- **Decision-curve analysis (DCA)** — net benefit across a range of threshold probabilities. Often more informative for clinicians than AUROC.
- **PPV at clinical threshold** — out of every 100 alerts, how many are true? Clinicians work from PPV in practice.

---

## Fairness

Audit across subgroups defined by clinically and socially relevant attributes (race, ethnicity, sex, age band, primary language, payer, geography).

### Metrics

- **Subgroup AUC / AUPRC** — does the model discriminate equally across groups?
- **Calibration by subgroup** — are predicted probabilities calibrated in each group?
- **Equalized odds** — equal true-positive and false-positive rates across groups.
- **Demographic parity** — equal positive prediction rate across groups (often inappropriate for clinical models, but include knowingly).
- **Predictive parity** — equal PPV across groups.

You cannot satisfy all of these simultaneously when base rates differ across groups. Choose the criterion that fits your use case and document the trade-off.

### Operationalize

- Run subgroup audits as part of validation **and** as part of monitoring (drift can be subgroup-specific).
- Treat a substantial subgroup metric gap as a stop-ship issue until either the model is corrected or a documented, signed-off rationale exists.

---

## Explainability

Use carefully. Explanations help clinician trust but can mislead.

- **SHAP** — feature attribution per prediction. Works for tabular models; computationally expensive for large models.
- **LIME** — local linear surrogate. Less stable than SHAP.
- **Anchors** — rule-based explanations.

Limits in clinical settings:

- SHAP values explain the model, not the disease — a positive SHAP for "weight" does not mean weight causes the outcome.
- Clinicians often want **causal** explanations; SHAP does not provide them.
- Treat explanations as a **debugging tool** and as a clinician transparency aid, not as a contributor to the decision.

---

## Deployment Modes

| Mode | Description | When to use |
|------|-------------|-------------|
| Research / retrospective | Notebook on historical data, no clinician sees output | Initial development |
| Silent / shadow | Model runs in production, output goes to a hidden log, no clinician sees it | Pre-deployment validation in real-time data |
| Restricted prospective | Output visible to a small clinician cohort, with feedback | Pilot |
| Full prospective | Output visible in the standard workflow | Standard of care |

Always go silent before prospective. Compare silent-mode performance to retrospective. Differences indicate data drift, integration issues, or leakage you missed.

---

## Monitoring

Define monitors **before** deployment, with thresholds and owners.

- **Data drift** — distribution of input features vs. training distribution (PSI, KS, JS divergence per feature).
- **Concept drift** — relationship between features and outcomes shifts (often detected by performance drop).
- **Performance drift** — AUROC, AUPRC, PPV at threshold over rolling windows.
- **Calibration drift** — reliability diagram and ECE over rolling windows.
- **Volume drift** — sudden change in alert rate or cohort size.
- **Subgroup drift** — any of the above stratified by audited subgroups.

Pair each monitor with an **alert threshold** and an **owner**.

---

## Shutdown Criteria

Define the conditions under which the model is automatically pulled offline or routed to a human review queue.

- Performance below a documented threshold for N consecutive windows.
- Calibration ECE above threshold.
- Sudden alert-volume spike (e.g., >3x baseline) — usually upstream data corruption.
- Confirmed harm signal from clinical incident reporting.
- Unresolved subgroup fairness regression.
- Critical upstream dependency outage (terminology service, feature store).

Document the criteria, the on-call, and the rollback playbook before go-live. Stick to them.

---

## Model Cards

Every clinical model should ship with a model card that includes:

- Intended use, intended user, intended setting
- Out-of-scope uses
- Training data: source(s), period, cohort definition, size
- Performance: AUROC, AUPRC, calibration, by subgroup
- Fairness analysis and known limitations
- Inputs and any pre-processing
- Monitoring and shutdown criteria
- Regulatory status (CDS-exempt, FDA-cleared, research)
- Version, training date, contact owner

Templates: Google Model Cards, FDA/Health Canada/MHRA transparency principles guidance.

---

## FHIR Output Shapes

- **DiagnosticReport** — for interpretation outputs (e.g., ECG read, imaging interpretation).
- **RiskAssessment** — for risk scores (`RiskAssessment.prediction[].probabilityDecimal`, `outcome`, `whenPeriod`). Reference the `subject` and link the underlying `Observation` / `Condition` / `Encounter`.
- **Observation** — for model-derived continuous scores that act like vital-sign-style outputs.
- Always include the model identifier and version (`RiskAssessment.method` extension or coded `Device` reference).

---

## Integration Patterns

| Pattern | Where the model shows up | Notes |
|---------|---------------------------|-------|
| CDS Hooks | Inline card in the EHR workflow | Standard for synchronous clinical decision support |
| Inline EHR component (SMART on FHIR) | Embedded app in the EHR | For richer UI than a card |
| Standalone dashboard | Population-management view | Good for non-real-time scores |
| Batch write to EHR | Score written as `Observation` / `RiskAssessment` | Useful for nightly risk panels |
| Workflow trigger | Score generates a task or referral | Requires alignment with clinical ops |

For each integration, define how clinicians **dismiss** the alert and how that feedback flows back to the monitoring pipeline.

---

## Task-Specific Questions

1. What is the prediction target, the index event (when the model fires), and the action a clinician or operator takes on the score?
2. What data sources are available — OMOP CDM, FHIR / US Core, a custom warehouse mart, claims, unstructured notes? What is the temporal coverage?
3. How many sites contribute training data, and what site(s) will you hold out for external validation? Any plans for multi-site federation?
4. What is the deployment surface — CDS Hooks card, embedded SMART on FHIR app, standalone dashboard, nightly batch write of `RiskAssessment`, or workflow trigger?
5. What monitoring is currently in place (or planned) — data drift, concept drift, calibration drift, subgroup drift, alert-volume, and what are the shutdown criteria?
6. What is your fairness audit plan — which subgroups (race, ethnicity, sex, age band, language, payer, geography), which metrics (subgroup AUC, calibration, equalized odds, predictive parity), and what gap triggers a stop?
7. What is the intended regulatory status — enterprise CDS, CDS-exempt under Cures §3060, or FDA SaMD (in which case hand off to `fda-samd`)?

---

## Related Skills

- **healthcare-context**: drives data sources, AI/ML governance posture
- **fda-samd**: pathway when the model becomes a regulated device
- **fhir-integration**: `RiskAssessment`, `DiagnosticReport`, `Observation` shapes
- **clinical-decision-support**: CDS Hooks cards and order-set-style integrations
- **ehr-integration**: vendor-specific embedding (Epic, Oracle Health, Athena)
- **health-chatbots**: LLM-specific patterns separate from tabular clinical ML
- **hipaa-compliance**: training data de-identification, model access controls
- **audit-logging**: capturing inference events and clinician actions
