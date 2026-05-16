---
name: fhir-integration
description: When the user wants to design, implement, debug, or review FHIR integrations. Use when the user mentions "FHIR," "FHIR R4," "FHIR R5," "FHIR resource," "FHIR bundle," "FHIR search," "CapabilityStatement," "US Core," "USCDI," "CARIN BB," "Da Vinci," "mCODE," "IPS," "bulk data," "$export," "OperationOutcome," or names specific resources like Patient/Encounter/Observation/Condition/MedicationRequest. For SMART OAuth flows, see smart-on-fhir. For HL7 v2 pipe-delimited messages, see hl7-v2. For terminology services and code systems, see terminology-services. For CDA documents, see cda-ccda.
metadata:
  version: 1.0.0
---

# FHIR Integration

You are an expert in HL7 FHIR R4 and R5 integration. Your goal is to help engineers and informaticists design, implement, and debug FHIR-based interoperability — resources, RESTful interactions, search, Bundles, profiles, Implementation Guides (IGs), bulk data, and conformance — without fabricating codes, profiles, or behavior. When unsure of a binding or profile, point the reader to the authoritative IG.

## Initial Assessment

Check `.agents/healthcare-context.md` (fallback: `.claude/healthcare-context.md`) before answering. The context file tells you:

- **FHIR version** (R4 vs. R5) — operations, resource elements, and search parameters differ.
- **Implementation Guides** in use (US Core 6.x / 7.x / 8.x, USCDI v3/v4, CARIN BB, Da Vinci PDex/PAS/CRD/DTR, mCODE, IPS) — these drive must-support elements and required value set bindings.
- **EHR vendor** — Epic, Oracle Health (Cerner), Meditech, Athenahealth, etc. each have vendor-specific quirks and scopes.
- **HIPAA role and jurisdiction** — affects auth, audit, and consent expectations.

If the file does not exist, ask the smallest set of questions needed (FHIR version, IGs, server vs. client role, EHR vendor) and offer to save the answers to the context file.

---

## Core Concepts

### Resources and the Resource Model

FHIR organizes clinical and administrative data as **resources** — granular, independently addressable JSON/XML objects with a stable URL: `https://server/fhir/{ResourceType}/{id}`.

Common clinical resources you will encounter:

| Resource | Purpose |
|----------|---------|
| Patient | Demographic and administrative info on a person receiving care |
| Encounter | Interaction between patient and provider(s) for care delivery |
| Observation | Measurements and simple assertions (vitals, labs, social history) |
| Condition | Clinical conditions, problems, diagnoses |
| Procedure | Actions performed on/for a patient |
| MedicationRequest | Order or prescription for medication |
| MedicationStatement | Record of a medication being taken (history) |
| AllergyIntolerance | Allergy or intolerance risk |
| DiagnosticReport | Findings/interpretation of diagnostic tests |
| DocumentReference | Pointer to a clinical document (often C-CDA) |
| ServiceRequest | Request for a procedure, lab, imaging, or referral |
| Immunization | Immunization event |
| CarePlan / CareTeam / Goal | Care planning data |
| Coverage / ExplanationOfBenefit / Claim | Payer-facing financial resources |

Conformance resources (`StructureDefinition`, `ValueSet`, `CodeSystem`, `ConceptMap`, `CapabilityStatement`, `SearchParameter`, `OperationDefinition`) describe a server's behavior and constraints.

### RESTful Interactions

| Interaction | Method | URL pattern |
|-------------|--------|-------------|
| read | `GET` | `/{Resource}/{id}` |
| vread | `GET` | `/{Resource}/{id}/_history/{vid}` |
| update | `PUT` | `/{Resource}/{id}` |
| patch | `PATCH` | `/{Resource}/{id}` (JSON Patch or FHIRPath Patch) |
| delete | `DELETE` | `/{Resource}/{id}` |
| history (instance/type/system) | `GET` | `/{Resource}/{id}/_history`, `/{Resource}/_history`, `/_history` |
| create | `POST` | `/{Resource}` |
| search | `GET` / `POST` | `/{Resource}?param=value` (POST to `_search`) |
| transaction | `POST` | `/` with Bundle.type=transaction |
| batch | `POST` | `/` with Bundle.type=batch |
| capabilities | `GET` | `/metadata` |
| operation | `GET` / `POST` | `/${operation}` or `/{Resource}/${operation}` |

### Versioning, ETags, Concurrency

- Each resource has `meta.versionId` and `meta.lastUpdated`. Servers return `ETag: W/"{versionId}"` and `Last-Modified`.
- Clients send `If-Match: W/"{versionId}"` on update/patch for optimistic concurrency. The server returns `412 Precondition Failed` on conflict.
- Use `If-None-Exist` on conditional create and `If-Match`/`If-None-Match` on conditional update for idempotency.

---

## Search

Search is performed by GET with query params, or by POST to `/{Resource}/_search` with form-encoded params (preferred when params contain PHI).

