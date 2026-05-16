# DICOM Audit Message Format and IHE ATNA

A working reference for the legacy DICOM Audit Message format (used by IHE ATNA for many years over syslog) and the modern ATNA-on-FHIR evolution. Verify against the current DICOM PS3.15 Annex A (Audit Message Schema) and the current IHE ITI Technical Framework (ATNA profile and the Basic Audit Log Patterns / BALP supplement). Examples below use synthetic identifiers. Consult your interoperability lead for binding decisions.

## What ATNA Is

**ATNA (Audit Trail and Node Authentication)** is an IHE ITI profile that addresses two interlocking concerns:

1. **Node authentication** — bidirectional TLS / mTLS between healthcare nodes so only authenticated systems communicate
2. **Audit logging** — every security-relevant event is logged in a standardized format to a centralized Audit Record Repository

Together, these establish a "trusted node" baseline for interoperability between healthcare systems.

## ATNA Actors

| Actor | Role |
|---|---|
| Secure Node | A node that authenticates other nodes via TLS, audits all PHI access, and reports audit events |
| Secure Application | A subset of Secure Node functionality for applications inside a trusted environment |
| Audit Record Repository | Receives audit messages from Secure Nodes and stores them |
| Audit Record Forwarder | Forwards audit messages (often used in distributed architectures) |
| Time Server / Time Client | NTP-based time synchronization (ATNA depends on synchronized time) |

## The DICOM Audit Message (Legacy)

The historical ATNA audit format is the DICOM Audit Message, defined in DICOM PS3.15 Annex A. It is an XML schema with a clear structure mapping to the actors and events of an audit-worthy interaction.

### Top-Level Structure

```
AuditMessage
├── EventIdentification          (1)
│   ├── EventID                  (coded — what event)
│   ├── EventActionCode          (C / R / U / D / E)
│   ├── EventDateTime            (ISO 8601)
│   ├── EventOutcomeIndicator    (0=Success, 4=Minor failure, 8=Serious failure, 12=Major failure)
│   ├── EventTypeCode            (refines EventID)
│   └── EventOutcomeDescription  (optional human-readable)
├── ActiveParticipant            (1..N)
│   ├── UserID                   (the user / system identifier)
│   ├── AlternativeUserID        (network address, process ID, etc.)
│   ├── UserName                 (display name)
│   ├── UserIsRequestor          (true/false)
│   ├── RoleIDCode               (the role the participant played)
│   ├── NetworkAccessPointID     (IP, hostname)
│   ├── NetworkAccessPointTypeCode
│   └── MediaIdentifier / MediaType (for media events)
├── AuditSourceIdentification    (1)
│   ├── AuditSourceID            (the system that emitted the audit)
│   ├── AuditEnterpriseSiteID
│   └── AuditSourceTypeCode
└── ParticipantObjectIdentification (0..N)
    ├── ParticipantObjectID        (the object — patient, study, report)
    ├── ParticipantObjectTypeCode  (1=Person, 2=System Object, 3=Organization, 4=Other)
    ├── ParticipantObjectTypeCodeRole
    ├── ParticipantObjectDataLifeCycle
    ├── ParticipantObjectIDTypeCode
    ├── ParticipantObjectName
    ├── ParticipantObjectQuery
    └── ParticipantObjectDetail
```

### Illustrative — Patient Record Access (Synthetic)

```xml
<AuditMessage>
  <EventIdentification
    EventActionCode="R"
    EventDateTime="2026-03-14T14:22:08Z"
    EventOutcomeIndicator="0">
    <EventID csd-code="110112" codeSystemName="DCM" originalText="Query"/>
    <EventTypeCode csd-code="ITI-18" codeSystemName="IHE Transactions" originalText="Registry Stored Query"/>
  </EventIdentification>

  <ActiveParticipant
    UserID="synthetic-practitioner-001"
    AlternativeUserID="session=abc123"
    UserName="Smith, John MD"
    UserIsRequestor="true"
    NetworkAccessPointID="10.20.30.40"
    NetworkAccessPointTypeCode="2">
    <RoleIDCode csd-code="110150" codeSystemName="DCM" originalText="Application User"/>
  </ActiveParticipant>

  <AuditSourceIdentification
    AuditSourceID="EHR-Server-01"
    AuditEnterpriseSiteID="HospitalA"/>

  <ParticipantObjectIdentification
    ParticipantObjectID="synthetic-patient-001"
    ParticipantObjectTypeCode="1"
    ParticipantObjectTypeCodeRole="1">
    <ParticipantObjectIDTypeCode csd-code="2" originalText="Patient Number"/>
  </ParticipantObjectIdentification>
</AuditMessage>
```

