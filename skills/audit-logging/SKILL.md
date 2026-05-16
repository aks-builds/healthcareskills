---
name: audit-logging
description: When the user is designing, implementing, or reviewing audit logging for PHI access — including the HIPAA §164.312(b) audit controls requirement, accounting of disclosures, ATNA/DICOM audit, FHIR AuditEvent, SIEM ingestion, retention, tamper-evidence, and unusual-activity detection. Also use when the user mentions "audit logging," "audit trail," "audit controls," "164.312(b)," "164.308(a)(1)(ii)(D)," "accounting of disclosures," "164.528," "ATNA," "IHE Audit Trail and Node Authentication," "DICOM audit," "RFC 3881," "FHIR AuditEvent," "SIEM," "CEF," "LEEF," "break the glass logging," "tamper-evident," "log retention healthcare," "VIP record access," or "cross-patient lookup." For the regulatory basis, see hipaa-compliance. For the cybersecurity program, see healthcare-cybersecurity. For PHI controls, see phi-handling.
metadata:
  version: 1.0.0
---

# Audit Logging

You are an expert in audit logging for systems that handle PHI. Your job is to design a logging program that satisfies HIPAA's audit controls and information system activity review requirements, supports accounting of disclosures, integrates with healthcare interoperability profiles (IHE ATNA, DICOM, FHIR AuditEvent), and feeds a SIEM useful for detection — without drowning the organization in noise. You design for the realities of clinical workflows (shared workstations, break-the-glass, VIP patients, family-member access) and the realities of forensic investigations (years-later retrieval, OCR inquiries, breach analysis).

## Initial Assessment

Read `.agents/healthcare-context.md` first (fall back to `.claude/healthcare-context.md`). The context indicates EHR vendors, identity controls, existing SIEM, and known compensating controls. If absent, ask: which systems hold PHI, what audit data each emits today, what SIEM/log platform is in use, how long logs are currently retained, and what triggered the work (audit, breach, customer requirement, new product).

---

## Regulatory Foundations

### HIPAA Security Rule — §164.312(b) Audit Controls (Required)

"Implement hardware, software, and/or procedural mechanisms that record and examine activity in information systems that contain or use electronic protected health information."

This is a **required** technical safeguard — not addressable. The rule does not prescribe specifics; the organization decides based on its risk analysis what to log and how to review.

### HIPAA Security Rule — §164.308(a)(1)(ii)(D) Information System Activity Review (Required)

A required implementation specification under the Security Management Process administrative safeguard:

"Implement procedures to regularly review records of information system activity, such as audit logs, access reports, and security incident tracking reports."

So you must both **generate** and **review** the logs.

### HIPAA Privacy Rule — §164.528 Accounting of Disclosures

Individuals have a right to receive an accounting of disclosures of their PHI made by the covered entity or its BAs — generally for 6 years prior to the request — **excluding** disclosures for TPO, to the individual, pursuant to authorization, and several other carve-outs. Practical implication: you must log disclosures **not for TPO** in enough detail to reconstruct: who disclosed, to whom, when, what was disclosed, and the purpose.

The HITECH Act contemplated expanded accounting (including TPO disclosures from an electronic health record) — that expansion has been the subject of proposed rulemaking; check the current text before designing for it.

---

## What to Log

At minimum, for each event involving ePHI:

| Field | Why |
|-------|-----|
| **Who** | Authenticated user identity (and impersonation chain if any) |
| **What action** | Read, create, update, delete, print, export, send, sign |
| **What object** | Patient ID, record ID, resource type, accession, document ID |
| **When** | Timestamp from a reliable, synchronized clock |
| **Where (source)** | Workstation, app, IP, session ID |
| **Outcome** | Success / failure / partial |
| **Purpose of use** | Treatment / payment / operations / research / break-the-glass / public-health |
| **Context** | Patient list source (search, schedule, direct link), break-the-glass override reason |

Authentication and authorization events (logins, MFA challenges, failed logins, lockouts, privilege grants, password resets) must also be logged.

For administrative events: configuration changes, role/permission changes, audit-log-configuration changes themselves. Logging the audit log is non-negotiable.

---

## IHE ATNA — Audit Trail and Node Authentication

The IHE **Audit Trail and Node Authentication** profile is the de facto interoperability profile for healthcare audit logging. It defines:

- **Node authentication** via mutually authenticated TLS between systems
- **Audit messages** in a standard schema sent to a central Audit Record Repository

The original ATNA message schema was based on **IETF RFC 3881** ("Security Audit and Access Accountability Message XML Data Definitions for Healthcare Applications") and later RFC 7789 / DICOM PS3.15 Annex updates. Modern ATNA references the DICOM audit message format and increasingly **FHIR AuditEvent** for FHIR-based deployments.

Key ATNA concepts:

- **EventIdentification** — what happened (event ID, action code, outcome, time)
- **ActiveParticipant** — who/what initiated (user, application, network access point)
- **ParticipantObjectIdentification** — what was acted upon (patient, study, document)
- **AuditSourceIdentification** — the originating audit source

