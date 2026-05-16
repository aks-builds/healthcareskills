# 21 CFR Part 11 Audit Trail Requirements

A working reference for the audit trail expectations under 21 CFR Part 11 §11.10(e) and related FDA data integrity guidance (ALCOA+). Verify the current text of Part 11 and the most recent FDA data integrity guidance (the "Data Integrity and Compliance With Drug CGMP" guidance and equivalent for devices and clinical) — language and expectations evolve. Consult QA / Validation / Regulatory Affairs for binding interpretation.

## What §11.10(e) Requires

§11.10(e) requires that closed systems used to create, modify, maintain, or transmit electronic records have procedures and controls to ensure: **use of secure, computer-generated, time-stamped audit trails to independently record the date and time of operator entries and actions that create, modify, or delete electronic records.** Record changes shall not obscure previously recorded information. The audit trail documentation shall be **retained for a period at least as long as that required for the subject electronic records** and shall be available for agency review and copying.

This is the **substantive** requirement. The rule does NOT prescribe a specific format, schema, file type, or storage medium. FDA does NOT publish a "standard schema" for Part 11 audit trails — design is the regulated user's responsibility, subject to validation and inspection.

## Audit Trail Content Elements

A defensible audit trail captures, at minimum, for every recorded action:

| Element | Purpose |
|---|---|
| Date and time (with time zone, ideally UTC) | When the action happened — must be computer-generated, not user-supplied |
| User identifier | Who performed the action — unique identifier, no shared accounts |
| Action / event type | What was done (create / read / modify / delete / sign / etc.) |
| Record / object identifier | Which record was acted on |
| Field-level before/after values | For modifications — old value and new value |
| Reason for change | Where required by predicate rule, organizational policy, or for clinically significant changes |
| Authentication context | Authentication method used (in some implementations) |
| Source / system identifier | Which system / module produced the event (for federated audit) |
| Session / transaction identifier | Correlation across multi-step actions |

For e-signatures specifically (§11.50 / §11.70), the signed record must include the printed name of the signer, the date and time, and the meaning of the signing (review, approval, responsibility), and these must be subject to the same controls as the underlying record.

## ALCOA+ Principles

FDA data integrity guidance frames audit trail expectations using ALCOA+:

| Letter | Principle |
|---|---|
| A | **Attributable** — traceable to the individual or system that generated it |
| L | **Legible** — readable, permanent, durable |
| C | **Contemporaneous** — recorded at the time of the activity, not after the fact |
| O | **Original** — the first capture (or true copy thereof) |
| A | **Accurate** — correct, complete, free from errors and editing |
| + | **Complete, Consistent, Enduring, Available** — all four extensions |

An audit trail that fails on any of these (e.g., timestamps set by the user, gaps where edits weren't logged, log entries that can be back-dated, retention shorter than the predicate record) is a defect.

## Technical Controls

### Time Synchronization
- All systems contributing to the audit trail must synchronize to a common authoritative time source (NTP or PTP)
- Time zone should be consistent across systems (UTC strongly preferred for storage; UI can localize)
- Clock drift, summer-time changes, and post-event time corrections must not corrupt past entries

### Immutability / Tamper Resistance
- Audit log records must not be modifiable by application users
- Database-level controls (append-only tables, immutable storage, write-once-read-many storage)
- Separation of duties — application admins are NOT audit-log admins
- Hashing / chaining for tamper-evidence in higher-risk applications

### Storage and Retention
- Retention must meet the predicate rule's retention period for the underlying record (clinical trial records, batch records, study records — durations vary)
- Backups and archives must preserve the audit trail with the same integrity properties
- Migration of audit trails during system retirement must be validated (the migrated trail is a "true copy" of the original)

### Access and Review
- Audit-trail viewing access for those who need it (QA, auditors, investigators)
- Periodic audit-trail review SOP — define what gets reviewed, by whom, on what cadence
- Documentation of review (who reviewed, when, what was found, what was actioned)

## Common Audit Trail Failures (FDA 483 Themes)

Drawn from publicly observed FDA inspection findings (verify specifics against published 483s and warning letters):

- **Disabled audit trail** — system has the feature but it was turned off
- **User-editable timestamps** — operator could backdate entries
- **Shared accounts** — no attributability
- **Audit trail not retained per predicate rule** — purged early, or retention not set
- **No periodic audit-trail review** — feature exists, SOP absent
- **Audit trail only at the database layer** — missing the application/user-action context
- **Reason for change not captured** for changes that materially affect the record
- **Audit trail not preserved through system migration / retirement**
- **Excel-as-record** — Excel files used as regulated records without audit trail integrity
- **Test results re-runnable until acceptable** — laboratory data integrity failures, audit trail missing or showing the pattern

## Open vs. Closed System Considerations

§11.30 imposes additional controls on open systems (where access is not under the control of the entity responsible for the records — typically systems that traverse the internet to an external party). Open systems require additional measures to ensure the authenticity, integrity, and as appropriate, confidentiality of records — often including digital signatures and encryption. The audit-trail integrity expectations remain.

Most modern enterprise SaaS for regulated use is treated as a **closed system** because the regulated organization controls authentication and access (via federated identity, BAA, contractual controls).

## E-Signature Audit Trail Specifics

For e-signatures:
- The signing event must be captured with the user, date, time, and meaning of the signing
- The signature must be inextricably linked to the signed record (re-signing on copy, embedded in PDF, equivalent)
- Two-component signature (§11.200) — at minimum, an identification code and a password, OR a biometric — with the second component required for sessions that have lapsed beyond a defined interval
- One-time certification letter (§11.100(c)) to FDA stating intent to use e-signatures as legally binding equivalent of handwritten signatures

## Documentation Expectations

For an inspection-ready audit trail program:

- Validation evidence that the audit trail captures the required content
- Documented SOPs for audit-trail review, deviation handling, and system retirement
- Training records for users on the audit trail's purpose
- Records of periodic audit-trail reviews
- Evidence the audit trail has been preserved through any migration or system change

## Final Disclaimers

- Verify the current text of Part 11 §11.10(e), §11.50, §11.70, §11.100, §11.200, §11.300
- Verify current FDA data integrity guidance (CGMP and equivalent for devices and clinical)
- Cross-reference the audit-logging skill for the technical implementation patterns common to healthcare (FHIR AuditEvent, ATNA, retention)
- Consult QA / Validation / Regulatory Affairs for binding interpretation
