# FHIR AuditEvent Resource

A working reference for the HL7 FHIR AuditEvent resource and its use in healthcare audit logging. Verify against the current FHIR specification (the resource structure has evolved across R3/R4/R5, and FHIR R5 reorganized parts of the audit model). The examples below are illustrative — they use synthetic identifiers, not real PHI. Consult the FHIR specification and your interoperability lead for binding decisions.

## What AuditEvent Is

AuditEvent is the FHIR resource for representing the record of an event that affected a resource (read, create, update, delete) or an action taken on the system (login, logout, configuration change). It is the FHIR-native form factor for clinical-system audit records and is the basis for IHE ATNA's modern transport when FHIR is used in place of the historical DICOM Audit Message.

## Core Structure (Verify Against Current FHIR Version)

The resource has, broadly:

| Element | Purpose |
|---|---|
| `type` / `category` (R5) | The classification of the event (often a Coding from the audit-event-type code system, e.g., "RESTful Operation") |
| `subtype` / `code` (R5) | The specific event (e.g., "read", "create", "vread", "search", "history-instance") |
| `action` | C / R / U / D / E (Create, Read, Update, Delete, Execute) — a single character indicating the action |
| `severity` (R5) | Severity rating |
| `recorded` | The time the event was recorded — must be computer-generated, ISO 8601, ideally with timezone offset (UTC strongly preferred) |
| `outcome` | Success / failure / serious failure / major failure (numeric or coded depending on version) |
| `outcomeDesc` / `outcome.detail` (R5) | Human-readable detail on the outcome |
| `purposeOfEvent` / `authorization` (R5) | The purpose-of-use — e.g., TREATMENT, PAYMENT, OPERATIONS, RESEARCH, BREAK-THE-GLASS — coded from a recognized purpose-of-use system (often a v3 ActReason code system) |
| `agent` | Who (or what) participated in the event. One or more agents — at minimum the human user; often also the device, application, organization |
| `agent.who` | Reference to the Practitioner / Patient / Device / Organization / Person |
| `agent.role` | The role(s) of the agent at the time of the event |
| `agent.requestor` | Boolean — did this agent initiate the event? |
| `agent.network` | The network endpoint (IP address, hostname, user-agent) |
| `source` | The system that detected/observed the event — often the audit-emitting service |
| `source.site` | The physical or logical location of the source |
| `source.observer` | Reference to the observing system |
| `source.type` | The type of source (web server, app server, mobile, etc.) |
| `entity` | The data / object affected by the event. One or more entities — at minimum the patient and the record |
| `entity.what` | Reference to the resource (Patient, Observation, MedicationRequest, etc.) |
| `entity.role` | The role of the entity (Patient, Domain Resource, etc.) |
| `entity.lifecycle` | The lifecycle stage (Origination, Access, De-identification, etc.) |
| `entity.detail` | Free-form name/value pairs for additional context |

Verify each element name and structure against the current FHIR version — R5 introduced `category`, `severity`, and restructured `purposeOfEvent` into `authorization`; R4 and earlier use slightly different element names.

## Illustrative Example — Read Event (Synthetic)

```json
{
  "resourceType": "AuditEvent",
  "type": {
    "system": "http://terminology.hl7.org/CodeSystem/audit-event-type",
    "code": "rest",
    "display": "RESTful Operation"
  },
  "subtype": [{
    "system": "http://hl7.org/fhir/restful-interaction",
    "code": "read",
    "display": "read"
  }],
  "action": "R",
  "recorded": "2026-03-14T14:22:08Z",
  "outcome": "0",
  "purposeOfEvent": [{
    "coding": [{
      "system": "http://terminology.hl7.org/CodeSystem/v3-ActReason",
      "code": "TREAT",
      "display": "treatment"
    }]
  }],
  "agent": [{
    "type": {
      "coding": [{
        "system": "http://terminology.hl7.org/CodeSystem/v3-ParticipationType",
        "code": "AUT"
      }]
    },
    "who": {
      "reference": "Practitioner/synthetic-practitioner-001"
    },
    "requestor": true,
    "network": {
      "address": "10.20.30.40",
      "type": "2"
    }
  }],
  "source": {
    "observer": {
      "reference": "Device/synthetic-ehr-server-01"
    },
    "type": [{
      "system": "http://terminology.hl7.org/CodeSystem/security-source-type",
      "code": "4",
      "display": "Application Server"
    }]
  },
  "entity": [{
    "what": {
      "reference": "Patient/synthetic-patient-001"
    },
    "role": {
      "system": "http://terminology.hl7.org/CodeSystem/object-role",
      "code": "1",
      "display": "Patient"
    }
  }]
}
```

