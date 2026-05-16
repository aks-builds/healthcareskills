# FHIR Search Parameters

Practical reference for FHIR search — common parameters, prefixes, modifiers, chaining, _include / _revinclude, and result control.

## Contents

- Basic Search Syntax
- Parameter Types and Prefixes
- Token Modifiers
- String Modifiers
- Reference Modifiers and Chaining
- Reverse Chaining (_has)
- _include and _revinclude
- Result Parameters (_count, _sort, _summary, _elements, _total)
- Composite and Special Parameters
- Common Searches per Resource
- POST to _search
- Pagination
- Gotchas

---

## Basic Search Syntax

```
GET /[Resource]?param1=value1&param2=value2
```

- One search returns a Bundle with `type = searchset`.
- Empty result is still a 200 OK with `total = 0`.
- Unknown parameters are typically ignored unless the server is in strict mode (`Prefer: handling=strict`).

---

## Parameter Types and Prefixes

| Type | Examples | Prefixes (date/number/quantity) |
|------|----------|---------------------------------|
| number | `Observation?value-quantity=7.2` | `eq, ne, gt, lt, ge, le, sa, eb, ap` |
| date | `Observation?date=ge2025-01-01` | same as number |
| string | `Patient?family=smith` | (use modifiers, not prefixes) |
| token | `Condition?code=http://snomed.info/sct\|73211009` | `(none — use modifiers)` |
| reference | `Observation?subject=Patient/123` | (use modifiers) |
| quantity | `Observation?value-quantity=ge5.0\|http://unitsofmeasure.org\|mg/dL` | same as number |
| uri | `ValueSet?url=http://hl7.org/fhir/ValueSet/...` | |
| composite | `Observation?component-code-value-quantity=...` | |
| special | `Patient?_id=123`, `Patient?_lastUpdated=ge2025-01-01` | |

**Date prefixes**:

- `eq` equal (default)
- `ne` not equal
- `gt` `lt` `ge` `le` greater/less than (or equal)
- `sa` starts after
- `eb` ends before
- `ap` approximately

---

## Token Modifiers

Token = `system|code`. The system is the canonical code-system URI.

| Modifier | Meaning |
|----------|---------|
| `:text` | Match on display/text |
| `:not` | Exclude this code |
| `:above` | Code is at or above (subsumption — for hierarchical systems) |
| `:below` | Code is at or below (children/descendants) |
| `:in` | Code is in a value set (`?code:in=http://...vs`) |
| `:not-in` | Code is not in a value set |
| `:of-type` | Identifier type code (CodeableConcept.coding) |

Examples:

```
GET /Condition?code:below=http://snomed.info/sct|73211009
GET /Condition?code:in=http://hl7.org/fhir/us/core/ValueSet/us-core-diabetes
GET /Patient?identifier:of-type=http://terminology.hl7.org/CodeSystem/v2-0203|MR|MRN-123456
```

---

## String Modifiers

| Modifier | Meaning |
|----------|---------|
| (none) | Starts-with, case-insensitive (default) |
| `:exact` | Case-sensitive exact match |
| `:contains` | Substring, case-insensitive |

```
GET /Patient?family:contains=mit
GET /Patient?name:exact=Smith
```

---

## Reference Modifiers and Chaining

References can be searched by relative reference or by the referenced resource's parameters (chaining).

### Direct reference

```
GET /Observation?subject=Patient/example-1
GET /Observation?subject=Patient/example-1&category=laboratory
```

### Reference type qualifier

```
GET /Observation?subject:Patient=example-1
```

### Identifier on reference

```
GET /Observation?subject:identifier=http://hospital.example.org/mrn|MRN-123
```

### Chaining (.)

Filter a resource by a parameter of its referenced resource:

```
GET /Observation?subject:Patient.identifier=http://hospital.example.org/mrn|MRN-123
GET /MedicationRequest?patient.gender=female
GET /Encounter?practitioner.name=smith
```

Multi-hop chaining is possible but server support varies — keep chains shallow.

---

## Reverse Chaining (_has)

Filter resources whose references point at them.

```
GET /Patient?_has:Observation:subject:code=http://loinc.org|85354-9
GET /Patient?_has:Condition:subject:code:in=http://hl7.org/fhir/us/core/ValueSet/us-core-diabetes
```

Reads as: find Patients that have at least one Observation whose `subject` is the patient and whose `code` is the LOINC BP panel code.

---

## _include and _revinclude

`_include` pulls referenced resources into the result Bundle. `_revinclude` pulls resources whose references point at the matched resources.

