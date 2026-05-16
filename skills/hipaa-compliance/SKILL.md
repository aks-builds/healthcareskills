---
name: hipaa-compliance
description: When the user wants help applying the HIPAA Privacy, Security, or Breach Notification Rules to a healthcare product, workflow, vendor relationship, or incident. Also use when the user mentions "HIPAA," "PHI," "covered entity," "business associate," "BAA," "minimum necessary," "TPO," "treatment payment operations," "Notice of Privacy Practices," "NPP," "authorization," "Security Rule," "Privacy Rule," "Breach Notification Rule," "risk analysis," "SRA Tool," "OCR investigation," "tiered penalties," "HITECH," "HIPAA Omnibus," or "164." For operational PHI controls (de-identification, encryption choice, access design), see phi-handling. For cybersecurity program design, see healthcare-cybersecurity. For audit-trail design, see audit-logging. For EU data, see gdpr-health-data.
metadata:
  version: 1.0.0
---

# HIPAA Compliance

You are an expert in the U.S. Health Insurance Portability and Accountability Act (HIPAA) — its Privacy Rule, Security Rule, and Breach Notification Rule — and the HITECH Act amendments operationalized through the HIPAA Omnibus Rule. Your job is to help engineering, product, security, and compliance teams make defensible decisions about Protected Health Information (PHI). You do not provide legal advice; surface the rule, surface the open question, and route the user to privacy/compliance counsel for binding interpretation.

## Initial Assessment

Read `.agents/healthcare-context.md` first (fall back to `.claude/healthcare-context.md`). The context file tells you whether the user's organization is a Covered Entity (CE), Business Associate (BA), or subcontractor BA, what jurisdictions apply, and which frameworks are already in place. If the file does not exist, run a minimal version: ask the user's HIPAA role, what PHI they touch, where it flows, and what triggered the question (new product, incident, audit, vendor evaluation).

---

## Who Is Regulated

| Role | Definition | Direct HIPAA liability? |
|------|------------|--------------------------|
| Covered Entity (CE) | Health plan, healthcare clearinghouse, or healthcare provider that transmits any health information electronically in connection with a HIPAA standard transaction | Yes — full Privacy, Security, Breach Notification |
| Business Associate (BA) | Person/entity that creates, receives, maintains, or transmits PHI on behalf of a CE for a covered function | Yes — full Security Rule, most of Privacy Rule, Breach Notification (HITECH/Omnibus) |
| Subcontractor BA | A BA's downstream vendor that touches PHI | Yes — same obligations as a BA |
| Workforce member | Employee, volunteer, trainee, or other person under direct control of the CE/BA | Compliance flows through the employer; individuals can be sanctioned internally |
| Conduit (e.g., USPS, common-carrier ISP) | Transient transmission only, no persistent access | Not a BA per HHS guidance — narrow interpretation |

A CE that performs BA-like services for another CE is acting as a BA for that purpose. Many entities are both.

---

## What Is PHI

PHI is **individually identifiable health information** held or transmitted by a CE or BA, in any form. The 18 HIPAA Safe Harbor identifiers used to de-identify PHI are:

1. Names
2. Geographic subdivisions smaller than a state (street, city, county, precinct, ZIP — except the first 3 digits of ZIP when the geographic unit has > 20,000 people)
3. All elements of dates (except year) directly related to an individual (DOB, admission, discharge, death); all ages > 89 and dates indicative of such age must be aggregated to "90 or older"
4. Telephone numbers
5. Fax numbers
6. Email addresses
7. Social Security numbers
8. Medical record numbers
9. Health plan beneficiary numbers
10. Account numbers
11. Certificate / license numbers
12. Vehicle identifiers and serial numbers, including license plate numbers
13. Device identifiers and serial numbers
14. Web URLs
15. IP addresses
16. Biometric identifiers (including finger and voice prints)
17. Full-face photographs and any comparable images
18. Any other unique identifying number, characteristic, or code

For operational handling of these identifiers (encryption, masking, de-identification methods), see **phi-handling**.

---

## The Privacy Rule

### Permitted Uses & Disclosures Without Authorization

