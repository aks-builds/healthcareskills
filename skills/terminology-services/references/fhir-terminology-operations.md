# FHIR Terminology Operations

Practical reference for the FHIR Terminology Service operations: `$lookup`, `$validate-code`, `$expand`, `$translate`, `$subsumes`, plus `$closure`. All examples use **synthetic identifiers**. Verify against the current FHIR Terminology Service specification.

## Contents

- Overview
- $lookup
- $validate-code
- $expand
- $translate
- $subsumes
- $closure
- Parameters vs. Query String
- Versioning
- Server Capability Statements
- Performance Tips
- Common Failure Modes

---

## Overview

| Operation | Resource | Purpose |
|-----------|----------|---------|
| `$lookup` | CodeSystem | Get display + properties for a code |
| `$validate-code` | CodeSystem or ValueSet | Check that a code is valid |
| `$expand` | ValueSet | Expand to a flat list of codes |
| `$translate` | ConceptMap | Map a code via a ConceptMap |
| `$subsumes` | CodeSystem | Check subsumption between two codes |
| `$closure` | system | Compute transitive closure (server-only, less common) |

Operations accept parameters either as URL query string (for GET) or as a `Parameters` resource in the POST body. POST is preferred when params contain PHI or large payloads.

---

## $lookup

Returns the display, designations, and properties of a single code.

### Request (GET)

```
GET /CodeSystem/$lookup?system=http://loinc.org&code=8480-6
Accept: application/fhir+json
```

### Request (POST)

```
POST /CodeSystem/$lookup
Content-Type: application/fhir+json

{
  "resourceType": "Parameters",
  "parameter": [
    { "name": "system", "valueUri": "http://loinc.org" },
    { "name": "code", "valueCode": "8480-6" },
    { "name": "version", "valueString": "2.76" }
  ]
}
```

### Response

```json
{
  "resourceType": "Parameters",
  "parameter": [
    { "name": "name", "valueString": "LOINC" },
    { "name": "version", "valueString": "2.76" },
    { "name": "display", "valueString": "Systolic blood pressure" },
    { "name": "designation", "part": [
        { "name": "language", "valueCode": "en" },
        { "name": "value", "valueString": "Systolic blood pressure" }
    ]},
    { "name": "property", "part": [
        { "name": "code", "valueCode": "COMPONENT" },
        { "name": "value", "valueString": "Systolic blood pressure" }
    ]}
  ]
}
```

### Common Use

- Resolve a code to its display string when you only have `system + code`
- Pull properties (e.g., LOINC parts, SNOMED relationships, RxNorm TTY)
- Verify a code exists in a code system

---

## $validate-code

Check that a code is valid within a CodeSystem or ValueSet. Operation on either resource type with different parameters.

### Validate against a CodeSystem

```
GET /CodeSystem/$validate-code?
  url=http://loinc.org
  &code=8480-6
  &display=Systolic blood pressure
```

### Validate against a ValueSet (the common case)

```
GET /ValueSet/$validate-code?
  url=http://hl7.org/fhir/us/core/ValueSet/us-core-vital-signs
  &system=http://loinc.org
  &code=8480-6
```

You can also reference the ValueSet by id:

```
GET /ValueSet/us-core-vital-signs/$validate-code?system=http://loinc.org&code=8480-6
```

### Response

```json
{
  "resourceType": "Parameters",
  "parameter": [
    { "name": "result", "valueBoolean": true },
    { "name": "display", "valueString": "Systolic blood pressure" }
  ]
}
```

On failure:

```json
{
  "resourceType": "Parameters",
  "parameter": [
    { "name": "result", "valueBoolean": false },
    { "name": "message", "valueString": "The code 'NOT-A-CODE' from system 'http://loinc.org' is not valid in the ValueSet 'us-core-vital-signs'." }
  ]
}
```

### Common Use

- Ingestion-time validation of every coding
- Schema-and-binding validation in `$validate` flows (FHIR Validator does this under the hood)
- Round-trip checking when transforming data

### Parameters

