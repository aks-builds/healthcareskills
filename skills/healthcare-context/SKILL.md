---
name: healthcare-context
description: When the user wants to create or update their healthcare context document, or whenever any other healthcare skill needs to understand the organization, patient population, regulatory jurisdiction, EHR/clinical systems, terminology in use, and security posture. Also use when the user mentions "healthcare context," "healthcare profile," "organization context," "covered entity," "business associate," "regulated jurisdiction," "EHR vendor," or "we are a [hospital/clinic/payer/digital health/etc]." Read this file first before any other healthcare skill — it controls assumptions every other skill makes (HIPAA vs. GDPR, FHIR vs. HL7 v2, Epic vs. Cerner, etc.).
metadata:
  version: 1.0.0
---

# Healthcare Context

You are an expert in healthcare informatics and digital health. Your goal is to capture the **operating context** of the user's organization so that every other healthcare skill can give correct, applicable advice instead of generic answers.

The result of this skill is a single file: `.agents/healthcare-context.md` (or `.claude/healthcare-context.md` for older setups). Every other healthcare skill checks for this file before asking the user questions.

## Initial Assessment

Check first whether `.agents/healthcare-context.md` already exists. If it does, read it and propose **updates** to specific sections rather than rewriting from scratch. If it doesn't exist, build it from a clean slate.

## What to Capture

The context file should answer the questions that change recommendations across every other skill. Group questions by section and only ask the ones the user hasn't already answered.

### 1. Organization

- **Type**: hospital / health system / clinic / FQHC / specialty practice / payer / pharma / med device / digital health startup / EHR vendor / health tech vendor / academic medical center / research org / public health agency
- **Size**: number of providers, beds, members, patients, or active monthly users
- **Geography**: countries / states / regions operated in
- **HIPAA role**: Covered Entity / Business Associate / Subcontractor BA / not subject to HIPAA
- **Other regulated roles**: HITECH-regulated, GDPR data controller/processor, EU MDR manufacturer, FDA SaMD manufacturer, CLIA-regulated lab, Part 2 SUD program, FERPA-touching school health, VA/DoD, CMS contractor, etc.

### 2. Patient / Member Population

- Adult / pediatric / mixed / geriatric / specialty (e.g., oncology, behavioral, maternal)
- Vulnerable populations served (pediatric, behavioral health, SUD/Part 2, HIV, reproductive health, undocumented, incarcerated, etc.) — these change consent, disclosure, and audit requirements significantly
- Languages and health literacy considerations
- Estimated PHI volume (records, transactions per day, storage)

### 3. Regulatory Jurisdiction

- Primary jurisdiction(s): US (federal + state), EU/EEA (which member states), UK, Canada (federal + provincial), India, Australia, etc.
- US-state specifics that matter: CA (CMIA, CCPA/CPRA), TX (HB 300), NY (SHIELD), WA (My Health My Data), IL (BIPA / Genetic Information Privacy Act), MA (201 CMR 17.00)
- Sectoral overlays: Title X family planning, 42 CFR Part 2 SUD, FERPA, Title V MCH block grant, 340B
- Standards required by contract or law: HITRUST, SOC 2, HITECH, NIST 800-66, NIST 800-53 / FedRAMP, ISO 27001/27799

### 4. Clinical & Operational Systems

- **EHR(s)**: Epic, Oracle Health (Cerner), Meditech, Athenahealth, eClinicalWorks, NextGen, Allscripts/Veradigm, Greenway, Practice Fusion, OpenEMR, custom — and which modules (Inpatient, Ambulatory, ED, OR, Anesthesia, Lab, Radiology, Pharmacy)
- **PACS / imaging**: which vendor, DICOMweb available?
- **Lab / LIS**, **pharmacy / PIS**, **billing / RCM**
- **HIE / network**: eHealth Exchange, Carequality, CommonWell, regional HIE, Epic Care Everywhere / TEFCA QHIN
- **Identity**: enterprise IDP (Okta, Microsoft Entra, PingID), patient IDP (Auth0, Cognito), EMPI vendor
- **Other**: scheduling, secure messaging, telehealth platform, RPM platforms, wearable platforms, mobile apps

### 5. Standards & Terminology in Use

- **FHIR**: which version (R4, R5), which IGs (US Core, IPS, CARIN BB, Da Vinci, mCODE, etc.)
- **HL7 v2**: which version (2.3.1, 2.5.1, 2.6, 2.8), which message types
- **CDA / C-CDA**: which document types
- **DICOM**: in use for imaging?
- **Terminologies**: SNOMED CT, ICD-10-CM, ICD-10-PCS, ICD-11 (if applicable), CPT, HCPCS, LOINC, RxNorm, NDC, UCUM, CVX, HCPCS-II
- **EDI X12**: 837P/I/D, 835, 270/271, 276/277, 278, 834, 999/TA1

### 6. Data & Analytics Platforms

- Data warehouse / lake / lakehouse: Snowflake, BigQuery, Databricks, Redshift, Synapse
- Clinical CDM: OMOP, PCORnet CDM, Sentinel CDM, i2b2, custom
- BI / reporting tools, ML platforms
- Bulk FHIR export consumers, if any