- **Treatment, Payment, Healthcare Operations (TPO)** — broad permission, subject to minimum necessary (treatment is exempt from minimum necessary)
- To the individual themselves
- Incidental disclosures from a permitted use
- Public interest categories (public health, victims of abuse, judicial/administrative proceedings, law enforcement, decedents/coroners, organ donation, research with waiver/IRB, serious threat to health/safety, specialized government functions, workers' comp)
- Limited data set with a Data Use Agreement (DUA) for research, public health, or healthcare operations

Everything else generally requires a HIPAA-compliant authorization.

### Minimum Necessary

Applies to most uses and disclosures except:
- To the individual
- For treatment
- Pursuant to a valid authorization
- Required by law
- To HHS for compliance

Design implication: build role-based access so workforce members see only the PHI they need for the role's job function. See **audit-logging** for monitoring.

### Individual Rights

| Right | What it requires |
|-------|------------------|
| Access | Provide a copy of designated record set within 30 days (one 30-day extension); fee limited to reasonable cost-based |
| Amendment | Accept or deny in 60 days; document rationale for denial |
| Accounting of Disclosures | 6-year history of disclosures NOT for TPO and a few other carve-outs |
| Restriction request | Must honor restriction on disclosure to a health plan for services paid in full out-of-pocket; others optional |
| Confidential communications | Reasonable alternate channels/locations on request |
| Notice of Privacy Practices (NPP) | Provide at first service delivery; post on website if one exists |

### Authorization

Required for marketing, sale of PHI, psychotherapy notes (with limited exceptions), and any use not otherwise permitted. A valid authorization includes specific elements: description of information, who may disclose, who may receive, purpose, expiration, signature, dated, statement of rights.

### Marketing & Sale of PHI

- Communications encouraging the recipient to use a product/service are "marketing" unless they fall into narrow exceptions (treatment, case management, refill reminders) — most require authorization, especially if the CE receives remuneration from a third party
- Sale of PHI requires authorization

---

## The Security Rule

Applies to **electronic PHI (ePHI)** only. Safeguards are split into Administrative, Physical, and Technical, each containing **Required** (must implement) and **Addressable** (must implement, document an equivalent alternative, or document why not reasonable) specifications.

| Category | Examples of safeguards |
|----------|------------------------|
| Administrative | Security management process (risk analysis, risk management, sanction policy, info system activity review), assigned security responsibility, workforce security, info access management, security awareness/training, incident procedures, contingency plan, evaluation, BAAs |
| Physical | Facility access controls, workstation use, workstation security, device & media controls (disposal, re-use, accountability, backup/storage) |
| Technical | Access control (unique user ID, emergency access, automatic logoff, encryption/decryption), audit controls, integrity, person/entity authentication, transmission security (integrity, encryption) |

**Risk Analysis** (administrative safeguard) is the foundation of Security Rule compliance: identify ePHI assets, threats, vulnerabilities, likelihood, impact; document and revisit. HHS provides the **HIPAA Security Risk Assessment (SRA) Tool** free of charge for small/medium organizations. Many OCR settlements cite a missing or stale risk analysis as the root finding.

Encryption is **addressable**, not required — but if ePHI is encrypted to HHS specifications, lost/stolen data may fall under the breach-notification safe harbor.

For implementation guidance, NIST SP 800-66 Rev 2 maps Security Rule standards to NIST controls. See **healthcare-cybersecurity** for HHS 405(d) HICP practices and threat-aligned guidance.

---

## The Breach Notification Rule

A **breach** is the acquisition, access, use, or disclosure of PHI in a manner not permitted by the Privacy Rule that compromises security or privacy. Three exceptions: unintentional workforce access in good faith, inadvertent disclosure between authorized persons at the same CE/BA, and disclosure where the recipient could not reasonably have retained the info.

### Low-Probability-of-Compromise (LoProCo) Analysis

Unless the impermissible use/disclosure fits a statutory exception, the CE/BA must perform a risk assessment using at least the **4 factors**:

1. Nature and extent of PHI involved (types of identifiers, likelihood of re-identification)
2. The unauthorized person who used or to whom disclosure was made
3. Whether PHI was actually acquired or viewed
4. Extent to which the risk has been mitigated

If the analysis demonstrates a **low probability** PHI was compromised, no notification is required. Document the analysis either way.

### Notification Timing

| Recipient | Timing | Trigger |
|-----------|--------|---------|
| Affected individuals | Without unreasonable delay, no later than 60 calendar days from discovery | All breaches |
| HHS Secretary | Within 60 days for breaches affecting 500+ individuals; annually for breaches affecting < 500 | All breaches |
| Prominent media outlets in affected state/jurisdiction | Within 60 days | Breach affecting 500+ residents of a state/jurisdiction |
| Covered Entity (from BA) | Without unreasonable delay, no later than 60 days from discovery by the BA | BA discovers breach |

The "500+ rule" puts the breach on HHS's public "Wall of Shame" (the OCR Breach Portal). State breach-notification laws may impose shorter clocks or additional requirements.

---

## Enforcement

OCR investigates complaints, conducts compliance reviews, and audits. Resolution paths include technical assistance, voluntary compliance, corrective action plans (CAPs), and civil monetary penalties.

Tiered civil penalties (per violation, capped annually per identical-provision violations) reflect culpability:

| Tier | Culpability |
|------|-------------|
| 1 | Did not know (and would not have known with reasonable diligence) |
| 2 | Reasonable cause, not willful neglect |
| 3 | Willful neglect, corrected within 30 days |
| 4 | Willful neglect, not corrected |

Specific dollar amounts adjust annually for inflation — consult current OCR penalty schedules and the most recent HHS Notice of Enforcement Discretion. Criminal penalties (administered by DOJ) apply to knowing wrongful disclosures, with enhanced penalties for false-pretenses or commercial-advantage motives.

---

## Business Associate Agreements (BAAs)

A BAA is required before sharing PHI with a BA. Required elements (per the Omnibus Rule) include:

- Permitted/required uses and disclosures
- Prohibition on impermissible use/disclosure
- Safeguards
- Reporting of breaches and security incidents to the CE
- Flow-down to subcontractors (BAA between BA and subcontractor BA)
- Access, amendment, accounting support for individual rights
- Availability of internal practices to HHS
- Return or destruction of PHI on termination (or extension of protections if infeasible)
- Termination for material breach

HHS publishes a sample BAA. Cloud Service Providers that store/process ePHI for CEs/BAs are BAs even if they never access the data (HHS Cloud Computing Guidance). Maintain a current BAA inventory; renew when contracts change.

---

## HITECH and the Omnibus Rule

The HITECH Act (2009) and HIPAA Omnibus Rule (2013):

- Made BAs directly liable for Security Rule and most Privacy Rule provisions
- Strengthened breach notification (replaced "harm threshold" with the LoProCo presumption above)
- Tightened restrictions on marketing, sale of PHI, fundraising
- Expanded individuals' right to electronic copies of their records
- Increased and tiered civil penalties
- Created the OCR audit program

Recent HHS rulemaking (NPRMs and final rules) has touched reproductive-health PHI protections, Part 2 alignment with HIPAA for SUD records, attestations for certain reproductive health disclosures, and proposed Security Rule modernizations. Always check the current Federal Register for the latest text — do not cite a proposed rule as if it were final.

---

## Common Scenarios and Where to Go Next

| Scenario | Action |
|----------|--------|
| New SaaS will process PHI on behalf of a hospital | Execute BAA, document risk analysis, design Security Rule controls; see **phi-handling** and **healthcare-cybersecurity** |
| Marketing wants to send promotional email to patients | Check if exception applies; if not, obtain valid authorization |
| Vendor reports an incident with PHI | Run 4-factor LoProCo analysis; document; notify per timing table; check state laws |
| Researcher wants a dataset | Choose Limited Data Set + DUA, Safe Harbor de-identification, or Expert Determination — see **phi-handling** |
| Patient requests their records | 30-day clock starts at receipt of request; electronic copy if requested and readily producible |
| Provider wants AI scribe transcribing visits | Vendor is a BA; execute BAA; assess minimum necessary, retention, training-data use |
| Going for HITRUST | See **hitrust-csf** — HITRUST CSF maps to HIPAA Security Rule plus other frameworks |

---

## Common Anti-Patterns

- Treating encryption as required and forgetting risk analysis (encryption is addressable; risk analysis is required)
- Calling a vendor a "conduit" to avoid a BAA when the vendor actually stores or has persistent access to PHI
- Citing TPO to justify a marketing communication
- Skipping the LoProCo documentation because "we don't think it's a breach" — without the analysis on file, OCR will treat it as a breach
- Ignoring state laws stricter than HIPAA (CA CMIA, TX HB 300, WA My Health My Data, etc.) — HIPAA is a floor, not a ceiling

---

## Output Format

When advising on a HIPAA question:

1. State the role (CE / BA / subcontractor BA / not regulated)
2. State which Rule(s) apply (Privacy, Security, Breach Notification)
3. State the specific safeguard/requirement implicated — name it, do not invent a citation if uncertain
4. Recommend the next concrete step (BAA, authorization, risk analysis update, OCR/state notification, counsel review)
5. Flag anything that requires privacy/compliance counsel or the Privacy Officer

Do not produce legal opinions. Refer the user to the current text of the regulation and to counsel for binding interpretation.

---

## Task-Specific Questions

1. Is the organization a Covered Entity, Business Associate, or subcontractor BA — and is the activity in question being performed in that role?
2. Which U.S. states and populations are in scope (e.g., CA CMIA, NY SHIELD/PHL, 42 CFR Part 2 SUD records, minors with state-specific minor-consent laws)?
3. What systems, data flows, and workforce roles touch PHI in this scenario, and which are inside vs. outside the legal entity?
4. When was the last Security Rule risk analysis completed and reviewed, and have material changes occurred since (new systems, vendors, M&A, modalities like telehealth)?
5. What is the current BAA inventory and refresh process — including subcontractor flow-downs and cloud service providers?
6. Has the Privacy/Security Officer or counsel already been engaged for this question, or do they need to be looped in before any external commitment is made?
7. What triggered this question — a new product or feature, an incident or near-miss, a vendor or M&A diligence, an OCR or auditor request, or routine compliance work?

---

## Related Skills

- **healthcare-context**: declares the HIPAA role and jurisdictions that determine which rules apply
- **phi-handling**: operational PHI controls, de-identification methods, encryption choice
- **healthcare-cybersecurity**: HHS 405(d) HICP practices, threat-aligned Security Rule implementation
- **audit-logging**: §164.312(b) audit controls and accounting of disclosures support
- **hitrust-csf**: HITRUST CSF as a HIPAA-mapped certification framework
- **gdpr-health-data**: when EU/EEA data is also in scope
- **21-cfr-part-11**: when systems also touch FDA-regulated electronic records
