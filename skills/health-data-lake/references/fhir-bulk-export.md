# FHIR Bulk Data Export

Reference for the HL7 FHIR Bulk Data Access (`$export`) standard — the canonical population-extract pattern for moving large volumes of clinical data out of a FHIR server into a data lake.

## Contents

- What `$export` Is
- Scopes — System / Group / Patient
- The Asynchronous Flow
- Kickoff Request
- Polling for Status
- Downloading NDJSON Files
- Filtering with _type, _since, _typeFilter
- Authentication and SMART Backend Services
- Consuming NDJSON in the Lake
- CMS BCDA Specifics
- Common Pitfalls

---

## What `$export` Is

`$export` is a FHIR operation that returns a population-level extract as NDJSON (newline-delimited JSON, one FHIR resource per line) at a pre-signed download URL. It runs **asynchronously**: the client kicks off the export, polls a status endpoint, and downloads files when ready.

Why NDJSON: streaming-friendly, language-agnostic, splittable for parallel ingest, and easy to write to object storage. Spark, Snowflake, Databricks, BigQuery, and DuckDB all read NDJSON natively.

The Bulk Data IG is the basis. CMS BCDA (Beneficiary Claims Data API) is one well-known production implementation; most major EHR FHIR endpoints (Epic, Oracle Health / Cerner, MEDITECH, athenahealth — verify current support) implement at least Group-level `$export`.

---

## Scopes — System / Group / Patient

Three scopes determine what population the export covers.

### System (`/$export`)

Exports resources at the FHIR server level, across all patients the requesting principal is authorized to see. Used by:

- Health systems exporting their entire population for a research warehouse or lake.
- Population-health platforms ingesting a complete EHR slice.

System exports are large and not all FHIR servers support them.

### Group (`/Group/{id}/$export`)

Exports resources for the patients in a defined FHIR `Group` (the panel, ACO attribution list, study cohort, registry roster, etc.). This is the **most common production pattern** because it bounds the scope precisely and aligns with how operational populations are usually managed.

Examples:

- MSSP ACO attributed beneficiaries (CMS BCDA).
- Disease registry cohort.
- Risk-bearing-arrangement panel.
- Clinical-trial participant list.

The `Group` resource is the contract between the data producer and consumer. Update the `Group.member` list when the cohort changes.

### Patient (`/Patient/$export`)

Exports across all patients in scope without an explicit Group. Behavior varies by server; some treat it as system-export, some require a Group.

Single-patient exports use `/Patient/{id}/$export` and are less common — they exist mainly for patient-mediated app workflows.

---

## The Asynchronous Flow

```
Client                                Server
  | --- POST $export?_type=... ---> | (validate, queue)
  | <-- 202 Accepted, Content-Location: <statusUrl>
  |
  | --- GET <statusUrl> ---------->  | (still working)
  | <-- 202 Accepted, X-Progress: "50%"
  |
  | (poll periodically, respect Retry-After)
  |
  | --- GET <statusUrl> ---------->  | (done)
  | <-- 200 OK, JSON manifest with file URLs
  |
  | --- GET <fileUrl1> ----------->  |
  | <-- 200 OK, NDJSON stream
  | --- GET <fileUrl2> ----------->  |
  | ...
  |
  | --- DELETE <statusUrl> -------> | (cleanup, release storage)
```

Honor `Retry-After` on poll responses. Servers throttle aggressively; ignoring `Retry-After` is the fastest way to get blocked.

---

## Kickoff Request

```http
POST /Group/{groupId}/$export HTTP/1.1
Host: fhir.example.org
Accept: application/fhir+json
Prefer: respond-async
Authorization: Bearer <SMART backend services token>

(no body for GET-style)
```

Optional parameters (passed as query parameters or in a `Parameters` body):

- `_outputFormat` — typically `application/fhir+ndjson` (default).
- `_type` — comma-separated FHIR resource types to include (e.g., `Patient,Condition,MedicationRequest,Observation`). Strongly recommended; without `_type` you receive every resource type the server supports, which is rarely what you want.
- `_since` — ISO 8601 timestamp; only resources with `meta.lastUpdated` after this are exported. Drives incremental refresh.
- `_typeFilter` — server-side filter expressed as a comma-separated list of FHIR search expressions. Example: `Observation?category=laboratory,Observation?category=vital-signs`. Server support varies — verify with your FHIR endpoint.
- `_elements` — comma-separated list of top-level elements to include (server may support, may not).

