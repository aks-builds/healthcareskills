# Clinical ML Monitoring Checklist

This reference covers post-deployment monitoring for clinical machine-learning models: what to monitor, how to set thresholds, what triggers a shutdown, and a model-card template. Set up monitoring **before** deployment, with thresholds and named owners. Adding monitoring after a model goes live almost never happens.

## What to monitor

### 1. Data drift

The distribution of input features in production vs. training.

- **Per-feature drift**: Population Stability Index (PSI), Kolmogorov-Smirnov (KS) distance, Jensen-Shannon (JS) divergence, Wasserstein distance.
- **Multivariate drift**: covariance shifts, PCA-based detectors, density-ratio estimation.
- **Missingness drift**: per-feature missingness rate over time.

### 2. Concept drift

The relationship between features and outcomes shifts.

- Often detected indirectly by performance drop.
- Cohort-stratified residual analysis can localize the shift.

### 3. Performance drift

The headline metrics on a rolling window of production data with labels.

- AUROC over rolling N-day windows.
- AUPRC over rolling windows.
- PPV at the deployed clinical threshold.
- Sensitivity / specificity at threshold.
- Note: requires labels, which typically lag — the monitor has built-in latency.

### 4. Calibration drift

- Reliability diagram on rolling windows.
- ECE on rolling windows.
- Brier score on rolling windows.

### 5. Volume drift

- Alert rate per day / week.
- Cohort size per day / week.
- Sudden alert-rate spike (e.g., >3x baseline) is almost always upstream data corruption rather than a real performance change.

### 6. Subgroup drift

- All of the above, stratified by the audited subgroups (race, ethnicity, sex, age band, language, payer, geography).
- Subgroup drift can be invisible in aggregate metrics.

### 7. Clinical-action drift

- Alert-to-action conversion rate over time. If clinicians stop acting on the alerts, that's a signal — could be alert fatigue, could be loss of trust, could be workflow change.
- Time-to-action distribution.
- Override / dismissal rate.

### 8. Upstream-dependency health

- Terminology service availability (LOINC mappings, ICD lookups).
- Feature-store latency and freshness.
- EHR API error rate.
- Model-serving latency and error rate.

## Thresholds and owners

For every monitor:

- **Threshold** — a numeric trigger that crosses to "alert" or "incident."
- **Owner** — a named person or team that gets paged.
- **Runbook** — what the on-call does when the alert fires (triage, investigate, escalate, shut down).
- **Severity** — informational, warning, page-the-on-call.

### Example thresholds (pre-register these — do not pick after observing drift)

| Monitor | Threshold | Severity |
|---------|-----------|----------|
| PSI per feature | > 0.2 | Warning |
| PSI per feature | > 0.25 | Page |
| AUROC drop from baseline | > 0.03 | Warning |
| AUROC drop from baseline | > 0.05 sustained 14 days | Page + consider shutdown |
| Calibration ECE | > pre-registered threshold (e.g., 0.05) | Warning |
| Calibration ECE | > 1.5x threshold sustained 14 days | Page + consider shutdown |
| Alert volume | > 3x rolling baseline | Page (almost always upstream) |
| Subgroup AUC gap | > pre-registered tolerance | Warning + fairness review |
| Clinical action rate | drops > 30% from baseline | Warning + clinical review |

## Shutdown criteria

Define explicit conditions under which the model is pulled offline or routed to a human review queue. Document these before go-live and stick to them.

Recommended shutdown triggers:

- **Performance** below documented threshold for N consecutive windows.
- **Calibration ECE** above threshold for N consecutive windows.
- **Alert volume** spike confirmed as upstream data corruption.
- **Confirmed harm signal** from clinical incident reporting (one event may be enough, depending on severity).
- **Unresolved subgroup fairness regression** past a defined tolerance.
- **Critical upstream dependency** outage that cannot be remediated within SLA.
- **Regulatory direction** (FDA, internal regulatory affairs) to pause.

For each trigger, document:

- Who decides to shut down (named role, escalation path).
- Who notifies clinical operations.
- How the model is removed from the workflow (feature flag, EHR config change, route to bypass).
- What the user-facing message is during downtime.
- How the model is brought back online (re-validation, sign-off, who approves).

## Rollback playbook

- A feature flag that disables model output in the EHR with no code deploy.
- A documented rollback test, exercised at least quarterly.
- A backup of the prior production model in case the rollback is to a previous version, not a complete shutdown.

## Model card template

Every clinical model should ship with a model card. Use this template as a starting point.

```
# Model Card — [Model Name]

## 1. Intended use
- Primary use case (one sentence)
- Intended user (role)
- Intended setting (where / when)
- Intended patient population

## 2. Out-of-scope uses
- Uses explicitly NOT supported
- Populations explicitly NOT supported
- Settings explicitly NOT supported

## 3. Inputs
- Feature list with definitions, sources, time windows
- Input pre-processing / imputation
- Required upstream dependencies (terminology service, feature store)

## 4. Output
- Output shape (probability, risk tier, recommendation)
- Output range
- FHIR shape (RiskAssessment / DiagnosticReport / Observation)
- Threshold for action (if any) and rationale

## 5. Training data
- Source(s), period covered
- Cohort definition (link to versioned definition)
- Size (rows, patients, encounters)
- Demographic composition
- Known limitations / biases

## 6. Validation results
- Discrimination: AUROC, AUPRC (overall and by subgroup)
- Calibration: reliability diagram, ECE, Brier
- Clinical utility: PPV at threshold, decision-curve analysis
- Fairness analysis with audited subgroups
- External validation results (if any)

## 7. Monitoring
- Monitors active in production (list)
- Thresholds and severity per monitor
- Owners per monitor
- Frequency

## 8. Shutdown criteria
- Performance triggers
- Calibration triggers
- Volume triggers
- Harm-signal triggers
- Fairness triggers
- Dependency triggers

## 9. Regulatory status
- Enterprise CDS (CDS-exempt under §3060) / FDA-cleared (510(k) / De Novo / PMA) / research
- Submission references if applicable
- PCCP scope if applicable

## 10. Version and lineage
- Model version, training date
- Training pipeline reference
- Code repository commit
- Data snapshot

## 11. Owners
- Technical owner
- Clinical owner
- Regulatory owner (if applicable)

## 12. Change history
- Version history with dates, what changed, why
```

## Cadence

- **Daily**: data drift, alert volume, upstream-dependency health, serving latency.
- **Weekly**: performance metrics (where labels are available), calibration, subgroup metrics.
- **Monthly**: full subgroup audit, full fairness review, alert-to-action conversion review.
- **Quarterly**: model-card review, rollback drill, shutdown-criteria review, threshold review.
- **Annually**: external-validation refresh (where applicable), regulatory-status review.

## Anti-patterns

- "We'll add monitoring later." Later does not happen. Build it before deployment.
- Single AUROC chart as "monitoring." That's a dashboard, not monitoring.
- Thresholds set after observing drift. Set them before, with clinical and statistical justification.
- No named owner per monitor. Without an owner, alerts go to the void.
- No rollback test. Untested rollbacks fail under pressure.
- "We only monitor when there's a complaint." Complaints are a lagging, biased signal.

## Verify-current sources

- Google Model Cards
- FDA / Health Canada / MHRA Transparency for Machine Learning-Enabled Medical Devices: Guiding Principles
- IMDRF guidance on ML-enabled medical devices
- Methods literature on data-drift detection (PSI, KS, Wasserstein, density-ratio estimation)
- Calibration / reliability-diagram methodology (Niculescu-Mizil & Caruana)