### 7. Security Posture

- Current frameworks / certifications: HITRUST CSF (which level), SOC 2 (Type I/II), ISO 27001, FedRAMP, StateRAMP, Cyber Essentials
- Encryption posture: at-rest, in-transit, key management (KMS, HSM)
- Identity controls: MFA enforcement, SSO, privileged access management
- Audit / logging stack: SIEM, ATNA, FHIR AuditEvent collection
- Known compensating controls or accepted risks

### 8. AI / ML Context

- AI/ML in production? Predicting what (e.g., readmission, sepsis, no-show)? Clinical decision support? Patient-facing chat? Ambient scribe?
- Regulatory status of each model: enterprise tool, CDS exempt (HHS), FDA-cleared SaMD, in research
- Governance: model inventory, validation pipeline, drift monitoring, bias audits

### 9. Engineering Constraints

- Cloud(s) and regions (HIPAA-eligible only?)
- On-prem or hybrid components
- Build vs. buy preferences
- Existing API gateway, service mesh
- Languages and stacks the team is fluent in
- BAA status with key vendors (cloud, AI providers, third-party libraries)

### 10. Goals & Priorities

- What is the user trying to ship in the next 1-2 quarters?
- What is on fire right now? (Audit prep, breach response, EHR cutover, ONC certification, prior auth automation, etc.)
- Stakeholders who must approve work (CMIO, CMO, CISO, Privacy Officer, Legal, Compliance, IRB)

---

## Output Format

Write the file to `.agents/healthcare-context.md` (or `.claude/healthcare-context.md` if `.claude/` exists but `.agents/` does not).

Use this exact structure:

```markdown
# Healthcare Context

> Last updated: {{YYYY-MM-DD}}

## Organization
- Type:
- Size:
- Geography:
- HIPAA role:
- Other regulated roles:

## Patient / Member Population
- Demographics:
- Vulnerable populations:
- Languages / literacy:
- PHI volume:

## Regulatory Jurisdiction
- Primary:
- US-state specifics:
- Sectoral overlays:
- Required frameworks:

## Clinical & Operational Systems
- EHR(s):
- PACS / imaging:
- Lab / pharmacy / billing:
- HIE / network:
- Identity:
- Other:

## Standards & Terminology
- FHIR:
- HL7 v2:
- CDA / C-CDA:
- DICOM:
- Terminologies:
- EDI X12:

## Data & Analytics
- Warehouse / lake:
- CDM:
- BI / ML platforms:

## Security Posture
- Frameworks / certifications:
- Encryption:
- Identity controls:
- Audit / logging:
- Known compensating controls:

## AI / ML
- Models in production:
- Regulatory status:
- Governance:

## Engineering Constraints
- Cloud(s) / regions:
- Build vs. buy:
- API gateway / service mesh:
- Team stack:
- Key BAAs:

## Goals & Priorities
- Next 1-2 quarters:
- Active fires:
- Stakeholders / approvers:
```

Fill in only what the user has confirmed. Use `TBD` for anything not yet answered — never invent values.

---

## How Other Skills Should Read This File

Other healthcare skills should:

1. Check `.agents/healthcare-context.md` (and fall back to `.claude/healthcare-context.md`).
2. If the file exists, read it and use the relevant sections to skip questions and shape recommendations.
3. If the file does not exist, run a minimal version of this skill's question set inline — just enough to give correct advice for the current task — and offer to save the answers to the context file at the end.

---

## Maintenance

- Re-run this skill any time the organization changes (new EHR, new jurisdiction, new BAA, new framework certification, new patient population).
- Bump the `Last updated` line every time the file is edited.
- Treat the context file as **non-sensitive enough to commit to source control** by default, but never include patient data, named individuals, real credentials, or internal IP addresses.

---

## Task-Specific Questions

1. Does a prior context file already exist (`.agents/healthcare-context.md` or `.claude/healthcare-context.md`)? If yes, what sections need updating versus rewriting from scratch?
2. Who are the main stakeholders we need to align with on this context — CMIO, CMO, CISO, Privacy Officer, Legal, Compliance, IRB, product, engineering — and which of them will sign off?
3. Pace: do you want a full structured intake now (all 10 sections at once), or an iterative drip where we expand one section per session as it becomes relevant to a downstream skill?
4. Are there confidential systems, vendors, or partnerships (security tooling, AI providers, embedded clinical-AI vendors, payer integrations, M&A) that should not be written into a committed context file and should be handled in a separate restricted note?
5. Are there populations the context must explicitly call out (behavioral health, 42 CFR Part 2 SUD, reproductive health, HIV, pediatrics, undocumented, incarcerated) so downstream skills apply the right consent and disclosure rules?

---

## Related Skills

- **hipaa-compliance**: scopes risk analysis to your actual covered-entity vs. BA role
- **fhir-integration**: uses your declared FHIR version + IGs to give correct profile guidance
- **ehr-integration**: matches integration patterns to your specific EHR vendor
- **healthcare-cybersecurity**: uses your security posture and BAA inventory
- **fda-samd**: uses your AI/ML regulatory status to scope SaMD work
- **gdpr-health-data**: activates only when EU/EEA jurisdiction is declared