### Illustrative — DICOM Study Retrieve (Synthetic)

```xml
<AuditMessage>
  <EventIdentification
    EventActionCode="R"
    EventDateTime="2026-03-14T14:25:11Z"
    EventOutcomeIndicator="0">
    <EventID csd-code="110107" codeSystemName="DCM" originalText="Import"/>
    <EventTypeCode csd-code="ITI-43" codeSystemName="IHE Transactions" originalText="Retrieve Document Set"/>
  </EventIdentification>

  <ActiveParticipant
    UserID="PACS-Workstation-12"
    UserIsRequestor="true"
    NetworkAccessPointID="10.20.40.55"
    NetworkAccessPointTypeCode="2">
    <RoleIDCode csd-code="110153" codeSystemName="DCM" originalText="Source Role ID"/>
  </ActiveParticipant>

  <AuditSourceIdentification
    AuditSourceID="PACS-Server-02"/>

  <ParticipantObjectIdentification
    ParticipantObjectID="1.2.840.113619.2.55.3.synthetic.example"
    ParticipantObjectTypeCode="2"
    ParticipantObjectTypeCodeRole="3">
    <ParticipantObjectIDTypeCode csd-code="12" originalText="Study Instance UID"/>
  </ParticipantObjectIdentification>

  <ParticipantObjectIdentification
    ParticipantObjectID="synthetic-patient-001"
    ParticipantObjectTypeCode="1"
    ParticipantObjectTypeCodeRole="1">
    <ParticipantObjectIDTypeCode csd-code="2" originalText="Patient Number"/>
  </ParticipantObjectIdentification>
</AuditMessage>
```

## Transport — Historically Syslog

ATNA traditionally delivers DICOM Audit Messages over **syslog** (RFC 5424 with TLS per RFC 5425, or sometimes UDP for in-trusted-network deployments — verify current IHE ATNA transport options). The audit message is the syslog message body; the syslog headers carry the facility, severity, and timestamp envelope.

## ATNA on FHIR / BALP (Modern)

IHE has more recently published the **Basic Audit Log Patterns (BALP)** supplement, which defines how FHIR AuditEvent resources represent the same security-relevant events historically captured in the DICOM Audit Message. BALP defines patterns for common event types (read, create, update, delete, login, query, break-the-glass) using the FHIR AuditEvent resource (see fhir-auditevent.md).

Modern deployments typically:

- Use FHIR AuditEvent for newer FHIR-native systems
- Continue to use DICOM Audit Message + syslog for legacy DICOM / IHE XDS systems
- Centralize both into the Audit Record Repository (SIEM) with a normalization layer

## Event Code Systems

Common code systems referenced in ATNA audit messages:

| System | Use |
|---|---|
| DICOM (DCM) | Event IDs, role IDs, participant object roles |
| IHE Transactions | Specific IHE transaction codes (e.g., ITI-18 Registry Stored Query, ITI-41 Provide and Register, ITI-43 Retrieve Document Set) |
| HL7 v3 ActReason | Purpose-of-use codes (TREAT, PAYMENT, OPERATIONS, ETREAT for emergency treatment, etc.) |
| HITSP / FHIR audit-event-type | Modern FHIR AuditEvent type and subtype |

Verify the current value sets in the IHE ITI Technical Framework and the FHIR specification.

## Audit Record Repository Considerations

- Capacity planning — DICOM Audit Messages are larger than typical syslog events; volumes scale with imaging activity
- Indexing — store the patient ID, study UID, user ID, event type, timestamp for efficient query
- Retention — per HIPAA documentation, state law, CMS, and study-specific requirements (see retention-by-jurisdiction.md)
- Integrity — append-only / tamper-evident storage
- Separation of duties — the Audit Record Repository admin is not the application admin
- Time synchronization — all sources synced to a common time server (ATNA's Time Client/Server actors)

## Common Pitfalls

- Sending audit messages over plain UDP syslog without TLS — defeats ATNA's confidentiality
- Including free-text PHI in event descriptions — turns the audit log itself into a PHI repository to be protected
- Different time zones across emitting systems — store UTC, render in local time
- No purpose-of-use code on access events — accounting of disclosures cannot filter TPO
- Treating DICOM Audit Message and FHIR AuditEvent as interchangeable without a normalization layer — they have different schemas and code systems

## Final Disclaimers

- Verify against the current DICOM PS3.15 Annex A (Audit Message Schema)
- Verify against the current IHE ITI Technical Framework (ATNA profile, BALP supplement)
- Verify current syslog transport options for ATNA
- Consult your interoperability lead and Privacy/Security Officer for binding design decisions