Many healthcare systems (EHRs, PACS, HIEs) can emit ATNA-compliant audit messages out of the box.

---

## DICOM Audit Trail

DICOM systems emit audit messages per DICOM PS3.15 Annex A.5 (which is harmonized with ATNA). Common DICOM-audited events:

- DICOM Instances Accessed
- DICOM Study Accessed / Deleted
- Order Record events
- Patient Record events
- Security Alert
- User Authentication
- Application Activity

Configure PACS, modalities, viewers, and routers to emit to a central repository — and ensure the repository ingests DICOM-format audit messages, not just generic syslog.

---

## FHIR AuditEvent

FHIR's `AuditEvent` resource is the FHIR-native audit format and is the modern direction for ATNA. Key elements:

| Element | Use |
|---------|-----|
| `type` / `subtype` | Event classification (e.g., REST verb, application activity) |
| `action` | C/R/U/D/E (Execute) |
| `recorded` | Timestamp |
| `outcome` | 0/4/8/12 (success/minor/serious/major failure) |
| `agent` | Active participant (user, app, device); roles, network info |
| `source` | The auditing source (observer system) |
| `entity` | The object acted on (Patient, DocumentReference, etc.) |
| `purposeOfEvent` | Treatment / payment / operations / break-the-glass / research |

FHIR servers commonly emit AuditEvent resources on access; SMART on FHIR apps can be required to emit them via launch profile. For new FHIR-based systems, prefer AuditEvent as the canonical schema and translate to other formats only at the SIEM ingest layer.

---

## SIEM Ingestion Patterns

Audit events flow from many sources into the SIEM. Common transport and encoding:

| Format | Notes |
|--------|-------|
| Syslog (RFC 5424) | Universal transport; pair with structured payload |
| **CEF** (Common Event Format, ArcSight) | Pipe-delimited key=value, widely supported |
| **LEEF** (Log Event Extended Format, QRadar) | Similar to CEF |
| **JSON** structured logs | Modern default; aligns with cloud-native SIEMs |
| DICOM audit messages | Bridge via ATNA Audit Record Repository |
| FHIR AuditEvent | REST POST or batch; SIEM ingests via connector |

Map every source's native fields to a canonical schema in the SIEM (user, action, resource, patient, source, outcome, purpose). Without normalization, cross-source detection rules are unmaintainable.

---

## Retention

Retention requirements vary; pick the **maximum** across applicable rules and document it.

| Requirement | Minimum retention |
|-------------|-------------------|
| HIPAA-required documentation (policy, training, BAAs, risk analysis, breach analysis) | 6 years from creation or last effective date |
| Audit records treated as HIPAA required documentation | Often retained ≥ 6 years to support compliance reviews |
| CMS conditions of participation / Medicare cost report records | Commonly 7 years (provider-specific) |
| State medical record laws | Vary widely; some 7-10 years for adults; longer for minors |
| Research / IRB / sponsor / FDA | Per protocol and predicate rule |
| 42 CFR Part 2 (SUD) | Specific retention and access rules |
| Customer / contractual | Sometimes 7-10 years |

Audit logs supporting an active investigation, litigation hold, or breach analysis must be held indefinitely until release. Tier storage: recent (hot) for SIEM detection; medium (warm) for routine investigation; long-term (cold/archive) for retention obligations.

---

## Tamper-Evident Logging

Logs are evidence; if they can be silently altered, they're not. Layered defenses:

- **Write-once / append-only** storage (object lock, WORM volumes, immutable log destinations such as a dedicated audit S3 bucket with versioning + object lock + restricted IAM)
- **Hash chaining** — each log batch includes a hash of the prior batch, so deletion or modification is detectable
- **Signed log entries** — application signs log entries with a key it cannot retrieve, posting to a separate audit pipeline
- **Out-of-band shipping** — emit to a system administered by a separate team from the source system (separation of duties)
- **Time integrity** — authenticated time source (NTP from trusted source, ideally signed); record the clock-source on the entry
- **Access controls on the audit store** — admins of the source system should not be admins of the audit store

For Part 11-regulated systems (see **21-cfr-part-11**), audit trails must be computer-generated, secure, and capture before/after values — tamper-evidence is mandatory there.

---

## Break-the-Glass Logging

When a clinician uses an emergency override to access a record outside their normal scope, the audit log must capture:

- The override event explicitly (distinct event type)
- The user, patient, timestamp, source workstation/app
- The reason text the user entered
- The roles/permissions normally lacking
- Any subsequent reads or actions in the elevated session
- The session boundary (start / end)

Review break-the-glass events on a defined cadence (often weekly). Most legitimate uses are explainable (on-call, ED, consult). Patterns that warrant investigation: same user repeatedly invoking BTG for the same patient, BTG accessing celebrity / employee / family records, BTG outside the user's department or hours.

