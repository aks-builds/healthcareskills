# FHIR Bulk Data Export Guide

Reference for the FHIR Bulk Data Access IG (`$export`). Verify against the current published IG version (Bulk Data Access 2.0 at time of writing; check for revisions).

## Contents

- What Bulk Data Is For
- Three Scopes: System, Patient, Group
- Kickoff Request
- Status Polling
- Result Manifest
- Downloading NDJSON Files
- _typeFilter and _since
- Output Format and Compression
- Cancelling an Export
- Errors
- Authentication (SMART Backend Services)
- Server Considerations
- Common Failure Modes
- Worked Example

---

## What Bulk Data Is For

Bulk Data is for population-scale extraction of FHIR resources — typically for analytics, quality measures, payer data exchange, research cohorts, and Healthcare Operations under TEFCA / CMS rules. It is asynchronous and NDJSON-based, not a synchronous REST call.

It is **not** for real-time clinical lookups (use search), and not for syncing tiny deltas (use Subscription or polling).

---

## Three Scopes

| Scope | Endpoint | Use |
|-------|----------|-----|
| System | `GET /$export` | Everything the requester is allowed to see (system access) |
| Patient (all patients) | `GET /Patient/$export` | All patients the requester is allowed to see |
| Group | `GET /Group/{id}/$export` | A specific Group of patients (e.g., a cohort, an enrolled population) |

The most common scope in practice is **Group** — define your cohort with a `Group` resource, then export.

---

## Kickoff Request

Required headers:

- `Accept: application/fhir+json`
- `Prefer: respond-async`

Optional query parameters:

| Parameter | Purpose |
|-----------|---------|
| `_type` | Comma-separated resource types to include |
| `_since` | FHIR instant — only return resources modified on/after this time |
| `_typeFilter` | URL-encoded search expressions applied per resource type |
| `_outputFormat` | `application/fhir+ndjson` (default), gzip-encoded NDJSON, parquet (server-dependent) |
| `_elements` | Limit elements returned (server-dependent) |
| `includeAssociatedData` | LatestProvenanceResources, Relevant, _include:iterate (server-dependent) |

Example:

```
GET /Group/cohort-001/$export?_type=Patient,Condition,Observation&_since=2025-01-01T00:00:00Z
Accept: application/fhir+json
Prefer: respond-async
Authorization: Bearer <token>
```

Response:

```
HTTP/1.1 202 Accepted
Content-Location: https://server.example.org/fhir/exports/abc123/status
```

The `Content-Location` URL is the **status endpoint** — store it and poll it.

---

## Status Polling

```
GET https://server.example.org/fhir/exports/abc123/status
Authorization: Bearer <token>
```

Three states:

| State | Response |
|-------|----------|
| In progress | `202 Accepted` with optional `X-Progress` header containing free-text progress |
| Complete | `200 OK` with JSON manifest in the body |
| Failed | `4xx` / `5xx` with `OperationOutcome` |

Respect `Retry-After` headers when polling — don't hammer the status endpoint.

---

## Result Manifest

On `200 OK`, the body is a JSON manifest:

```json
{
  "transactionTime": "2026-03-01T12:34:56Z",
  "request": "https://server.example.org/fhir/Group/cohort-001/$export?_type=Patient,Condition,Observation",
  "requiresAccessToken": true,
  "output": [
    { "type": "Patient", "url": "https://server.example.org/fhir/exports/abc123/Patient.001.ndjson" },
    { "type": "Patient", "url": "https://server.example.org/fhir/exports/abc123/Patient.002.ndjson" },
    { "type": "Condition", "url": "https://server.example.org/fhir/exports/abc123/Condition.001.ndjson" },
    { "type": "Observation", "url": "https://server.example.org/fhir/exports/abc123/Observation.001.ndjson" }
  ],
  "deleted": [
    { "type": "Bundle", "url": "https://server.example.org/fhir/exports/abc123/deleted.001.ndjson" }
  ],
  "error": []
}
```

Key fields:

- `transactionTime` — the server's logical time for this export; use as `_since` on the next run.
- `requiresAccessToken` — most servers require auth on the file URLs too.
- `output` — array of `{type, url}` pairs. One resource type may produce multiple files.
- `deleted` — Bundle resources representing tombstones (per the spec).
- `error` — OperationOutcome resources for partial failures.

