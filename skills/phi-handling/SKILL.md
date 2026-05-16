---
name: phi-handling
description: When the user is designing or reviewing the operational controls around PHI — how to de-identify, anonymize, pseudonymize, encrypt, mask, minimize, or securely retain Protected Health Information across structured data, free text, images, audio, and biometrics. Also use when the user mentions "Safe Harbor de-identification," "Expert Determination," "limited data set," "DUA," "k-anonymity," "l-diversity," "t-closeness," "differential privacy," "encryption at rest," "TLS 1.2," "KMS," "HSM," "RBAC," "ABAC," "break the glass," "minimum necessary in practice," "DICOM PHI," "burned-in PHI," "audio PHI," "biometric PHI," "secure deletion," "retention schedule," or "cross-border PHI." For the regulatory backdrop, see hipaa-compliance. For audit-log design, see audit-logging. For EU rules, see gdpr-health-data.
metadata:
  version: 1.0.0
---

# PHI Handling

You are an expert in the operational handling of Protected Health Information. Your job is to translate HIPAA, GDPR, and analogous regimes into concrete engineering controls — choosing the right de-identification method, the right encryption posture, the right access model, and the right retention path for each kind of PHI an organization holds.

## Initial Assessment

Read `.agents/healthcare-context.md` first (fall back to `.claude/healthcare-context.md`). The context tells you the HIPAA role, jurisdictions, EHR systems, encryption posture, and identity controls already in place. If absent, ask: what PHI elements are involved, where they flow (system A to system B to vendor C), what the downstream use is (clinical, billing, research, analytics, AI training), and which jurisdictions apply.

---

## The 18 HIPAA Identifiers (Practitioner View)

The Safe Harbor 18 are the practical "things to redact" list for U.S. data:

1. Names
2. Geographic subdivisions smaller than a state (full ZIP code redaction beyond first 3 digits, and only if the 3-digit ZIP covers > 20,000 people; otherwise mask to `000`)
3. Dates directly related to an individual (DOB, admission, discharge, death) — keep year only; ages > 89 collapse to "90+"
4. Phone numbers
5. Fax numbers
6. Email addresses
7. SSNs
8. MRNs
9. Health plan beneficiary numbers
10. Account numbers
11. Certificate / license numbers
12. Vehicle identifiers / VIN / license plates
13. Device identifiers / serial numbers
14. URLs
15. IP addresses
16. Biometric identifiers (fingerprints, voice prints, retinal scans)
17. Full-face photos and comparable images
18. Any other unique identifying number, characteristic, or code (catch-all)

This is the "do at least this" list for HIPAA Safe Harbor. It is not sufficient by itself if the residual data still allows re-identification (the **no actual knowledge** clause — see below).

---

## De-identification Methods (HIPAA)

### Safe Harbor (§164.514(b)(2))

1. Remove all 18 identifiers above
2. The CE has **no actual knowledge** that the remaining information could be used alone or in combination to re-identify an individual

Safe Harbor is mechanical and auditable, but conservative — it can over-redact useful data (e.g., dates of service for episode analytics).

### Expert Determination (§164.514(b)(1))

A qualified statistician/expert applies "generally accepted statistical and scientific principles and methods" and determines that the risk is "very small" that the information could be used, alone or in combination with other reasonably available information, to identify an individual. The expert must document the methods and results.

Use when Safe Harbor is too lossy (research, registry, analytics) and you have access to qualified expertise. Re-run on a defined cadence and on dataset changes.

### Limited Data Set + DUA (§164.514(e))

A LDS removes 16 of the 18 identifiers but **may retain dates and limited geographic information (city/state/ZIP)**. It is **still PHI** but may be shared for research, public health, or healthcare operations under a Data Use Agreement that binds the recipient to specified safeguards, prohibited uses, and breach reporting.

---

## Anonymization vs. Pseudonymization

| Concept | What it is | HIPAA / GDPR view |
|---------|------------|-------------------|
| Anonymization | Irreversible removal of links to an identifiable person | If properly done (Safe Harbor / Expert Determination under HIPAA; Recital 26 under GDPR) — out of scope of the rule |
| Pseudonymization | Replace direct identifiers with a token; key/lookup kept separate | Still personal data under GDPR; still PHI under HIPAA if the CE can re-identify |
| Tokenization | Form of pseudonymization, often format-preserving | Same as pseudonymization |
| Coded data | Same — code held by sender; recipient has no key | Recipient may treat as de-identified; sender still holds PHI |

Engineering implication: pseudonymization is a useful **security control** (limits blast radius if a downstream system is compromised) but it is not a regulatory off-switch.

---

## Statistical Disclosure Control in Healthcare

When releasing or sharing analytics datasets, the Expert Determination method commonly uses one or more of:

| Technique | Idea | Common use in healthcare |
|-----------|------|--------------------------|
| k-anonymity | Each record indistinguishable from at least *k-1* others on quasi-identifiers | Public-use health surveys, claims releases (e.g., k = 5, 10, 20) |
| l-diversity | Each k-anonymous group has at least *l* well-represented sensitive values | Mitigates homogeneity attacks (e.g., everyone in a bucket has the same diagnosis) |
| t-closeness | Distribution of a sensitive attribute in any group is close to distribution in the whole dataset | Mitigates skewness attacks |
| Differential privacy | Calibrated noise injection bounded by privacy budget ε | Public-health surveillance, some federated learning settings |
| Suppression / generalization | Drop rare cells; widen bins (age 0-4, 5-9, ...) | Public-use tables, registry exports |

These are tools, not guarantees. Combine with access controls, DUAs, and re-identification risk assessment.

---

## Encryption

### At Rest

- Algorithms: **AES-256** in an authenticated mode (GCM) or vetted alternative (AES-CBC + HMAC)
- Key management: dedicated KMS or HSM (cloud KMS such as AWS KMS, GCP KMS, Azure Key Vault Managed HSM; on-prem HSMs validated to FIPS 140-2/140-3 as required)
- Envelope encryption for application-layer fields (especially direct identifiers, free text containing PHI, audio, images)
- Distinct keys per tenant or per data class where blast radius matters; rotate per policy

### In Transit

- **TLS 1.2 or higher** (prefer 1.3) with modern cipher suites
- Strict certificate validation, no self-signed in production, HSTS where applicable
- For HL7 v2 over MLLP: tunnel through TLS or a VPN; raw MLLP over the public internet is unsafe
- For DICOM: TLS profile per IHE ATNA
- For FHIR: TLS plus OAuth/SMART app authorization

If ePHI is encrypted to HHS specifications (per the HHS guidance referenced by the Breach Notification Rule), lost/stolen data is not considered "unsecured PHI," which generally avoids breach notification. Encryption alone does not satisfy the Security Rule — risk analysis, access controls, audit, and contingency planning are still required. See **hipaa-compliance**.

---

## Access Controls

| Model | Use when |
|-------|----------|
| **RBAC** (role-based) | Clear role taxonomy maps to access needs (nurse, billing clerk, lab tech) |
| **ABAC** (attribute-based) | Access depends on context (treating relationship, department, time-of-day, patient consent flags) |
| **ReBAC** (relationship-based) | Care-team graphs — "is this clinician on the care team for this patient?" |
| **Purpose-of-use** | Required to disambiguate TPO vs. research vs. quality reporting; usually combined with one of the above |

### Break-the-Glass

For emergency or unanticipated access (ED override, on-call coverage, unanticipated consult), allow access with **explicit acknowledgement and mandatory enhanced logging**. The audit record must capture the override, the reason text, the user, the patient, and the timestamp. Review break-the-glass events on a defined cadence (often weekly). See **audit-logging**.

### Minimum Necessary in Practice

- Default to least-privileged role
- Filter PHI in API responses to fields actually needed (avoid "return the entire FHIR resource for a billing dashboard")
- Mask direct identifiers in analytics environments unless re-identification is needed
- For free-text fields, treat the whole field as PHI — do not assume the structured fields contain all identifiers

---

## Structured vs. Free-Text PHI

Free-text narrative (notes, scanned-document OCR, message bodies) routinely contains direct identifiers. Treat the full narrative as PHI unless it has been processed through a vetted de-identification pipeline. Common pipelines:

- Rule-based NER with healthcare-specific dictionaries
- Trained NER models for clinical text (e.g., open-source de-identification tools)
- Hybrid pipelines with human-in-the-loop review for downstream research data

De-identified text should still be subject to the Expert Determination process if it leaves the organization.

---

## Image PHI

DICOM images carry PHI in two places:

1. **DICOM header tags** — patient identifying tags (PatientName 0010,0010; PatientID 0010,0020; PatientBirthDate 0010,0030; many more)
2. **Burned-in pixel data** — text annotations baked into the image (especially ultrasound, secondary capture, screenshots)

For research or external sharing:

- Apply a DICOM **de-identification profile** (basic profile, plus options retain-longitudinal-temporal, retain-patient-characteristics, retain-device, etc., per the DICOM PS3.15 Confidentiality Profile)
- Detect and redact burned-in text (pixel inspection + OCR + masking)
- Strip private tags unless explicitly retained
- Validate after — load the de-identified study and confirm no residual identifiers

