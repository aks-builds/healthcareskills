# Risk Stratification Models

Reference comparison of risk stratification and predictive models commonly used in population health. **Verify current model versions, coefficient sets, and license terms against the model owner's published specification before deploying.** Coefficients and category definitions are version-specific and payment-year-specific; do not mix versions across reporting periods.

## Contents

- Diagnosis-based risk adjustment models
- Readmission risk models
- Falls and frailty risk
- Mental health and substance use risk
- Custom ML approaches
- Choosing a model

---

## Diagnosis-based risk adjustment models

These models take ICD-10-CM diagnoses (plus demographics, sometimes pharmacy and utilization) and produce a relative risk score used for payment, benchmarking, and stratification.

### CMS-HCC (CMS Hierarchical Condition Categories)

- **Owner**: CMS.
- **Purpose**: Medicare Advantage risk adjustment; Medicare ACO benchmarking (MSSP, ACO REACH).
- **Inputs**: ICD-10-CM diagnoses, age, sex, Medicaid status, originally-disabled flag, institutional / community status.
- **Output**: Risk score normalized so the average Medicare beneficiary is approximately 1.0.
- **Notes**: Payment-year-specific coefficient set; versions evolve. Verify against the current CMS Advance Notice and Rate Announcement for the relevant payment year.

### HHS-HCC

- **Owner**: CMS / CCIIO.
- **Purpose**: ACA marketplace (individual and small-group) risk adjustment.
- **Inputs**: Diagnoses, demographics, metal-tier-specific factors.
- **Output**: Plan-liability risk score.
- **Notes**: Distinct from CMS-HCC; do not substitute one for the other. Verify against the current HHS-HCC version published by CMS.

### ACG (Adjusted Clinical Groups)

- **Owner**: Johns Hopkins (commercial license).
- **Purpose**: Commercial, Medicaid, and integrated-system risk adjustment, segmentation, and predictive modeling.
- **Inputs**: ICD-10-CM diagnoses, age, sex; optionally pharmacy (Rx-MGs), utilization.
- **Output**: Multiple products including ADGs (Aggregated Diagnosis Groups), EDCs (Expanded Diagnosis Clusters), ACG categories, and predictive scores (e.g., probability of hospitalization, high-cost user prediction).
- **Notes**: Licensed software; output families and version differ. Verify version with the licensee.

### DxCG / Cotiviti DCG

- **Owner**: Verisk / Cotiviti (commercial license).
- **Purpose**: Commercial and Medicare risk adjustment and prediction.
- **Inputs**: Diagnoses, demographics.
- **Output**: Concurrent and prospective risk scores.
- **Notes**: Licensed; verify version.

### CDPS (Chronic Illness and Disability Payment System)

- **Owner**: University of California, San Diego.
- **Purpose**: Medicaid risk adjustment.
- **Inputs**: ICD-10-CM diagnoses.
- **Output**: Categorical risk groups and risk scores.
- **Notes**: Public-domain; widely used by state Medicaid programs. Verify current version.

---

## Readmission risk models

These models predict 30-day (or other window) readmission risk, used for care transition prioritization and bundled-payment risk management.

### LACE / LACE+

- **Owner**: Ontario research consortium (LACE original publication).
- **Purpose**: 30-day readmission and early death risk after discharge.
- **Inputs**: Length of stay (L), Acuity of admission (A), Charlson comorbidity index (C), Emergency department visits in prior 6 months (E). LACE+ extends with additional variables.
- **Output**: Numeric score; categorical risk band.
- **Notes**: Originally derived in Canadian data; performance varies by population. Verify against your population before clinical use.

### HOSPITAL score

- **Owner**: International research consortium.
- **Purpose**: 30-day potentially avoidable readmission risk.
- **Inputs**: Hemoglobin at discharge, discharge from oncology service, sodium at discharge, ICD-coded procedure during admission, type of admission, prior admissions in 12 months, length of stay.
- **Output**: Numeric score; categorical risk band.
- **Notes**: Verify against current published validation literature.

### Custom readmission models

- Many systems build custom ML readmission models on local data, incorporating EHR-rich features (vitals trajectory, labs, social factors).
- Validate for calibration, discrimination, and equity stratification before deployment. Re-validate periodically.

---

## Falls and frailty risk

### SAFER, Morse, Hendrich II

- **Purpose**: Falls risk in hospitalized or ambulatory older adults.
- **Inputs**: Mobility, history of falls, medications, mental status, vision, gait.
- **Notes**: Each is a specific instrument with defined scoring. Verify against the instrument's licensed publication before clinical use; do not mix scoring rules.

### Electronic Frailty Index (eFI)

- **Purpose**: Frailty in primary care populations using EHR data.
- **Inputs**: Cumulative deficit model across diagnoses, symptoms, signs, lab abnormalities.
- **Notes**: Open-source; widely used in UK and increasingly elsewhere. Verify current deficit list.

### Hospital Frailty Risk Score (HFRS)

- **Purpose**: Frailty risk in admitted patients.
- **Inputs**: ICD-10 codes associated with frailty.
- **Notes**: Verify current ICD-10 code list against the published HFRS specification.

---

## Mental health and substance use risk

- **PHQ-2 / PHQ-9** for depression screening (severity scoring).
- **GAD-7** for anxiety.
- **AUDIT-C / AUDIT** for alcohol use.
- **DAST-10** for drug use.

These are screening instruments, not population-level risk models, but they feed into segmentation overlays and care management prioritization. Each has a specific licensed scoring; verify before implementation.

---

## Custom ML approaches

Many organizations build custom ML models on local data:

- **Avoidable utilization** (ED super-utilizer, ambulatory care sensitive admissions, frequent 30-day readmits).
- **Specific-condition risk** (incident T2DM, CKD progression, CHF decompensation).
- **Uplift / impactability** — predicts the change in outcome from intervention, not baseline risk. Important for CM ROI.

Custom ML expectations:
- Hold out a temporal validation set.
- Calibration in addition to discrimination (AUC alone is insufficient).
- Equity stratification — performance by race, ethnicity, language, payer, geography.
- Monitor for drift; re-validate periodically.
- Explain model output to end users with sane reason-codes.

---

## Choosing a model

| Use case | Typical model family | Note |
|----------|---------------------|------|
| MA risk adjustment / payment | CMS-HCC | Mandated; payment-year version specific |
| ACO benchmarking (MSSP, REACH) | CMS-HCC | CMS-specified |
| ACA marketplace risk adjustment | HHS-HCC | CMS-specified |
| Commercial / self-funded analytics | ACG or DxCG | Licensed; depends on contract |
| Medicaid analytics | CDPS or ACG | CDPS is public-domain |
| 30-day readmission risk | LACE / HOSPITAL / custom | Validate locally |
| Care management prioritization | Layered: risk score + uplift + contactability | Avoid raw cost ranking |
| Falls risk | SAFER / Morse / Hendrich II | Pick one per care setting |
| Frailty | eFI / HFRS | Pick one per care setting |

Mark every model selection with its version, the data window used, and the validation date. Verify license terms before implementing any commercial model and verify open-domain models against the current publication.