---

## Downloading NDJSON Files

Each file is **newline-delimited JSON** — one FHIR resource per line, no commas, no array wrapping.

```
GET https://server.example.org/fhir/exports/abc123/Patient.001.ndjson
Accept: application/fhir+ndjson
Authorization: Bearer <token>
```

Parse line-by-line; large files don't fit in memory. Many servers serve files gzipped — honor `Content-Encoding: gzip`.

---

## _typeFilter and _since

`_since` is simple: filter by `meta.lastUpdated`. Use the previous export's `transactionTime` to do incremental pulls.

`_typeFilter` is the richer mechanism — apply a FHIR search expression per resource type:

```
_typeFilter=Observation?category=laboratory&_typeFilter=Condition?clinical-status=active
```

URL-encode the inner `?` and `&`:

```
_typeFilter=Observation%3Fcategory%3Dlaboratory
```

Server support for `_typeFilter` varies; check the CapabilityStatement / IG conformance for the server.

---

## Output Format and Compression

- Default: `application/fhir+ndjson`.
- Many servers offer gzip or other compression — negotiate via `Accept-Encoding`.
- Some servers offer Parquet / CSV output via vendor extensions — not part of the core spec.

---

## Cancelling an Export

```
DELETE https://server.example.org/fhir/exports/abc123/status
```

Response: `202 Accepted` (cancellation requested). The server may continue running but should stop producing new files.

---

## Errors

If kickoff or status returns `4xx` / `5xx`, parse the `OperationOutcome`:

```json
{
  "resourceType": "OperationOutcome",
  "issue": [{
    "severity": "error",
    "code": "too-costly",
    "diagnostics": "Group cohort-001 has 4.2M members; export refused. Use a smaller Group or contact admin."
  }]
}
```

Common error codes: `too-costly`, `not-supported`, `security`, `processing`, `timeout`.

---

## Authentication (SMART Backend Services)

Bulk export endpoints almost always use **SMART Backend Services**:

1. Client registers a public key (JWK) with the auth server.
2. To get an access token, client constructs a signed JWT (JWS) with itself as issuer/subject, the token endpoint as audience, and an `exp`.
3. Client POSTs to the token endpoint with `grant_type=client_credentials`, `client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer`, and the JWT.
4. Server returns an access token with `system/*.read` (or scoped) permissions.

Defer to the smart-on-fhir skill for full backend services flow details, key registration, and JWT claim requirements.

---

## Server Considerations

When designing an export endpoint:

- Materialize on a read-replica / data lake, not the OLTP store.
- Use object storage (S3, GCS, Azure Blob) for output files with signed URLs.
- Enforce per-client export rate limits.
- Capture audit events per file download.
- Honor `Prefer: respond-async` strictly; clients depend on the contract.
- Make file URLs unguessable (long opaque IDs).

---

## Common Failure Modes

- **Polling too fast** → server rate-limits or refuses.
- **No Authorization on file URLs** → 403; check `requiresAccessToken` in the manifest.
- **Splitting NDJSON files at comma** instead of newline → parse failures.
- **Not handling gzip** → reads garbage bytes.
- **Treating partial errors as fatal** — the `error` array contains OperationOutcomes for skipped resources but the rest of the export is still valid.
- **Re-running full exports nightly** instead of `_since` incremental — wasteful and slow.
- **Group resource changes mid-export** — most servers snapshot membership at kickoff; verify the server's behavior.

---

## Worked Example: Nightly Incremental Export

1. Day 0: Full export of `Group/cohort-001` for `_type=Patient,Condition,Observation,MedicationRequest`. Record `transactionTime = T0` from the manifest.
2. Day 1: Kickoff `?_type=Patient,Condition,Observation,MedicationRequest&_since=T0`. Poll until 200. Process new/changed resources and tombstones. Record `transactionTime = T1`.
3. Day 2: Same with `_since=T1`. Repeat.

Store `transactionTime` per cohort + per resource-type set, not globally. If you add a resource type to `_type` on day N, you need a full re-pull for that type (without `_since`) before resuming incrementals.