Photos, fundus images, dermatology images, and full-face photos are themselves identifiers (#17). Crop, blur faces, or obtain authorization.

---

## Audio PHI

Voice recordings, dictation, ambient-scribe captures, and call-center audio commonly contain names, dates, MRNs, and biometric voice prints. Treat as PHI in full. For analytics or training:

- Diarize, transcribe through a BA, redact transcript with a de-id pipeline
- Strip or distort speaker voice if the voice itself is the identifier (biometric)
- Retain raw audio only as long as needed for the documented purpose; encrypt at rest with restricted access

---

## Biometric Handling

Fingerprints, retinal/iris scans, palm vein, voice prints, face recognition templates are PHI (#16) and may also fall under state biometric laws (e.g., Illinois BIPA) and GDPR special category data. Practical controls:

- Store template, not the raw biometric, when feasible
- Per-purpose templates (don't reuse one template across systems)
- Strict access logging
- Honor deletion requests carefully — biometrics are not reissuable

---

## Secure Deletion

| Surface | Approach |
|---------|----------|
| Application database row | Logical delete + scheduled purge; consider cryptographic erasure (delete the per-record key) |
| Object storage | Delete object + lifecycle to permanently remove; account for cross-region replicas, snapshots |
| Backups | Documented retention; out-of-policy backups must age out; legal hold supersedes |
| Tape / archive | NIST SP 800-88 media sanitization (Clear / Purge / Destroy based on confidentiality) |
| Hardware decommission | Per NIST 800-88; document chain-of-custody |

Cryptographic erasure (destroying the key) is a defensible approach when physical sanitization is infeasible (cloud-only environments).

---

## Retention

HIPAA does not set a single retention period for the medical record — that is set by state law and clinical guidelines (often 7-10 years for adults, longer for minors counted from age of majority). HIPAA **does** require retention of HIPAA-specific documentation (policies, NPP versions, training records, BAAs, complaint logs, risk analyses, breach analyses) for **6 years** from creation or last effective date. Logs and audit records are covered under audit controls — see **audit-logging**.

Establish a written retention schedule mapped to:

- Medical/dental/imaging records (state-specific)
- Billing/claims records (CMS commonly cites 7 years for Medicare)
- HIPAA documentation (6 years)
- Audit logs (HIPAA min 6 years where treated as required-documentation; longer per breach investigation needs)
- Research consent and records (per IRB / sponsor / FDA)

---

## Cross-Border Transfer Flags

For any flow that leaves the country of collection:

- US to outside: contractually required safeguards (BAA, often standard data protection terms); some federal contracts forbid offshore PHI access
- EU/EEA to outside the EEA: GDPR Chapter V transfer mechanism (adequacy decision, SCCs 2021/914 with Transfer Impact Assessment, BCRs) — see **gdpr-health-data**
- Specific country regimes may layer on (UK GDPR, Swiss FADP, Brazil LGPD, India DPDPA, Canada PIPEDA + provincial, Australia Privacy Act with My Health Records Act)

Flag the flow in design review; do not assume "cloud region X is in country Y" without checking provider data residency commitments.

---

## De-identification Example (Illustrative — Synthetic Data Only)

**Before (PHI):**

```
PatientName: Doe, Jane R
DOB: 1972-03-14
MRN: 8472193
EncounterDate: 2026-04-22
Address: 415 Maple St, Springfield, IL 62704
Phone: (217) 555-0144
Note: "Patient seen today by Dr. Smith, scheduled follow-up 2026-05-15..."
```

**After Safe Harbor de-identification:**

```
PatientName: REDACTED
Year: 1972
MRN: REDACTED
EncounterYear: 2026
ZIP3: 627        # 3-digit ZIP if covering > 20,000 people; otherwise 000
Phone: REDACTED
Note: "Patient seen today by [PROVIDER], scheduled follow-up [DATE]..."
```

After Expert Determination, more granular dates and geography may be retained if the expert documents that re-identification risk is "very small" given the dataset and recipient.

---

## Common Anti-Patterns

- Treating pseudonymized data as anonymized
- Sharing the linkage key in the same system as the pseudonymized data
- Forgetting burned-in PHI in DICOM
- De-identifying structured fields and leaving free-text untouched
- Using production PHI in development or QA environments
- Training ML models on PHI without a documented purpose, BAA, and data minimization
- Skipping re-identification risk re-assessment after dataset changes

---

## Task-Specific Questions

1. What is the downstream use of the data — TPO (treatment / payment / operations), research, marketing, internal analytics, AI/ML training, or external sharing?
2. What modalities of PHI are involved — structured fields, free-text narrative, DICOM images, audio, video, biometric templates — and which are the highest-risk in this dataset?
3. Who are the downstream consumers (internal teams, named vendors, research collaborators, public release) and what is the legal/contractual basis for the flow (BAA, DUA, authorization, Article 9 condition)?
4. Which de-identification method is the right target — Safe Harbor, Expert Determination, Limited Data Set + DUA, pseudonymization, or a combination — and who will sign off?
5. What jurisdictions and overlay regimes apply (HIPAA, state laws like CA CMIA / WA My Health My Data / IL BIPA, GDPR, Part 2)?
6. What is the encryption posture today (algorithms, KMS/HSM, key custody, per-tenant or per-record keys) and what is the retention/deletion schedule?
7. Has anyone evaluated re-identification risk on the residual data, and on what cadence is that re-evaluated?

---

## Related Skills

- **hipaa-compliance**: the regulatory basis for these operational controls
- **healthcare-cybersecurity**: encryption, identity, and threat-aligned controls
- **audit-logging**: logging requirements for PHI access and break-the-glass
- **gdpr-health-data**: GDPR-specific obligations for EU/EEA data
- **hitrust-csf**: certification framework that audits these controls
- **dicom-imaging** (if present): DICOM-specific de-identification profiles
- **clinical-research** (if present): research data de-identification and DUAs
