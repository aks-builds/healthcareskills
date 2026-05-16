# Good Machine Learning Practice (GMLP) — 10 Guiding Principles

In late 2021, **FDA**, **Health Canada**, and the **UK MHRA** jointly published **"Good Machine Learning Practice for Medical Device Development: Guiding Principles"** — a set of 10 principles to inform the development of safe, effective, high-quality medical devices that use AI / ML.

GMLP is **guidance**, not regulation. But it is increasingly referenced in submissions, audits, and regulator expectations. Use it as a design checklist and as a structure for the evidence package in a submission.

**Verify the principles against the current joint publication. The principles below summarize the published version; the regulators have signaled additional joint work that may evolve them.**

## The 10 principles (summary)

### 1. Multi-disciplinary expertise is leveraged throughout the total product life cycle

Bring clinical, engineering, data-science, regulatory, human-factors, and ethics expertise across design, development, validation, and post-market activities. ML in healthcare is not a single-discipline problem.

**Engineering implications:**
- Cross-functional design reviews at each phase.
- Clinical co-design of cohorts, features, outputs, and clinician-facing UI.
- Ethics review of fairness / impact analyses.

### 2. Good software engineering and security practices are implemented

Treat the ML system as a regulated software system. Apply IEC 62304 lifecycle, configuration management, code review, automated testing, secure coding standards. Treat the model + training pipeline + serving infrastructure as one system.

**Engineering implications:**
- Source control for code, training pipeline, configuration.
- Reproducible training runs with versioned data, parameters, environment.
- Software bill of materials (SBOM) per release.
- Secure-by-design — see FDA premarket cybersecurity guidance.

### 3. Clinical study participants and datasets are representative of the intended patient population

The training, validation, and test datasets should reflect the patients the device will see in deployment. Underrepresentation of subgroups risks subgroup performance gaps.

**Engineering implications:**
- Document training data demographics by audited subgroups.
- Flag underrepresentation and either expand data or constrain intended use.
- Run subgroup audits as part of every validation.

### 4. Training data sets are independent of test sets

Standard ML hygiene. Patients in training do not appear in test. Sites in training may or may not appear in test depending on the intended generalization. Cross-validation folds split by patient (or site), never by encounter.

**Engineering implications:**
- Strict split discipline enforced in code, not by convention.
- Independence checks: same MRN, same encounter, same household-address heuristics.

### 5. Selected reference datasets are based upon best available methods

If the device uses a reference dataset for benchmarking or for ground-truth labeling, the reference is established using accepted methods (e.g., adjudicated clinical reviews, established gold-standard tests). Document the method and its limitations.

**Engineering implications:**
- Documented annotation / adjudication process with inter-rater agreement.
- Versioned reference datasets with release notes.

### 6. Model design is tailored to the available data and reflects the intended use of the device

Model architecture, complexity, and design choices fit the data volume, the data quality, and the deployment context. A 100M-parameter model trained on 500 patients is misaligned. So is a real-time alert architecture built without latency targets.

**Engineering implications:**
- Model selection matches data scale, signal complexity, interpretability needs.
- Architectural choices documented relative to intended use.

### 7. Focus is placed on the performance of the human-AI team

The device's performance is the performance of the clinician-and-device together — not the model alone. Optimize for the workflow outcome, not for AUROC.

**Engineering implications:**
- Workflow studies and human-factors testing.
- Metrics include alert-to-action conversion, time-to-disposition, missed-alert investigation rate.
- UI design that supports the clinician's decision, with affordances for override and feedback.

### 8. Testing demonstrates device performance during clinically relevant conditions

The testing covers the real conditions of use — patient mix, data drift, missingness patterns, integration latency, clinician workload conditions, and demographic / site variation.

**Engineering implications:**
- External validation on at least one site / organization different from training.
- Stress testing with realistic missingness and degraded data.
- Performance under deployment-equivalent latency.
- Silent / shadow deployment before prospective use.

### 9. Users are provided clear, essential information

Users — clinicians and patients — receive the information needed to use the device safely. Labeling, model card, in-product explanations, intended use, performance characteristics, known limitations, contraindications.

**Engineering implications:**
- Model card published with each release.
- In-product transparency (input summary, contributing factors, performance reference).
- Clear labeling for cleared-vs-uncleared features.

### 10. Deployed models are monitored for performance and re-training risks are managed

Post-market: drift, performance, calibration, fairness, volume, clinical action. Re-training is itself a risk — controls and PCCP are how that risk is managed.

**Engineering implications:**
- Monitoring framework with thresholds and owners (see `clinical-ai-ml` `monitoring-checklist.md`).
- PCCP if the model is intended to be retrained post-market.
- Shutdown criteria and rollback playbook.

## How to use GMLP in a submission

A FDA submission for an ML-enabled device can be structured to address each GMLP principle explicitly. While not required, this provides a transparent evidence package and aligns with the international principles:

- Section X.1: "Multi-disciplinary expertise applied" — team composition, design-review cadence.
- Section X.2: "Software engineering and security" — SDLC, IEC 62304, cybersecurity threat model, SBOM.
- Section X.3: "Representative data" — training data demographics, subgroup coverage.
- Section X.4: "Independent training and test sets" — split methodology, independence checks.
- Section X.5: "Reference datasets" — annotation method, adjudication, versioning.
- Section X.6: "Model design tailored to data and use" — architecture rationale.
- Section X.7: "Human-AI team performance" — workflow testing, human-factors results.
- Section X.8: "Clinically relevant testing conditions" — external validation, stress testing, silent deployment.
- Section X.9: "Clear, essential information to users" — labeling, model card.
- Section X.10: "Post-market monitoring and re-training" — monitoring framework, PCCP.

This structure also supports a QMS audit and an internal governance review.

## Cross-references

- FDA / Health Canada / MHRA: Transparency for Machine Learning-Enabled Medical Devices: Guiding Principles (a complementary set on transparency)
- ISO 14971 — risk management — the safety analysis backbone
- IEC 62304 — software lifecycle
- IEC 62366-1 — usability engineering
- FDA Cybersecurity in Medical Devices guidance
- IMDRF SaMD risk categorization

## Verify-current sources

- FDA / Health Canada / MHRA: "Good Machine Learning Practice for Medical Device Development: Guiding Principles" — joint publication. Verify the current edition; the regulators have indicated additional joint work.
- FDA software-related guidance documents (CDS Software, PCCP, premarket submissions for device software functions, AI/ML action plan updates).
- IMDRF SaMD documents.
