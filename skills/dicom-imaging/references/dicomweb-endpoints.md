# DICOMweb Endpoints

Practical reference for the DICOMweb (PS3.18) RESTful services — QIDO-RS, WADO-RS, WADO-URI, STOW-RS, and UPS-RS. All examples use **synthetic UIDs**. Verify the current PS3.18 specification before relying on edge-case behavior.

## Contents

- URL Structure and Versions
- QIDO-RS — Query
- WADO-RS — Retrieve
- WADO-URI — Legacy Single-Instance Retrieve
- STOW-RS — Store
- UPS-RS — Workflow / Worklist
- Content Types
- Authentication
- CORS and Browser Clients
- Common Failure Modes
- Tooling

---

## URL Structure and Versions

Base URL is whatever the server publishes. Path patterns:

```
/studies
/studies/{StudyInstanceUID}
/studies/{StudyInstanceUID}/series
/studies/{StudyInstanceUID}/series/{SeriesInstanceUID}
/studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/instances
/studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/instances/{SOPInstanceUID}
/studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/instances/{SOPInstanceUID}/frames/{FrameList}
/studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/instances/{SOPInstanceUID}/bulkdata/{BulkdataURI}
/studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/instances/{SOPInstanceUID}/metadata
/studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/rendered
/studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/instances/{SOPInstanceUID}/rendered
/workitems
/workitems/{WorkitemUID}
```

Many servers also expose `/qido-rs`, `/wado-rs`, `/stow-rs` prefixes for the same paths — check the conformance statement.

---

## QIDO-RS — Query

QIDO-RS replaces C-FIND. Searches use HTTP GET with query parameters.

### Patterns

| Pattern | Returns |
|---------|---------|
| `GET /studies?{params}` | Matching studies |
| `GET /series?{params}` | Matching series across all studies |
| `GET /studies/{StudyInstanceUID}/series?{params}` | Matching series in one study |
| `GET /instances?{params}` | Matching instances across the server |
| `GET /studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/instances?{params}` | Matching instances in one series |

### Common Query Parameters

| Parameter | Example | Notes |
|-----------|---------|-------|
| `PatientID` | `PatientID=MRN-123456` | Exact match unless server supports `*` |
| `PatientName` | `PatientName=DOE^JANE*` | PN-formatted; some servers accept wildcards |
| `StudyDate` | `StudyDate=20260101-20260301` | Range syntax |
| `StudyTime` | `StudyTime=080000-180000` | Range |
| `AccessionNumber` | `AccessionNumber=ACC-555` | |
| `ModalitiesInStudy` | `ModalitiesInStudy=CT,MR` | Comma-separated |
| `StudyInstanceUID` | `StudyInstanceUID=1.2.840.113619.2.55.3.604688119...001` | |
| `SeriesInstanceUID` | `SeriesInstanceUID=...` | |
| `SOPInstanceUID` | `SOPInstanceUID=...` | |
| `Modality` | `Modality=CT` | Series-level |
| `00100020` | `00100020=MRN-123456` | Alternative — use the tag in (gggg)(eeee) hex |
| `includefield` | `includefield=00081030,0008103E` | Include otherwise-omitted attributes |
| `includefield=all` | | All matched attributes |
| `fuzzymatching` | `fuzzymatching=true` | If supported |
| `limit` / `offset` | `limit=50&offset=0` | Pagination |

### Example

```
GET /studies?PatientID=MRN-123456&StudyDate=20260101-20260301&Modality=CT&includefield=all&limit=50
Accept: application/dicom+json
Authorization: Bearer <token>
```

Response is `application/dicom+json` — a JSON array where each element is a DICOM dataset in the DICOM JSON model (each tag becomes a property with `vr` and `Value`).