### Common Modifiers and Prefixes

- String: `:contains`, `:exact`
- Token: `system|code`, `:not`, `:in`, `:below`, `:above`, `:of-type`
- Reference: `:identifier`, `:[ResourceType]`
- Date: prefixes `eq, ne, gt, lt, ge, le, sa, eb, ap`
- Quantity / Number: same prefixes

### Chaining and Reverse Chaining

- **Chain** (`.`) — filter on a referenced resource's parameter.
  `GET /Observation?subject:Patient.identifier=http://hospital|MRN-123`
- **Reverse chain** (`_has`) — filter resources whose references point at them.
  `GET /Patient?_has:Observation:subject:code=http://loinc.org|85354-9`
- **_include / _revinclude** — return related resources inline.
  `GET /Observation?code=http://loinc.org|2160-0&_include=Observation:subject&_revinclude=Provenance:target`

### Result Parameters

`_count`, `_sort`, `_summary`, `_elements`, `_total`, `_format`, `_pretty`, `_filter` (R5+), `_contained`, `_containedType`.

---

## Bundles

A Bundle is the container used for collections, search results, history, transactions, batches, documents, and messages.

| `Bundle.type` | Use |
|---------------|-----|
| `document` | Clinical document (Composition is the first entry) |
| `message` | Messaging — MessageHeader is the first entry |
| `transaction` | Atomic multi-resource write (all-or-nothing) |
| `batch` | Independent operations; each entry succeeds/fails on its own |
| `history` | Resource history results |
| `searchset` | Search results |
| `collection` | Arbitrary grouping (no processing semantics) |
| `transaction-response` / `batch-response` | Server responses |

Each transaction/batch entry includes a `request` (method + url) and may include `ifMatch`, `ifNoneExist`, etc.

---

## Profiles, Extensions, and Conformance

- **StructureDefinition** declares profiles — constraints on a base resource (cardinality, value set bindings, must-support).
- Resources declare conformance via `meta.profile = ["http://hl7.org/fhir/us/core/StructureDefinition/us-core-patient"]`.
- **Extensions** add new elements using `url` to identify the definition. Modifier extensions (`modifierExtension`) change meaning and must be understood.
- **CapabilityStatement** (`GET /metadata`) declares which resources, interactions, search params, and operations a server supports.

### Common US Implementation Guides

| IG | Purpose |
|----|---------|
| US Core (R4 6.x, 7.x, 8.x) | Baseline US data set aligned with USCDI v3/v4/v5 |
| USCDI | ONC-defined data classes/elements that certified EHRs must expose |
| CARIN BB | Consumer-Directed Payer Data Exchange (claims, EOB) |
| Da Vinci PDex | Payer-to-payer, payer-to-provider data exchange |
| Da Vinci PAS / CRD / DTR | Prior authorization, coverage requirements, documentation templates |
| mCODE | Minimal Common Oncology Data Elements |
| IPS | International Patient Summary |

Always verify the exact IG version in use against the published profile at `hl7.org/fhir/us/...` — must-support and cardinality change across versions.

### Terminology Binding Strengths

| Strength | Meaning |
|----------|---------|
| required | Code must come from the value set |
| extensible | Use the value set if a code exists; otherwise a different code is allowed |
| preferred | Use the value set when possible |
| example | Illustrative only |

---

## Operations

Operations are named functions invoked with a leading `$`.

| Operation | Scope | Purpose |
|-----------|-------|---------|
| `$validate` | type / instance | Validate against profile |
| `$everything` | Patient / Encounter | All resources related to the subject |
| `$expand` | ValueSet | Expand a value set |
| `$lookup` / `$validate-code` | CodeSystem | Lookup or validate a code |
| `$translate` | ConceptMap | Map a code between code systems |
| `$export` | System / Patient / Group | Bulk data export |
| `$match` | Patient | Patient matching |
| `$document` | Composition | Generate a Bundle of type document |

---

## Bulk Data Export

The Bulk Data Access IG defines `$export` for population-scale extraction.

- Kickoff: `GET /Group/{id}/$export?_type=Patient,Observation&_since=2024-01-01T00:00:00Z&_typeFilter=Observation%3Fcategory%3Dlaboratory`
- Headers: `Accept: application/fhir+json`, `Prefer: respond-async`
- Response: `202 Accepted` with `Content-Location` polling URL
- Poll until `200 OK` with a JSON manifest listing NDJSON files per resource type
- Auth via SMART Backend Services (asymmetric client credentials, JWS-signed JWT)

---

## Error Handling — OperationOutcome

FHIR returns errors as `OperationOutcome`. Use `issue.severity` (`fatal | error | warning | information`), `issue.code` (a FHIR-defined token like `not-found`, `conflict`, `invalid`, `security`, `processing`), and `issue.details`/`diagnostics` for human-readable info.

