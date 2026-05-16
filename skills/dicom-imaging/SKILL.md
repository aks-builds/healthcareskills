---
name: dicom-imaging
description: When the user wants to design, integrate, or troubleshoot DICOM medical imaging systems. Use when the user mentions "DICOM," "PACS," "VNA," "modality," "C-STORE," "C-FIND," "C-MOVE," "C-ECHO," "DIMSE," "DICOMweb," "WADO," "QIDO," "STOW," "UPS-RS," "MWL," "Modality Worklist," "MPPS," "Structured Report," "SR," "SOP Class," "AE title," "transfer syntax," "DICOM anonymization," or specific modalities like CT/MR/CR/DX/US/NM/PT/MG. For non-imaging clinical data exchange, see fhir-integration or hl7-v2. For AI on imaging, see medical-imaging-ai.
metadata:
  version: 1.0.0
---

# DICOM Imaging

You are an expert in the DICOM (Digital Imaging and Communications in Medicine) standard — the basis for almost all medical imaging exchange. Your goal is to help engineers and PACS administrators design integrations between modalities, PACS, VNAs, viewers, and analytics platforms, using DIMSE and DICOMweb correctly. Never invent tag values, SOP class UIDs, or transfer syntaxes — verify against the published DICOM standard (PS3.x parts) when in doubt.

## Initial Assessment

Check `.agents/healthcare-context.md` (fallback: `.claude/healthcare-context.md`) first. Useful sections:

- **PACS / VNA vendor** and version
- **Modalities** (CT, MR, CR/DX, US, NM, PT, MG, etc.) and their vendors
- **Connectivity model**: classic DIMSE only, DICOMweb-enabled, both
- **Cross-enterprise sharing**: XCA-I, IHE XDS-I.b, vendor cloud
- **AI workflows**: are inference results returned as Secondary Capture, SR, or non-DICOM?

Ask only what's missing.

---

## Object Model

DICOM organizes data hierarchically:

```
Patient
└── Study (StudyInstanceUID)
    └── Series (SeriesInstanceUID)
        └── Instance / SOP Instance (SOPInstanceUID)
```

A **SOP Instance** is one DICOM object — typically a single image, but can be a structured report, presentation state, encapsulated PDF, segmentation, or RT plan. Each SOP Instance is described by:

- **SOP Class UID** — what kind of object (e.g., CT Image Storage)
- **SOP Instance UID** — globally unique ID for this specific object
- **Transfer Syntax UID** — how it is encoded on the wire / disk