```json
[
  {
    "00080020": { "vr": "DA", "Value": ["20260115"] },
    "00080050": { "vr": "SH", "Value": ["ACC-555"] },
    "00080061": { "vr": "CS", "Value": ["CT"] },
    "00100010": { "vr": "PN", "Value": [{ "Alphabetic": "DOE^JANE" }] },
    "00100020": { "vr": "LO", "Value": ["MRN-123456"] },
    "0020000D": { "vr": "UI", "Value": ["1.2.840.113619.2.55.3.604688119.971.1740000000.001"] }
  }
]
```

---

## WADO-RS — Retrieve

WADO-RS replaces C-MOVE/C-GET. Use HTTP GET on study/series/instance/frame/bulkdata/metadata URLs.

### Retrieve a Full Study

```
GET /studies/1.2.840.113619.2.55.3.604688119.971.1740000000.001
Accept: multipart/related; type="application/dicom"; transfer-syntax=*
Authorization: Bearer <token>
```

Returns a multipart body with one DICOM file per part. `transfer-syntax=*` lets the server pick whatever it has.

### Retrieve a Specific Transfer Syntax

```
Accept: multipart/related; type="application/dicom"; transfer-syntax=1.2.840.10008.1.2.1
```

If the server doesn't have that transfer syntax, it returns 406 Not Acceptable.

### Retrieve Metadata Only (no pixel data)

```
GET /studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/instances/{SOPInstanceUID}/metadata
Accept: application/dicom+json
```

Returns DICOM JSON with `BulkDataURI` for large fields (PixelData, encapsulated sequences) so clients can fetch them on demand.

### Retrieve a Frame

```
GET /studies/{Study}/series/{Series}/instances/{Instance}/frames/1
Accept: multipart/related; type="application/octet-stream"
```

Frame list can be comma-separated: `frames/1,5,10`.

### Rendered Retrieval (JPEG / PNG)

```
GET /studies/{Study}/series/{Series}/instances/{Instance}/rendered
Accept: image/jpeg
```

Supports query params: `quality=80`, `window=center,width`, `viewport=cols,rows`, `iccprofile`, `annotation`.

### Bulkdata Retrieval

```
GET /studies/{Study}/series/{Series}/instances/{Instance}/bulkdata/7fe00010
Accept: application/octet-stream
```

For pixel data and other large encapsulated content. The BulkDataURI is returned in `/metadata` responses.

---

## WADO-URI — Legacy Single-Instance Retrieve

Older interface, single instance only. Uses query parameters, not path segments.

```
GET /wado?requestType=WADO
  &studyUID=1.2.840.113619.2.55.3.604688119.971.1740000000.001
  &seriesUID=1.2.840.113619.2.55.3.604688119.971.1740000000.002
  &objectUID=1.2.840.113619.2.55.3.604688119.971.1740000000.003
  &contentType=application/dicom
```

Or render to image:

```
?contentType=image/jpeg
&rows=512
&columns=512
&windowCenter=40
&windowWidth=400
&imageQuality=80
```

Largely superseded by WADO-RS for new builds — keep for legacy viewer compatibility.

---

## STOW-RS — Store

STOW-RS replaces C-STORE for the REST world.

### Store to a Study (server determines target)

```
POST /studies
Content-Type: multipart/related; type="application/dicom"; boundary=---boundary
Authorization: Bearer <token>

-----boundary
Content-Type: application/dicom
Content-ID: <urn:uuid:0001>

<DICOM bytes here>

-----boundary
Content-Type: application/dicom
Content-ID: <urn:uuid:0002>

<DICOM bytes here>

-----boundary--
```

### Store to a Specific Study UID

```
POST /studies/1.2.840.113619.2.55.3.604688119.971.1740000000.001
```

The server validates that each instance's StudyInstanceUID matches and accepts/rejects per-instance.

### Response

```
HTTP/1.1 200 OK
Content-Type: application/dicom+xml

<NativeDicomModel>
  <DicomAttribute tag="00081199" vr="SQ" keyword="ReferencedSOPSequence">
    <Item number="1">
      <DicomAttribute tag="00081150" vr="UI" keyword="ReferencedSOPClassUID">...</DicomAttribute>
      <DicomAttribute tag="00081155" vr="UI" keyword="ReferencedSOPInstanceUID">...</DicomAttribute>
    </Item>
  </DicomAttribute>
  <!-- FailedSOPSequence (00081198) lists per-instance failures with reason codes -->
</NativeDicomModel>
```