## Illustrative Example — Write Event (Synthetic)

```json
{
  "resourceType": "AuditEvent",
  "type": {
    "system": "http://terminology.hl7.org/CodeSystem/audit-event-type",
    "code": "rest"
  },
  "subtype": [{
    "system": "http://hl7.org/fhir/restful-interaction",
    "code": "update"
  }],
  "action": "U",
  "recorded": "2026-03-14T15:01:45Z",
  "outcome": "0",
  "purposeOfEvent": [{
    "coding": [{
      "system": "http://terminology.hl7.org/CodeSystem/v3-ActReason",
      "code": "TREAT"
    }]
  }],
  "agent": [{
    "who": { "reference": "Practitioner/synthetic-practitioner-001" },
    "requestor": true
  }],
  "source": {
    "observer": { "reference": "Device/synthetic-ehr-server-01" }
  },
  "entity": [
    {
      "what": { "reference": "Patient/synthetic-patient-001" },
      "role": { "code": "1", "display": "Patient" }
    },
    {
      "what": { "reference": "Observation/synthetic-observation-bp-001" },
      "role": { "code": "4", "display": "Domain Resource" },
      "lifecycle": { "code": "5", "display": "Amendment" }
    }
  ]
}
```

## Illustrative Example — Break-the-Glass (Synthetic)

```json
{
  "resourceType": "AuditEvent",
  "type": {
    "system": "http://terminology.hl7.org/CodeSystem/audit-event-type",
    "code": "rest"
  },
  "subtype": [{
    "system": "http://hl7.org/fhir/restful-interaction",
    "code": "read"
  }],
  "action": "R",
  "recorded": "2026-03-14T22:47:33Z",
  "outcome": "0",
  "purposeOfEvent": [{
    "coding": [{
      "system": "http://terminology.hl7.org/CodeSystem/v3-ActReason",
      "code": "ETREAT",
      "display": "Emergency Treatment"
    }]
  }],
  "agent": [{
    "who": { "reference": "Practitioner/synthetic-practitioner-022" },
    "requestor": true,
    "network": { "address": "10.20.30.99", "type": "2" }
  }],
  "source": {
    "observer": { "reference": "Device/synthetic-ehr-server-01" }
  },
  "entity": [
    {
      "what": { "reference": "Patient/synthetic-patient-073" },
      "role": { "code": "1", "display": "Patient" }
    },
    {
      "what": { "display": "Break-the-glass override" },
      "detail": [
        { "type": "btg-reason", "valueString": "ED arrival, patient unresponsive, no prior chart access" },
        { "type": "btg-acknowledgement", "valueBoolean": true }
      ]
    }
  ]
}
```

Note: actual codes for "Emergency Treatment" / break-the-glass vary across deployments — IHE ATNA, the HL7 Security and Privacy WG, and v3 ActReason all maintain code systems; verify which your environment uses.

## Common Use Cases

- **HIPAA §164.312(b) audit controls**: AuditEvent records satisfy the technical audit-controls requirement when retained per policy and reviewable
- **Accounting of disclosures support**: filter on `purposeOfEvent` to exclude TPO, then by recipient and patient
- **Break-the-glass review**: filter on BTG purpose-of-use code, route to review queue
- **Forensic investigation**: query AuditEvents for a patient over a time window after a complaint
- **IHE ATNA / BALP transport**: AuditEvent is the modern FHIR-native form factor for ATNA's audit messaging

## Retention and Storage

- AuditEvent records may themselves contain PHI (patient references, agent identifiers, network identifiers) — they must be protected as PHI
- Retention per HIPAA documentation rule (6 years) plus state/CMS overlay (see retention-by-jurisdiction.md)
- Tamper-resistance: append-only, separation of duties (app admins ≠ audit-store admins)
- Time synchronization across emitters (NTP/PTP; store in UTC)

## Common Pitfalls

- Missing `purposeOfEvent` — without it, accounting-of-disclosures filtering is impossible
- Using `recorded` from the application clock instead of a synchronized source
- Free-text PHI dumped into `entity.detail` — protect those payloads
- Forgetting to log failed-authentication and failed-authorization events
- One AuditEvent per request without correlation — chain via `agent.network` + session ID
- Multiple emitters with different code systems — standardize on terminology

## Final Disclaimers

- Verify the current FHIR specification version your environment uses (R4, R4B, R5)
- Verify the correct value sets and code systems for your jurisdiction and IHE profile (BALP, ATNA-FHIR, Basic Audit Log Patterns)
- Consult your interoperability lead and Privacy Officer for binding design decisions