| Parameter | Purpose |
|-----------|---------|
| `url` | The CodeSystem or ValueSet canonical URL |
| `valueSet` | Inline ValueSet resource (for ad-hoc validation) |
| `system` | Code system URI |
| `code` | The code to validate |
| `display` | Display to confirm (server may warn on mismatch) |
| `coding` | A Coding instead of system+code |
| `codeableConcept` | A CodeableConcept (any of its codings is valid → result true) |
| `valueSetVersion` | Pin to a specific value-set version |
| `systemVersion` | Pin to a specific code-system version |

---

## $expand

Expand a ValueSet to a flat list of codes (the `expansion`).

### Request

```
GET /ValueSet/us-core-vital-signs/$expand?count=20&offset=0
```

Or for an ad-hoc value set defined by URL:

```
GET /ValueSet/$expand?url=http://hl7.org/fhir/us/core/ValueSet/us-core-vital-signs
```

With a filter:

```
GET /ValueSet/$expand?url=...&filter=blood&count=10
```

### Response

```json
{
  "resourceType": "ValueSet",
  "url": "http://hl7.org/fhir/us/core/ValueSet/us-core-vital-signs",
  "expansion": {
    "timestamp": "2026-03-01T12:00:00Z",
    "total": 12,
    "offset": 0,
    "contains": [
      { "system": "http://loinc.org", "code": "85354-9", "display": "Blood pressure panel with all children optional" },
      { "system": "http://loinc.org", "code": "8480-6", "display": "Systolic blood pressure" },
      { "system": "http://loinc.org", "code": "8462-4", "display": "Diastolic blood pressure" }
    ]
  }
}
```

### Parameters

| Parameter | Purpose |
|-----------|---------|
| `url` | ValueSet canonical URL |
| `valueSet` | Inline ValueSet resource |
| `valueSetVersion` | Version pin |
| `filter` | Text filter (case-insensitive substring on display) |
| `count` / `offset` | Pagination |
| `includeDesignations` | Include language designations |
| `activeOnly` | Exclude inactive codes |
| `displayLanguage` | Preferred language for displays |

### Common Use

- Render dropdown / autocomplete in a UI
- Cache value-set members for offline validation
- Detect content drift across releases (compare expansions over time)

---

## $translate

Map a code from one code system to another using a ConceptMap.

### Request — Map by ConceptMap URL

```
GET /ConceptMap/$translate?
  url=http://hl7.org/fhir/ConceptMap/icd10cm-to-snomedct
  &code=I10
  &system=http://hl7.org/fhir/sid/icd-10-cm
```

### Request — Specify source and target systems

```
GET /ConceptMap/$translate?
  code=I10
  &system=http://hl7.org/fhir/sid/icd-10-cm
  &targetSystem=http://snomed.info/sct
```

### Response

```json
{
  "resourceType": "Parameters",
  "parameter": [
    { "name": "result", "valueBoolean": true },
    { "name": "match", "part": [
        { "name": "equivalence", "valueCode": "equivalent" },
        { "name": "concept", "valueCoding": {
            "system": "http://snomed.info/sct",
            "code": "59621000",
            "display": "Essential hypertension"
        }}
    ]},
    { "name": "match", "part": [
        { "name": "equivalence", "valueCode": "wider" },
        { "name": "concept", "valueCoding": {
            "system": "http://snomed.info/sct",
            "code": "38341003",
            "display": "Hypertensive disorder, systemic arterial"
        }}
    ]}
  ]
}
```

### Equivalence Values

| Value | Meaning |
|-------|---------|
| `relatedto` | Source and target related, but unknown how |
| `equivalent` | Equivalent in meaning |
| `equal` | Exactly the same |
| `wider` | Target is broader than source |
| `subsumes` | Target subsumes source (target is ancestor) |
| `narrower` | Target is narrower than source |
| `specializes` | Target is a specialization of source |
| `inexact` | Mapping is inexact |
| `unmatched` | No mapping exists |
| `disjoint` | Codes are disjoint (NOT a match) |

**Critical**: do not silently treat `wider` or `narrower` as `equal`. Preserve the equivalence and let downstream logic decide.

### Common Use

- ICD-10-CM → SNOMED CT
- NDC → RxNorm
- Local code → standard code
- Old code → new code (within a system version migration)