Response:

```http
HTTP/1.1 202 Accepted
Content-Location: https://fhir.example.org/bulk-status/12345
```

The `Content-Location` value is the **status URL**; persist it for polling.

---

## Polling for Status

```http
GET /bulk-status/12345 HTTP/1.1
Accept: application/json
Authorization: Bearer <token>
```

While running: `202 Accepted` with an optional `X-Progress` header.

When complete: `200 OK` with a manifest body:

```json
{
  "transactionTime": "2026-04-01T08:00:00Z",
  "request": "https://fhir.example.org/Group/123/$export?_type=Patient,Condition",
  "requiresAccessToken": true,
  "output": [
    {
      "type": "Patient",
      "url": "https://fhir.example.org/bulk-output/abc/Patient.ndjson",
      "count": 12345
    },
    {
      "type": "Condition",
      "url": "https://fhir.example.org/bulk-output/abc/Condition_0.ndjson",
      "count": 50000
    },
    {
      "type": "Condition",
      "url": "https://fhir.example.org/bulk-output/abc/Condition_1.ndjson",
      "count": 50000
    }
  ],
  "error": [],
  "deleted": []
}
```

Notes:

- One file per resource type, or several files per type if the server splits large outputs.
- `transactionTime` is the snapshot timestamp — store it; it becomes the next run's `_since`.
- `error` lists files with operation-outcome errors.
- `deleted` (Bulk Data v2) lists resources removed since the last export — important for keeping the lake in sync with deletions.

If failed: `4xx` / `5xx` with an `OperationOutcome` body.

---

## Downloading NDJSON Files

```http
GET /bulk-output/abc/Condition_0.ndjson HTTP/1.1
Accept: application/fhir+ndjson
Authorization: Bearer <token>   (if requiresAccessToken is true)
```

Stream to object storage directly (S3, ADLS Gen2, GCS) without buffering in memory. NDJSON files for a busy health system can be tens to hundreds of GB.

After all files are downloaded and persisted:

```http
DELETE /bulk-status/12345 HTTP/1.1
Authorization: Bearer <token>
```

Many servers retain output for only 24–72 hours; download promptly and delete the job when done.

---

## Filtering with _type, _since, _typeFilter

### _type: Pick the Resource Types You Need

Always set `_type`. A typical operational pull might be:

```
_type=Patient,Encounter,Condition,Observation,MedicationRequest,
      Procedure,DiagnosticReport,Immunization,AllergyIntolerance
```

For an oncology registry, add `Specimen`, `MolecularSequence` (if supported), `CarePlan`.

For an ACO, add `ExplanationOfBenefit`, `Claim`, `Coverage`.

### _since: Incremental Refresh

After the first full export, store `transactionTime`. The next export uses `_since={previous transactionTime}` to retrieve only new or updated resources. Critical because re-exporting from scratch nightly is wasteful and slow.

Watch out: `_since` filters by `meta.lastUpdated`. If a source system updates `lastUpdated` for cosmetic reasons (touched but not changed), incremental ingest may include unchanged resources — design downstream merge logic accordingly (use the resource hash or per-field comparison if churn is a problem).

### _typeFilter: Narrow Server-Side

`_typeFilter` is the way to restrict each resource type to a subset that matches a FHIR search expression. Example: `Observation?category=laboratory` excludes vital signs and social-history observations.

Server support varies — some implementations support a limited subset of FHIR search parameters in `_typeFilter`. Verify with your endpoint and fall back to client-side filtering if needed.

---

## Authentication and SMART Backend Services

Production `$export` typically uses **SMART Backend Services** authentication:

1. Pre-register the client with the FHIR server, providing a public key (JWK or JWKS URL).
2. At export time, the client signs a JWT with the matching private key, requesting scopes (e.g., `system/Patient.read system/Observation.read`).
3. Exchange the JWT for an OAuth 2.0 access token at the token endpoint.
4. Pass the access token in `Authorization: Bearer <token>` on the kickoff, polling, and download requests.