UIDs are immutable and globally unique. Generate them from a registered root (your organization's OID) — do not re-use across studies.

---

## Common SOP Classes (Storage)

| Modality | Common SOP Class |
|----------|------------------|
| CT | CT Image Storage |
| MR | MR Image Storage |
| CR / DX | Computed Radiography / Digital X-Ray Image Storage |
| US | Ultrasound Image Storage / Ultrasound Multi-frame Image Storage |
| NM | Nuclear Medicine Image Storage |
| PT | Positron Emission Tomography Image Storage |
| MG | Digital Mammography X-Ray Image Storage (for Presentation / for Processing) |
| RF / XA | X-Ray Radiofluoroscopic / X-Ray Angiographic Image Storage |
| OT | Secondary Capture Image Storage (used heavily for AI results) |
| SR | Enhanced SR / Comprehensive SR / Basic Text SR |
| SEG | Segmentation Storage |
| PR | Grayscale / Color Softcopy Presentation State Storage |
| RT* | RT Plan / RT Structure Set / RT Dose / RT Image Storage |
| Encapsulated | Encapsulated PDF / CDA Storage |

Always look up the current SOP Class UID — they are long URN-style identifiers and easy to mistype.

---

## DIMSE Services

DIMSE (DICOM Message Service Element) is the classic verb-based protocol over TCP, using association negotiation.

| Service | Verb | Use |
|---------|------|-----|
| C-ECHO | "ping" | Verify association connectivity |
| C-STORE | push | Send a SOP Instance to a peer |
| C-FIND | query | Search for studies/series/instances/worklist entries |
| C-MOVE | pull (3-party) | Tell the source to send objects to a third AE |
| C-GET | pull (2-party) | Retrieve objects on the same association |
| C-CANCEL | cancel | Cancel a pending operation |
| N-CREATE / N-SET / N-ACTION / N-EVENT-REPORT / N-GET | normalized | Used for MPPS, Print, Storage Commitment, UPS, etc. |

### AE Titles

Each DICOM application has an **AE Title** (Application Entity Title, max 16 chars, no leading/trailing spaces). Association negotiation requires the calling AE, called AE, presentation contexts (SOP Class + acceptable Transfer Syntaxes), and supported roles. Misconfigured AE titles or whitelists are the #1 cause of "no images appearing" tickets.

### Query / Retrieve Models

C-FIND/C-MOVE/C-GET use models:

- Patient Root Q/R Information Model
- Study Root Q/R Information Model
- Patient/Study Only (legacy)

Pick at the negotiation stage — usually Study Root.

---

## Transfer Syntaxes

A Transfer Syntax UID encodes:

- **Byte order**: Little Endian (vast majority) vs. Big Endian (deprecated)
- **VR encoding**: Explicit vs. Implicit
- **Compression**: None / JPEG (lossy/lossless) / JPEG 2000 (lossy/lossless) / RLE / JPEG-LS / HEVC / MPEG2/4 / etc.

Defaults to memorize:

| Syntax | Use |
|--------|-----|
| Implicit VR Little Endian | DICOM default — required to support |
| Explicit VR Little Endian | Most common on disk |
| JPEG Baseline (Process 1) | Lossy 8-bit JPEG |
| JPEG Lossless, Non-Hierarchical (Process 14, SV1) | Lossless JPEG, common for CR/DX/MG |
| JPEG 2000 Lossless / JPEG 2000 | Modern lossless / lossy |
| RLE Lossless | Run-length encoding |

Always negotiate at least Implicit VR Little Endian; compressed streams require matching codecs on both sides.

---

## DICOMweb (RESTful DICOM)

The "web" services defined in PS3.18. URLs follow `/studies/{StudyInstanceUID}/series/{SeriesInstanceUID}/instances/{SOPInstanceUID}`.

| Service | Method | Purpose |
|---------|--------|---------|
| QIDO-RS | GET | Query studies/series/instances by attributes (replaces C-FIND) |
| WADO-RS | GET | Retrieve instances, frames, bulkdata, metadata (replaces C-MOVE/C-GET) |
| WADO-URI | GET | Legacy WADO — single instance, often rendered (JPEG/PNG) |
| STOW-RS | POST | Store one or more instances (replaces C-STORE) |
| UPS-RS | various | Unified Procedure Step — worklist + status, REST equivalent of MWL/MPPS |

### Examples (synthetic UIDs)

QIDO-RS — find studies for a patient:
```
GET /studies?PatientID=MRN-123456&StudyDate=20260101-20260301
Accept: application/dicom+json
```

WADO-RS — retrieve a series as multipart DICOM:
```
GET /studies/1.2.840.113619.2.55.3.604688119.971.1740000000.001/series/1.2.840.113619.2.55.3.604688119.971.1740000000.002
Accept: multipart/related; type="application/dicom"; transfer-syntax=*
```

STOW-RS — store a study:
```
POST /studies
Content-Type: multipart/related; type="application/dicom"; boundary=---boundary
```

### Authentication

DICOMweb commonly uses OAuth 2.0 bearer tokens (IHE IUA profile) or mTLS. Classic DIMSE has no built-in auth — rely on AE whitelisting + TLS (DICOM TLS).

---

## Modality Worklist (MWL) and MPPS

- **MWL (Modality Worklist)**: SCP exposes scheduled procedure steps; modalities query (C-FIND with Modality Worklist Information Model) at the start of an exam to populate patient demographics and accession numbers.
- **MPPS (Modality Performed Procedure Step)**: modality sends `N-CREATE` (in progress) and `N-SET` (completed/discontinued) to record actual performance, including dose for CT/XA.

Together, MWL + MPPS keep RIS, modality, and PACS synchronized.

---

## Key DICOM Tags

Tags are written `(GGGG,EEEE)` (group, element).

| Tag | Name | Notes |
|-----|------|-------|
| (0010,0010) | PatientName | Format `Last^First^Middle^Prefix^Suffix` |
| (0010,0020) | PatientID | Stable per assigning authority; can collide across sites |
| (0010,0030) | PatientBirthDate | YYYYMMDD |
| (0010,0040) | PatientSex | M / F / O |
| (0020,000D) | StudyInstanceUID | |
| (0020,000E) | SeriesInstanceUID | |
| (0008,0018) | SOPInstanceUID | |
| (0008,0016) | SOPClassUID | |
| (0008,0060) | Modality | CT, MR, CR, etc. |
| (0008,0050) | AccessionNumber | Order identifier from RIS |
| (0008,0070) | Manufacturer | |
| (0008,1030) | StudyDescription | |
| (0008,103E) | SeriesDescription | |
| (0010,1010) | PatientAge | |
| (0018,0050) | SliceThickness | |
| (0028,0010) / (0028,0011) | Rows / Columns | Image dimensions |

---

## Structured Reports (SR)

DICOM SR encodes coded findings as a tree of content items (numeric, text, code, image reference, container). Templates (e.g., TID 1500 for measurement reports, TID 4019 for CT dose) define expected structure. SR is the standard way to return AI-derived measurements and CAD output that downstream systems can parse — Secondary Capture images are a poor substitute.

---

## PACS, VNA, and DICOMweb Proxy Architecture

- **PACS**: stores and serves studies; classic DIMSE; often vendor-locked viewers.
- **VNA (Vendor-Neutral Archive)**: a long-term archive that holds DICOM (and often non-DICOM media) in a vendor-neutral way; typically front-ends multiple departmental PACS.
- **DICOMweb proxy / gateway**: translates between classic DIMSE and DICOMweb (and between DICOM TLS and OAuth). Common in cloud-edge deployments.
- **Zero-footprint viewers** (e.g., OHIF) consume DICOMweb directly.

A common pattern: modalities → router/gateway → on-prem PACS → VNA (long-term) + DICOMweb proxy → cloud AI / viewers.

---

## Security and Anonymization

- **DICOM TLS** is defined in PS3.15; mTLS is preferred for DIMSE associations.
- **Audit**: DICOM defines audit events aligned with IHE ATNA (see `ihe-profiles`).
- **De-identification**: PS3.15 Annex E defines the **Application Confidentiality Profile** and a long list of options (Basic Profile, Retain Patient Characteristics Option, Retain Longitudinal Temporal Information Option, Clean Pixel Data Option, etc.). Use a vetted library (e.g., DCMTK, pixelmed, dicom-anonymizer, GDCM) — never hand-roll. Pixel-burned PHI (common in US, MG, NM) requires OCR or manual review.

Always document which profile and which options were applied — auditors will ask.

---

## End-to-End Example: A CT Order

1. Order placed in RIS → MWL entry created.
2. CT tech selects worklist entry → modality C-FINDs MWL → fills patient demographics into scan.
3. MPPS N-CREATE sent at scan start; N-SET COMPLETED at end, with dose info.
4. Modality C-STOREs images to PACS (often via a router/gateway).
5. PACS notifies RIS / EHR via HL7 v2 ORU or IHE Imaging Document Source.
6. Radiologist dictates → SR / report → distributed.
7. Long-term archive replicates to VNA; DICOMweb proxy exposes images to AI and zero-footprint viewers.

---

## Common Pitfalls

- AE title / IP / port whitelist misconfiguration. Always C-ECHO before C-STORE.
- Compressed transfer syntaxes not negotiated → "association abort" or pixel data corruption.
- Reusing UIDs across studies — illegal; breaks ingest deduplication.
- Patient ID collisions across sites — use Issuer of Patient ID (0010,0021) to disambiguate.
- Burned-in PHI in pixel data missed by tag-only anonymization.
- Treating Secondary Capture as the canonical AI output (poor querying, no measurements) when SR is more appropriate.
- Assuming DICOMweb endpoints support all of QIDO/WADO/STOW — many do partial implementations.

---

## Task-Specific Questions

1. What PACS / VNA vendor and version are we integrating with, and what does its IHE/DICOM Conformance Statement say it supports?
2. Are we using classic DIMSE (C-ECHO/C-STORE/C-FIND/C-MOVE), DICOMweb (QIDO-RS / WADO-RS / STOW-RS / UPS-RS), or both?
3. Which modalities are involved (CT, MR, CR/DX, US, NM, PT, MG, RF/XA, RT, SR) and which SOP Classes do they produce?
4. What AE titles, IPs, ports, and presentation-context whitelists need to be configured on each side?
5. Which transfer syntaxes must be negotiated (Implicit VR LE baseline plus compressed: JPEG Lossless, JPEG 2000, RLE, etc.)?
6. Is de-identification in scope — and if so, which PS3.15 Annex E profile/options apply, and is there burned-in pixel PHI to handle?
7. How will downstream systems consume results (FHIR ImagingStudy/DiagnosticReport, HL7 v2 ORU, DICOM SR, secondary capture)?

---

## Related Skills

- **fhir-integration** — `ImagingStudy` and `DiagnosticReport` resources bridge DICOM to FHIR ecosystems
- **hl7-v2** — RIS/EHR notifications about imaging still typically use ORU^R01, ORM^O01, ADT
- **ihe-profiles** — SWF, XDS-I.b, XCA-I, IID, RAD-69 (cross-enterprise sharing) are all DICOM-centric
- **medical-imaging-ai** — model deployment, SR output, and Class II/III SaMD considerations
- **healthcare-cybersecurity** — DICOM TLS, network segmentation, and modality patching realities
- **audit-logging** — DICOM audit events and ATNA-aligned logging
- **phi-handling** — pixel-burned PHI and de-identification choices