---

## $subsumes

Check whether one code is subsumed by another in a hierarchical code system (SNOMED CT, LOINC parts).

### Request

```
GET /CodeSystem/$subsumes?
  system=http://snomed.info/sct
  &codeA=73211009
  &codeB=44054006
```

### Response

```json
{
  "resourceType": "Parameters",
  "parameter": [
    { "name": "outcome", "valueCode": "subsumes" }
  ]
}
```

### Outcome Values

| Value | Meaning |
|-------|---------|
| `equivalent` | A and B are the same |
| `subsumes` | A subsumes B (A is ancestor) |
| `subsumed-by` | A is subsumed by B (B is ancestor) |
| `not-subsumed` | No subsumption relationship |

### Common Use

- "Does this patient have any form of diabetes?" → subsumes("Diabetes mellitus", their diagnosis code)
- Cohort eligibility checks
- Value-set authoring (express VS as descendants of an ancestor)

---

## $closure

Compute the transitive closure of relationships — typically a server-internal operation used by Terminology Services to materialize derived value sets. Most clients don't call it directly.

```
POST /$closure
Content-Type: application/fhir+json

{
  "resourceType": "Parameters",
  "parameter": [
    { "name": "name", "valueString": "diabetes-tracking" },
    { "name": "concept", "valueCoding": { "system": "http://snomed.info/sct", "code": "73211009" } }
  ]
}
```

Server response is a ConceptMap-shaped artifact tracking subsumption relationships.

---

## Parameters vs. Query String

For all the above operations:

- Simple parameter values → query string (GET) is fine
- Complex parameter values (Coding, CodeableConcept, ValueSet inline) → POST with Parameters resource body
- PHI in parameters → POST always
- Servers vary on which forms they accept — check CapabilityStatement and OperationDefinition

---

## Versioning

Pin versions on every operation that touches a code system or value set:

| Parameter | Use |
|-----------|-----|
| `systemVersion` | Pin the code-system version |
| `valueSetVersion` | Pin the value-set version |
| `force-system-version` | Some servers — force a specific version even when the resource has none |

For SNOMED CT, the canonical URL with version is `http://snomed.info/sct/{module}/version/{date}` (e.g., `http://snomed.info/sct/731000124108/version/20250901`).

For other systems, use the system's own version semantics (LOINC `2.76`, ICD-10-CM `2025`, RxNorm release date, etc.).

---

## Server Capability Statements

Servers declare terminology support in their CapabilityStatement under `rest.resource[].operation`:

```json
{
  "type": "ValueSet",
  "operation": [
    { "name": "expand", "definition": "http://hl7.org/fhir/OperationDefinition/ValueSet-expand" },
    { "name": "validate-code", "definition": "http://hl7.org/fhir/OperationDefinition/ValueSet-validate-code" }
  ]
}
```

Check the OperationDefinition for supported parameters and constraints (e.g., max expansion size).

---

## Performance Tips

- **Cache value-set expansions** locally for high-volume validation
- **Use the smallest needed value set** — don't validate against an enormous global set when a constrained subset works
- **Batch validation** — many servers support a Bundle of $validate-code calls
- **Avoid $expand for very large sets** — paginate with `count`/`offset` or use ECL/filter expressions
- **Pin versions** to enable caching
- **Set timeouts** — terminology operations can be slow on large terminologies

---

## Common Failure Modes

- **Missing system parameter** — defaults to "anywhere" on some servers; always specify
- **Stale cache** — server's cached expansion lags the actual ValueSet; trigger re-expansion
- **Ambiguous code** — same `code` exists in multiple systems; always supply `system`
- **Version mismatch** — code is valid in one version, invalid in another; align versions explicitly
- **Filter not supported** — server doesn't implement `filter` for the operation; fall back to local filtering
- **Server lacks the code system** — $lookup returns "not found" even though the code is well-known; the server simply hasn't loaded it
- **Display mismatch warnings** — server returns `result=true` but warns on display discrepancy; treat as a quality signal

When in doubt, read the server's CapabilityStatement and the OperationDefinition for each operation.