Tokens are short-lived (minutes to an hour); refresh as needed. Key rotation, JWKS publication, and audit logging of which client pulled which Group when are non-negotiable.

For CMS BCDA specifically: BCDA uses its own SMART Backend Services-style flow with sandbox and production environments. Verify current onboarding documentation at the BCDA developer portal.

---

## Consuming NDJSON in the Lake

### Land at Bronze

Persist NDJSON files in the bronze layer partitioned by source, resource type, and date:

```
bronze/
  fhir/
    epic-prod/
      Patient/
        yyyy=2026/mm=04/dd=01/Patient.ndjson
      Condition/
        yyyy=2026/mm=04/dd=01/Condition_0.ndjson
        yyyy=2026/mm=04/dd=01/Condition_1.ndjson
```

Capture the manifest alongside the data:

```
bronze/fhir/epic-prod/_manifest/yyyy=2026/mm=04/dd=01/manifest.json
```

The manifest records `transactionTime`, the originating Group, the parameter set, and the file checksums. It is the audit trail and the input to the next incremental run.

### Parse at Silver

Read NDJSON with a FHIR-aware library or with Spark / DuckDB / Snowflake native JSON support. For Spark on Delta:

```python
df = spark.read.json("bronze/fhir/epic-prod/Condition/yyyy=2026/mm=04/dd=01/*.ndjson")
```

Then project FHIR's nested structure into flat silver tables (one per resource type), with EMPI-resolved enterprise patient ID joined in.

### Flat FHIR / SQL-on-FHIR

The FHIR community has been standardizing a **ViewDefinition** approach (sometimes called Flat FHIR or SQL-on-FHIR) that projects nested FHIR into SQL-friendly columns via a portable definition. Tools like **Pathling** support FHIRPath-based analytics over FHIR data; some implementations now consume ViewDefinitions. Worth tracking — saves writing the projection logic by hand.

### Handle Deletions

Bulk Data v2 includes a `deleted` list of resource references removed since the last export. Apply these as tombstones at silver — otherwise patients withdrawn from the Group, encounters voided, or observations corrected silently linger in the lake.

---

## CMS BCDA Specifics

CMS BCDA (Beneficiary Claims Data API) is the FHIR Bulk Export endpoint for MSSP ACOs and certain other CMS programs to retrieve Medicare claims for attributed beneficiaries.

- **Scope**: Group-level export, where the Group is the ACO's attributed beneficiary list.
- **Resources**: `ExplanationOfBenefit` (claims), `Patient`, `Coverage`. Verify currently supported types.
- **Cadence**: a defined refresh schedule; verify per current BCDA documentation.
- **Authentication**: SMART Backend Services with credentials managed at the ACO level.
- **Sandbox**: BCDA provides a sandbox endpoint with synthetic data for development.

BCDA is one of the strongest production validators that FHIR Bulk Export works at scale.

---

## Common Pitfalls

- **No `_type` filter.** Pulling every resource type the server supports is expensive and rarely useful. Set `_type` explicitly.
- **No `_since` for incremental.** Re-exporting full history every night is wasteful and may trip server-side throttling.
- **Ignoring deletions.** Without applying the `deleted` list, the lake drifts from the source of truth on retractions and Group membership changes.
- **Ignoring `Retry-After` on polling.** Aggressive polling gets clients throttled or banned.
- **No manifest persistence.** Without the manifest, the next incremental run does not know its `_since`, and audit reconstruction is hard.
- **Not handling split files.** A single resource type may produce multiple NDJSON files; consume all of them, in any order.
- **Token expiry during a long download.** Long-running exports require token refresh; build that in.
- **Lake-side schema drift.** FHIR profiles evolve; silver-layer schemas must be versioned and tolerant of new optional elements without crashing the pipeline.
- **Trusting `meta.lastUpdated` semantics blindly across vendors.** Some EHRs update `lastUpdated` for non-substantive changes; design downstream merge logic to compare content where it matters.
- **Storing the manifest in the same trust boundary as direct identifiers.** If the manifest references files containing PHI, ensure governance, encryption, and access control cover the manifest path.