```
GET /Observation?code=http://loinc.org|2160-0&_include=Observation:subject
GET /MedicationRequest?patient=Patient/123&_include=MedicationRequest:medication
GET /Patient?_id=123&_revinclude=Provenance:target
```

`_include=*` includes all referenced resources (use sparingly).
`_include:iterate=Resource:param` cascades includes (e.g., Observation → Patient → Practitioner).

---

## Result Parameters

| Parameter | Purpose |
|-----------|---------|
| `_count` | Page size (servers cap, often 50–500) |
| `_sort` | Sort spec; `-field` for descending; comma-separated for multi-key |
| `_summary` | `true` / `text` / `data` / `count` / `false` — limit returned elements |
| `_elements` | Comma-separated list of elements to return |
| `_total` | `none` / `estimate` / `accurate` — whether server counts total matches |
| `_format` | `json` / `xml` / `application/fhir+json` |
| `_pretty` | Boolean — pretty-print response |
| `_contained` | `true` / `false` / `both` — control contained resource searches |
| `_filter` | R5+ — single-string filter expression |

```
GET /Patient?_count=50&_sort=family,-birthdate&_total=accurate&_summary=text
GET /Observation?subject=Patient/123&_elements=code,effectiveDateTime,valueQuantity
```

---

## Composite Parameters

Some searches combine two parameters into a single composite (with `$` joining the values):

```
GET /Observation?component-code-value-quantity=http://loinc.org|8480-6$gt140
```

This finds Observations where component has LOINC `8480-6` (systolic BP) and component value > 140.

---

## Special Parameters

- `_id` — search by logical id
- `_lastUpdated` — search by `meta.lastUpdated` (date prefixes)
- `_profile` — search by asserted profile
- `_tag` — search by `meta.tag`
- `_security` — search by `meta.security`
- `_source` — search by `meta.source`
- `_text` — narrative-text search (full-text)
- `_content` — content text search

---

## Common Searches per Resource

### Patient

```
GET /Patient?identifier=http://hospital|MRN-123
GET /Patient?name=smith&birthdate=1985-02-03
GET /Patient?_id=example-1
```

### Encounter

```
GET /Encounter?patient=Patient/123&date=ge2025-01-01
GET /Encounter?patient=Patient/123&class=http://terminology.hl7.org/CodeSystem/v3-ActCode|IMP&status=in-progress
```

### Observation

```
GET /Observation?patient=Patient/123&category=laboratory&date=ge2025-01-01
GET /Observation?patient=Patient/123&code=http://loinc.org|85354-9&_sort=-date&_count=20
```

### Condition

```
GET /Condition?patient=Patient/123&category=problem-list-item&clinical-status=active
GET /Condition?patient=Patient/123&code:below=http://snomed.info/sct|73211009
```

### MedicationRequest

```
GET /MedicationRequest?patient=Patient/123&status=active&_include=MedicationRequest:medication
```

### AllergyIntolerance

```
GET /AllergyIntolerance?patient=Patient/123&clinical-status=active
```

### DocumentReference

```
GET /DocumentReference?patient=Patient/123&type=http://loinc.org|34133-9&status=current
```

---

## POST to _search

Use when query params contain PHI or are too long for a URL.

```
POST /Observation/_search
Content-Type: application/x-www-form-urlencoded

subject=Patient/example-1&code=http://loinc.org|85354-9&date=ge2025-01-01
```

Identical semantics to GET. Recommended for production when search params include identifiers.

---

## Pagination

Search bundles include `link` entries with `relation` of `self`, `next`, `previous`, `first`, `last`.

```
GET (initial)
GET <self link>  → Bundle with link[relation=next]
GET <next link>
...
```

The server controls page size; honor the `next` link rather than constructing it client-side.

---

## Gotchas

- **No code system on a token** — `?code=I10` (without `system|`) is ambiguous; servers may interpret as match-on-code-anywhere or refuse it.
- **Date precision** — `?date=2025-01-01` matches the whole day; `?date=ge2025-01-01T08:00:00Z` is more precise.
- **Modifier compatibility** — not all modifiers work on all servers; check the CapabilityStatement.
- **_include depth** — `_include:iterate` can explode result size.
- **Chaining limits** — many servers cap chains at one level; reverse-chain (`_has`) can be even more limited.
- **Sorting indeterminism** — sort by a stable field plus `_id` to avoid duplicates across pages.
- **_total=accurate** can be expensive on large datasets.
- **Custom parameters** must be declared in the CapabilityStatement; otherwise treat as nonstandard.

Always start from the server's CapabilityStatement (`GET /metadata`) to confirm which parameters and modifiers are supported.
