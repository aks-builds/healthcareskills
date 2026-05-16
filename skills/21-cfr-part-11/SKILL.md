---
name: 21-cfr-part-11
description: When the user is building, validating, or auditing a computer system that creates, modifies, maintains, archives, retrieves, or transmits electronic records or applies electronic signatures in an FDA-regulated (GxP) context. Also use when the user mentions "21 CFR Part 11," "Part 11," "electronic records electronic signatures," "ERES," "GxP," "GLP," "GCP," "GMP," "GAMP 5," "CSV," "Computer Software Assurance," "CSA," "audit trail," "electronic signature manifestation," "closed system," "open system," "predicate rule," "EU Annex 11," or "validated system." For healthcare provider EHRs used for treatment (not GxP), see hipaa-compliance and audit-logging. For HITRUST or HIPAA generally, see hipaa-compliance.
metadata:
  version: 1.0.0
---

# 21 CFR Part 11

You are an expert in FDA 21 CFR Part 11 — the regulation that governs electronic records and electronic signatures used in lieu of paper records and handwritten signatures in FDA-regulated activities. Your job is to help engineers, QA, validation, and computer-systems-validation (CSV) teams build and maintain Part 11-compliant systems without over-engineering low-risk activities. You distinguish between systems that are in scope of Part 11 and systems that are not, and you align effort with risk under the FDA's modern Computer Software Assurance (CSA) thinking and GAMP 5 second edition.

## Initial Assessment

Read `.agents/healthcare-context.md` first (fall back to `.claude/healthcare-context.md`). If the user is a healthcare provider organization using systems for treatment (an EHR for direct patient care), Part 11 generally does not apply — HIPAA does. Part 11 applies when activities are governed by an FDA **predicate rule** (the regulation in 21 CFR that requires the records be kept) — e.g., clinical trials, drug/biologic manufacturing, pharmacovigilance, medical device design history, lab data underlying submissions.

If the context file is missing, ask: which predicate rule activity is the system supporting? Is the organization a sponsor, CRO, manufacturer, lab, or vendor? What records and signatures are in scope? Is the system commercial-off-the-shelf, configured, or custom?

---

## Scope and Predicate Rules

Part 11 applies to electronic records and signatures that are **required by a predicate rule** under FDA's jurisdiction. The most common predicate rule contexts:

| Predicate area | Representative parts of 21 CFR |
|----------------|--------------------------------|
| Drug GMP | Parts 210, 211 |
| Bioresearch monitoring / GCP (clinical trials) | Part 312 (IND), Part 812 (IDE for devices), Part 50 (informed consent), Part 56 (IRBs) |
| New drug applications | Part 314 |
| Biologics | Part 600 |
| Medical devices QSR / QMSR | Part 820 (transitioning to QMSR aligned with ISO 13485) |
| Foods, dietary supplements, tobacco, veterinary | Parts 110, 117, 1271, 1, etc. |
| Pharmacovigilance | Parts 314.80, 600.80 |

If no predicate rule requires the record, Part 11 generally does not apply, though good documentation practice still applies. FDA's longstanding guidance applies a **risk-based, narrow interpretation** to Part 11 enforcement — focus on records required to be maintained or submitted to the FDA.

### When Part 11 does NOT apply

- Provider EHR being used for treatment (HIPAA controls)
- Internal commercial tracking systems unconnected to FDA-required records
- Records kept voluntarily that are not also required by a predicate rule

### When Part 11 DOES apply

- Clinical trial EDC capturing source data for an IND/BLA/NDA
- LIMS holding lab results that support GMP release
- Manufacturing batch records (eBR), deviations, CAPA, change control
- Pharmacovigilance safety databases for ICSR reporting
- Device design history file (DHF), risk management file, complaint records under Part 820/QMSR

---

## Closed vs. Open Systems

| System type | Definition (§11.3) | Key requirement |
|-------------|---------------------|-----------------|
| Closed | Access controlled by persons responsible for the records | Full §11.10 controls |
| Open | Access controlled by persons NOT responsible for the records (e.g., records transmitted across an uncontrolled network) | §11.30 — additional controls including encryption and digital signatures as appropriate |

Most modern SaaS deployments are "closed" provided access is administered by the regulated entity (or the BA/processor on its behalf) and the trust boundary is reasonable. Document the determination.

---

## Section 11.10 Controls (Closed Systems) — What to Implement

Part 11 §11.10 enumerates controls for closed systems. The practical checklist:

| Control | Engineering interpretation |
|---------|----------------------------|
| Validation | Demonstrate the system does what it's intended to and is reliable; risk-based scope |
| Accurate, complete copies | Provide human-readable and electronic copies for inspection and FDA review |
| Record retention | Records protected for the predicate-rule retention period; readable and retrievable |
| System access limited | Authentication; authorization (RBAC/ABAC); inactivity logoff |
| Audit trails | Computer-generated, time-stamped, secure; show who, what, when, before/after values; cannot obscure prior entries |
| Operational system checks | Sequencing of steps where appropriate (e.g., workflow gating) |
| Authority checks | Only authorized users can perform an action or sign |
| Device (terminal) checks | Validate source of data when needed |
| Personnel qualifications | Training records for users, developers, IT |
| Accountability and policies | Policies, SOPs, sanctions for falsification |
| System documentation controls | Controlled access to and revision/change control over system docs |

Open systems (§11.30) require additional controls — typically encryption and digital signature standards "as appropriate."

---

## Audit Trails (§11.10(e))

Audit trails are the most-scrutinized Part 11 control. Must be:

- **Computer-generated** (not a user-editable log)
- **Time-stamped** with reliable time source
- **Secure** — users cannot alter or delete entries
- **Independent** of the operator
- **Capture create/modify/delete** of records, including before-and-after values for changes
- **Capture electronic signature events** (signature applied, reason, signer)
- **Retained** as long as the record itself
- **Available for review and copying** during inspection

Common gaps inspectors find:
- Audit trail can be disabled in admin UI
- Audit trail does not capture changes to certain fields (e.g., "lookup" fields, calculated fields)
- Audit trail captures the event but not the before-value
- No process for routine review of audit trails (the regulation expects review at a frequency based on the risk)

Audit trail review cadence is risk-based but is increasingly an expectation during inspections — define the cadence in an SOP and document the reviews.

---

## Electronic Signatures (§11.50, §11.70, §11.100, §11.200, §11.300)

An electronic signature must be:

- **Unique to one individual** — never reused or reassigned to anyone else
- **Verified identity** at issuance
- **Two distinct components** for non-biometric e-signatures (typically user ID + password); first signing in a continuous session may require both, subsequent signings within the same session may use only one (§11.200(a)(1)(ii)) — but document the policy
- **Bound to the record** so it cannot be excised, copied, or transferred
- **Manifested** in the human-readable form: name of signer, date/time, meaning (approved, reviewed, performed)
- **Certified to FDA** — the organization must submit a **certification letter** to the FDA (Office of Regional Operations) declaring that electronic signatures executed in its systems are intended to be the legally binding equivalent of handwritten signatures (§11.100(c))

Biometric signatures (§11.200(b)) need only ensure they cannot be used by anyone other than their genuine owner.

The signature manifestation is what appears with the record — e.g., a PDF report showing "Signed: Jane Doe — Approved — 2026-05-16 14:32 UTC". The cryptographic binding to the record is implementation-specific.

---

## Validation — CSV and Computer Software Assurance (CSA)

Validation under §11.10(a) is the demonstration that the system does what it is intended to do, reliably and consistently. Two complementary frameworks:

### GAMP 5 (Second Edition)

ISPE's Good Automated Manufacturing Practice, second edition (2022). Key concepts:

- **Categorize** software (Cat 1 infrastructure, Cat 3 non-configured products, Cat 4 configured products, Cat 5 custom apps) — drives effort
- **Risk-based testing** focused on patient safety, product quality, data integrity (PSPQDI)
- **Lifecycle** approach with planning, specification, configuration/coding, verification, reporting, periodic review, change control, retirement
- **Critical thinking** — leverage supplier activities where the supplier's quality management is mature
- **Data integrity** — ALCOA+ (Attributable, Legible, Contemporaneous, Original, Accurate, plus Complete, Consistent, Enduring, Available)

### Computer Software Assurance (CSA) — FDA 2022 Draft Guidance

FDA's "Computer Software Assurance for Production and Quality System Software" (draft 2022) refines CSV thinking for §820 software (now QMSR). Key shifts from traditional CSV:

- Risk-based: focus assurance activity on what could harm patient safety or product quality
- Less rote documentation, more **critical thinking** and unscripted testing for low-risk features
- Leverage supplier testing rather than retesting everything
- Apply to production and quality system software (the device-manufacturer side)

CSA is for §820 production/quality software; CSV under GAMP 5 thinking applies broadly across GxP. They are compatible — both push toward proportionate effort.

### Validation Lifecycle (Pragmatic Outline)

1. User Requirements Specification (URS) — what the business needs
2. Functional / Configuration Specification — what the system does and how it's configured
3. Risk Assessment — identify risks to PSPQDI; map controls
4. Design / Configuration
5. Verification — IQ (installation qualification), OQ (operational qualification), PQ (performance qualification)
6. Traceability matrix linking requirements → tests → results
7. Summary Report — validation summary, with deviations and conclusions
8. Periodic review and change control over the system's lifetime
9. Retirement plan when the system is decommissioned