```json
{
  "resourceType": "OperationOutcome",
  "issue": [{
    "severity": "error",
    "code": "invariant",
    "details": { "text": "us-core-1: Patient.name must have a family or given name" },
    "expression": ["Patient.name[0]"]
  }]
}
```

---

## Request / Response Examples

All examples use synthetic identifiers.

### read

```
GET /Patient/example-1
Accept: application/fhir+json
```

```json
{ "resourceType": "Patient", "id": "example-1", "meta": { "versionId": "3" }, "name": [{ "family": "Smith", "given": ["Jane"] }] }
```

### create (with conditional create)

```
POST /Patient
If-None-Exist: identifier=http://hospital.example.org/mrn|MRN-123456
Content-Type: application/fhir+json
```

### update with optimistic concurrency

```
PUT /Patient/example-1
If-Match: W/"3"
Content-Type: application/fhir+json
```

Response on conflict: `412 Precondition Failed` with `OperationOutcome`.

### patch (JSON Patch)

```
PATCH /Patient/example-1
Content-Type: application/json-patch+json

[ { "op": "replace", "path": "/active", "value": true } ]
```

### search

```
GET /Observation?subject=Patient/example-1&code=http://loinc.org|85354-9&date=ge2025-01-01&_sort=-date&_count=50
```

### transaction Bundle

```json
{
  "resourceType": "Bundle",
  "type": "transaction",
  "entry": [
    { "fullUrl": "urn:uuid:1", "resource": { "resourceType": "Patient", "name": [{ "family": "Doe" }] }, "request": { "method": "POST", "url": "Patient", "ifNoneExist": "identifier=http://hospital|MRN-987" } },
    { "resource": { "resourceType": "Observation", "subject": { "reference": "urn:uuid:1" }, "status": "final", "code": { "coding": [{ "system": "http://loinc.org", "code": "8480-6" }] } }, "request": { "method": "POST", "url": "Observation" } }
  ]
}
```

### bulk export kickoff

```
GET /Group/heart-failure-cohort/$export?_type=Patient,Condition,Observation&_since=2025-01-01T00:00:00Z
Accept: application/fhir+json
Prefer: respond-async
```

### CapabilityStatement

```
GET /metadata
Accept: application/fhir+json
```

---

## R4 vs. R5 Differences (high level)

- New resources in R5 (e.g., `Permission`, `Requirements`, `ActorDefinition`, `SubscriptionTopic`); `Subscription` reworked.
- Some renamed elements and tightened cardinality.
- Subscriptions topic-based in R5 (vs. criteria-based in R4).
- `_filter` and improved search semantics.
- Always confirm against the FHIR spec version in your context file before promising behavior.

---

## Common Pitfalls

- Treating `Reference.reference` strings as opaque — they may be absolute, relative, or `urn:uuid:` (in Bundles). Resolve carefully.
- Ignoring `modifierExtension` — these change clinical meaning.
- Searching without `_count` or pagination handling — many servers cap pages at 50–100.
- Confusing `Observation.effective[x]` (when the observation pertains to) with `Observation.issued` (when it was released).
- Trusting `meta.profile` claims without `$validate` — profile assertion is not enforced unless validated.
- Forgetting that PATCH semantics, conditional updates, and `_history` are optional — check the CapabilityStatement.

---

## Task-Specific Questions

1. Which FHIR version are we targeting — R4 or R5? (Resource elements, operations, and Subscription mechanics differ.)
2. Which Implementation Guide(s) are in scope — US Core (which version: 6.x / 7.x / 8.x), USCDI v3/v4/v5, CARIN BB, Da Vinci (PDex/PAS/CRD/DTR), mCODE, IPS, or something else?
3. Are you building the server side, the client side, or both — and what is the auth model (SMART App Launch, SMART Backend Services, mTLS, API key)?
4. Which resources and interactions matter for this work (read, search, create/update, transaction, $export, $everything, $validate)?
5. Which EHR or FHIR server are you integrating against (Epic, Oracle Health/Cerner, Athenahealth, Meditech, HAPI, Azure API for FHIR, Google Cloud Healthcare, AWS HealthLake, custom)?
6. What's the data scope and consent model — single-patient, group/population, system-wide, and which HIPAA role does the deployment play?

---

## Related Skills

- **smart-on-fhir** — SMART App Launch, OAuth2 scopes (`patient/*.read`, `system/*.read`), and backend services for `$export` auth
- **hl7-v2** — when integrating with legacy systems that speak pipe-delimited HL7 v2 alongside FHIR
- **terminology-services** — for value set expansion, code translation, and binding validation
- **cda-ccda** — for `DocumentReference` payloads that contain C-CDA, and FHIR↔CDA bridging
- **ehr-integration** — vendor-specific FHIR endpoints (Epic, Oracle Health/Cerner, Athena, Meditech) and their scope quirks
- **hipaa-compliance** — required transmission security, audit, and minimum-necessary scoping for FHIR APIs
- **audit-logging** — FHIR `AuditEvent` patterns and IHE ATNA alignment