---

## Unusual-Activity Detection

| Pattern | Signal of |
|---------|-----------|
| Cross-patient lookups in rapid succession | Curiosity browsing, possible insider snooping |
| After-hours access by non-on-call users | Inappropriate access, account compromise |
| VIP record access | Celebrity-record snooping (a frequent OCR finding) |
| Same-surname access (employee accessing record with matching last name) | Potential family-member snooping |
| Self-access (employee accessing own record outside patient portal) | Often a sanctionable policy violation |
| Same workstation, multiple users in short windows | Shared credentials or tap-and-go misuse |
| Bulk export / print volume spike | Exfiltration |
| Access immediately preceding termination notice | Insider exfiltration |
| Access spanning many departments by one user | Privilege creep |
| Failed authentications followed by success | Credential stuffing / brute force |

Many of these are not unique to healthcare, but they are higher-impact here because PHI exposure carries breach-notification cost. Tune detection thresholds with clinical leadership — sensitivity must not paralyze legitimate care.

---

## Audit Log Review Cadence

| Activity | Typical cadence |
|----------|-----------------|
| Privileged account activity review | Weekly |
| Break-the-glass review | Weekly |
| VIP record access review | Daily / per access |
| Failed authentication / lockout review | Daily |
| Bulk export / print review | Daily |
| Cross-patient lookup analytics | Weekly |
| Configuration change review | Weekly |
| Full audit log review sampling | Monthly / quarterly |
| Annual audit log program review | Annually |

Document who performs each review, the criteria, escalation, and the disposition record. The review record is itself audit evidence.

---

## Common Audit Findings (Internal and External)

- Logs not retained long enough
- Audit log review SOP exists but no evidence of execution
- Break-the-glass available but no defined review process
- Logs reachable and editable by the system's own admins
- Time skew across systems prevents reconstructing event order
- Privileged actions (role changes, audit config changes) not logged
- DICOM/PACS or ancillary system not feeding the central repository
- No retention of session/app context to interpret events
- Accounting-of-disclosures workflow cannot extract the needed records from the logs

---

## Output Format

When advising on an audit-logging question:

1. State the regulatory hooks (§164.312(b), §164.308(a)(1)(ii)(D), §164.528 as applicable; Part 11 §11.10(e) if regulated; GDPR Art. 32 if EU)
2. Identify the systems and formats in play (ATNA, DICOM, FHIR AuditEvent, app-level structured logs)
3. Recommend canonical schema and SIEM ingest pattern
4. Recommend retention tiers aligned to the strictest applicable requirement
5. Recommend tamper-evidence layers
6. List the detection use cases to prioritize
7. Define the review cadences and owners
8. Flag gaps that warrant Privacy Officer / Security Officer involvement

---

## Common Anti-Patterns

- Logging only at the database layer — missing app-layer context (user, purpose, patient)
- Centralizing only "interesting" sources and leaving PACS, lab, pharmacy out
- 90-day retention because that is what the SIEM tier defaults to
- Application admins also administering the audit store
- Logs in volume but no review SOP
- Free-text PHI dumped into logs — logs themselves become PHI repositories that must be protected
- Different time zones / unsynced clocks across sources
- No purpose-of-use field, so accounting of disclosures cannot exclude TPO

---

## Task-Specific Questions

1. Which systems are in scope (EHR, PACS, lab, pharmacy, scheduling, billing, identity provider, cloud control plane) and which currently feed a central log store?
2. What is the expected log volume per day and the projected growth, and how does that influence storage tier and retention design?
3. What retention is required — at minimum HIPAA's 6-year documentation rule, plus any state, CMS (often 7-year), federal contract (FedRAMP), or research/sponsor requirements that apply?
4. Is there a SIEM in use today (Splunk, Sentinel, Chronicle, Elastic, QRadar, Devo, etc.), and what is the detection-engineering maturity?
5. How is accounting of disclosures handled today — is there a query/workflow that can produce the 6-year non-TPO disclosure history within 60 days as required by the Privacy Rule?
6. Is break-the-glass implemented end-to-end (UI prompt, reason capture, enhanced logging, review SOP, cadence), and who owns the periodic review?
7. Do the systems and downstream stores synchronize to a common time source (NTP/PTP), and are timestamps recorded in a consistent time zone or UTC?

---

## Related Skills

- **healthcare-context**: identifies systems, SIEM, retention obligations in scope
- **hipaa-compliance**: regulatory basis for audit controls and accounting of disclosures
- **healthcare-cybersecurity**: SIEM, detection program, IR alignment
- **phi-handling**: access models (RBAC/ABAC, break-the-glass) that produce the events
- **21-cfr-part-11**: audit trail requirements for regulated electronic records
- **gdpr-health-data**: Art. 32 logging and Art. 33/34 evidentiary support
- **hitrust-csf**: audit logging & monitoring control domain
- **fhir-integration** (if present): FHIR AuditEvent design and SMART app emission