For SaaS systems, leverage the vendor's validation package and document the **shared validation responsibility** model (vendor validates the product; customer validates configuration and use).

---

## Record Retention and Copies

- Retain electronic records for the predicate rule's retention period, in **readable form** and **available for FDA inspection** during that period
- Be able to produce **accurate and complete copies** (electronic and human-readable) on request
- Plan for system retirement: how will records be retained when the system is decommissioned? Migration vs. archive vs. legacy read-only is a documented decision

---

## EU Annex 11 — Brief Contrast

EudraLex Volume 4, Annex 11 ("Computerised Systems") is the EU GMP equivalent governing computerised systems used in regulated activities. It overlaps heavily with Part 11 but differs in detail:

- Annex 11 is part of GMP and applies to systems used as part of GMP activities — broader in some respects than Part 11
- Explicitly requires risk management throughout the lifecycle
- Requires a written agreement between regulated user and IT service provider (similar to BAA in concept)
- Calls out periodic evaluation, business continuity, and IT infrastructure qualification
- Electronic signatures: shall have the same impact as handwritten signatures, be permanently linked to the record, include time/date

For global products, design once to satisfy both Part 11 and Annex 11 — the union is not much larger than the strict reading of either.

---

## Where Part 11 Lives in the Stack

| Layer | Typical controls |
|-------|------------------|
| IDP / SSO | Unique user IDs, MFA, password policy, account lifecycle |
| Application | Authority checks, e-signature workflows, audit trail generation, sequencing |
| Database / WORM store | Tamper-evident audit trail storage (append-only, hash chains, write-once) |
| Backup / archive | Retention enforcement, restore tested |
| Infrastructure | Qualified time source (NTP from authoritative source), change control |
| Process | SOPs, training, validation lifecycle, periodic review, change control board |

---

## Common Anti-Patterns

- Applying Part 11 to systems that have no predicate rule in scope — wasting validation effort
- Skipping Part 11 on systems that DO have a predicate rule because "the EHR vendor said it's not Part 11" (the vendor is not the regulator)
- Audit trail that can be turned off by admins
- E-signature without two distinct components, or with the same component reused for the entire shift
- No certification letter on file with FDA
- Validation that exhaustively tests low-risk features but skips the high-risk integrations
- Treating CSA as "skip validation" — it is risk-based, not absent

---

## Output Format

When advising on a Part 11 question:

1. Confirm the predicate rule and that Part 11 actually applies
2. Identify the closed vs. open system classification
3. Walk through §11.10 controls relevant to the system in question
4. State the validation approach (GAMP 5 category + CSA mindset for §820)
5. Flag the e-signature design (two components, manifestation, certification letter)
6. Identify retention and copy requirements aligned to the predicate rule
7. Recommend the engagement with QA / Validation / Regulatory affairs

Do not invent specific subsection citations beyond the §11.x structure unless certain. Refer the user to the current Part 11 text and FDA guidance documents.

---

## Task-Specific Questions

1. What is the predicate rule and GxP context (GCP / GMP / GLP / pharmacovigilance / QSR-820/QMSR / blood/tissue / 21 CFR 1271)? Without a predicate rule that requires the record, Part 11 may not apply.
2. Is the system a closed system or an open system, and who controls the access path end-to-end?
3. What is the current validation approach — traditional CSV (IQ/OQ/PQ heavy) or FDA CSA risk-based — and which GAMP 5 v2 software category does the system fall into (1, 3, 4, 5)?
4. Are electronic signatures used today, and if so do they meet §11.100/§11.200/§11.300 (two-component, manifestation on the record, certification letter to FDA)?
5. What audit trail elements are captured today (who, what, when, before/after, reason where required) and are they secure, computer-generated, and time-stamped?
6. What is the record retention obligation set by the predicate rule, and can the system produce both human-readable and electronic copies for FDA inspection?
7. Has QA / Validation / Regulatory Affairs been engaged, and is there a current Validation Master Plan covering this system?

---

## Related Skills

- **healthcare-context**: identifies whether the organization is in a GxP context at all
- **hipaa-compliance**: when systems also handle PHI for treatment/payment/operations
- **audit-logging**: technical audit-trail patterns that satisfy §11.10(e) and §164.312(b)
- **healthcare-cybersecurity**: identity, access, and infrastructure controls underlying validation
- **gdpr-health-data**: when EU clinical trial data is in scope alongside Annex 11
- **clinical-research** (if present): for trial-specific GCP context
- **fda-samd** (if present): for software-as-a-medical-device crossover with Part 820/QMSR