Failures are not whole-request failures — individual instances may fail while others succeed. Always parse the response.

---

## UPS-RS — Workflow / Worklist

Unified Procedure Step REST services — the REST equivalent of MWL + MPPS.

### Common Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/workitems` | Create a UPS workitem |
| GET | `/workitems?{filters}` | Query workitems |
| GET | `/workitems/{WorkitemUID}` | Retrieve a workitem |
| PUT | `/workitems/{WorkitemUID}/state` | Change state (SCHEDULED, IN PROGRESS, CANCELED, COMPLETED) |
| POST | `/workitems/{WorkitemUID}/cancelrequest` | Request cancellation |
| POST | `/workitems/{WorkitemUID}/subscribers/{SubscriberAET}` | Subscribe to changes |

UPS-RS is the modern replacement for MWL/MPPS in DICOMweb-native workflows.

---

## Content Types

| Type | Use |
|------|-----|
| `application/dicom` | Raw DICOM file (Part 10) |
| `application/dicom+json` | DICOM JSON model |
| `application/dicom+xml` | DICOM Native XML model |
| `application/octet-stream` | Bulkdata, pixel-data frames |
| `image/jpeg` / `image/png` / `image/gif` | Rendered output |
| `multipart/related` | Container for multi-part responses |

`Accept` header negotiation drives the response format. `transfer-syntax=*` on retrieve lets the server pick any available syntax.

---

## Authentication

Most production deployments use one of:

- **OAuth 2.0 / OpenID Connect** — Bearer tokens via the IHE IUA (Internet User Authorization) profile.
- **mTLS** — client certificate auth.
- **API key** — vendor-specific header (less common in healthcare).
- **SAML 2.0** — via XUA for cross-enterprise federation.

The IHE IUA profile defines required scopes and token formats; defer to the ihe-profiles skill for IUA specifics.

---

## CORS and Browser Clients

For zero-footprint viewers (OHIF, etc.) hosted on a different origin than the DICOMweb server, the server must:

- Allow CORS preflight (`OPTIONS`) with appropriate headers.
- Expose `Content-Type` and any custom auth headers.
- Allow large response bodies.

Many DICOMweb gateways have CORS off by default in production; turn it on intentionally and scope origins narrowly.

---

## Common Failure Modes

- **406 Not Acceptable on retrieve** — server doesn't have the requested transfer syntax. Re-request with `transfer-syntax=*` or a syntax the server supports.
- **STOW-RS partial failures** — non-empty FailedSOPSequence; parse and retry only the failed instances.
- **QIDO-RS server limits** — many servers cap `limit` to 100 or 500; pagination required.
- **Mismatched StudyInstanceUID on STOW** — server rejects the instance.
- **Authorization on bulkdata URIs** — some servers require the same token; others use signed URLs. Read the conformance statement.
- **Multipart parsing bugs** — clients sometimes split on the wrong boundary string. Use a tested library (dcm4che, pydicom + dicomweb-client, dcmjs).
- **CORS pre-flight blocked** — viewer fails silently in browser DevTools.
- **Token expiration mid-retrieve** — long study downloads need refresh-token handling.

---

## Tooling

- **dcm4che / dcm4chee Arc**: reference open-source DICOMweb server and toolkit (Java).
- **pydicom + dicomweb-client**: Python client and tools.
- **dcmjs / dicomweb-client.js**: JavaScript DICOMweb client.
- **OHIF Viewer**: open-source zero-footprint viewer that consumes DICOMweb.
- **DCMTK**: classic DIMSE toolkit; some DICOMweb support via storescp/storescu and DCMTK web tools.
- **Google Cloud Healthcare API, Azure Health Data Services, AWS HealthLake imaging**: managed DICOMweb services (verify current feature parity with PS3.18).
