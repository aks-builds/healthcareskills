# IHE Actor and Transaction Tables

Quick reference to the actors and transactions defined by the most commonly deployed IHE profiles. **Verify against the current IHE Technical Framework volumes** — transaction numbers and required option lists can shift. Transactions are numbered as `Domain-NN` (ITI-41, RAD-69, etc.).

## Contents

- How to Read These Tables
- XDS.b Cross-Enterprise Document Sharing
- XDR Cross-Enterprise Document Reliable Interchange
- XCA Cross-Community Access
- XCPD Cross-Community Patient Discovery
- MHD Mobile access to Health Documents
- PIX (HL7 v2) / PIXm (FHIR)
- PDQ (HL7 v2) / PDQm (FHIR)
- ATNA Audit Trail and Node Authentication
- BPPC Basic Patient Privacy Consents
- IUA Internet User Authorization
- SWF.b Scheduled Workflow (Radiology)
- XDS-I.b Cross-Enterprise Document Sharing for Imaging
- Profile Grouping Notes

---

## How to Read These Tables

Each profile defines:

- **Actors** — abstract roles a system plays
- **Transactions** — numbered exchanges between actors
- **Options** — optional capabilities a system may declare
- **Required grouping** — actors that MUST be grouped with another actor from another profile

Vendors publish an **IHE Integration Statement** listing which actors, options, and transports they support. Always exchange these before integration design.

---

## XDS.b Cross-Enterprise Document Sharing

### Actors

| Actor | Role |
|-------|------|
| Document Source | Submits documents and metadata |
| Document Repository | Stores documents; forwards metadata to Registry |
| Document Registry | Indexes document metadata; answers queries |
| Document Consumer | Queries Registry and retrieves from Repository |
| Patient Identity Source | Feeds patient identifiers to Registry |
| Integrated Source/Repository | Combined Source + Repository in one system |
| On-Demand Document Source | Generates documents on demand at retrieval time |
| Document Administrator | Maintains the Registry / Repository (administrative) |

### Transactions

| Transaction | From → To | Purpose |
|-------------|-----------|---------|
| ITI-8 Patient Identity Feed | Patient Identity Source → Document Registry | HL7 v2 ADT feed of patient identities (legacy) |
| ITI-18 Registry Stored Query | Document Consumer → Document Registry | Query documents using stored queries (FindDocuments, GetDocuments, FindFolders, FindSubmissionSets, GetAll, GetFolders, etc.) |
| ITI-41 Provide and Register Document Set-b | Document Source → Document Repository | Submit documents + metadata |
| ITI-42 Register Document Set-b | Document Repository → Document Registry | Forward metadata to Registry |
| ITI-43 Retrieve Document Set | Document Consumer → Document Repository | Retrieve document(s) by uniqueId |
| ITI-44 Patient Identity Feed HL7 V3 | Patient Identity Source → Document Registry | HL7 v3 patient identity feed (alternative to ITI-8) |
| ITI-45 PIX Query (V3) | Patient Identifier Cross-reference Consumer → PIX Manager | Query PIX (paired with XDS) |
| ITI-46 PIX Update Notification (V3) | PIX Manager → PIX Consumer | Notify of identifier changes |
| ITI-51 Multi-Patient Stored Query | Document Consumer → Document Registry | Query across multiple patients |
| ITI-61 Register On-Demand Document Entry | On-Demand Document Source → Document Registry | Register on-demand document entries |
| ITI-62 Delete Document Set | Document Administrator → Document Registry | Administrative delete |

### Required Grouping

- Document Source SHALL be grouped with ATNA Secure Node (or Secure Application) and CT Time Client.
- Document Repository, Document Registry, Document Consumer: same grouping with ATNA + CT.
- Patient Identity Source: same.

### Actor / Transaction Matrix

| Actor | ITI-8 / 44 | ITI-18 | ITI-41 | ITI-42 | ITI-43 | ITI-51 |
|-------|----------|--------|--------|--------|--------|--------|
| Document Source | — | — | Send | — | — | — |
| Document Repository | — | — | Recv | Send | Send (response) | — |
| Document Registry | Recv | Recv | — | Recv | — | Recv |
| Document Consumer | — | Send | — | — | Send | Send |
| Patient Identity Source | Send | — | — | — | — | — |

---

## XDR Cross-Enterprise Document Reliable Interchange

### Actors

| Actor | Role |
|-------|------|
| Document Source (XDR) | Submits documents to a Document Recipient |
| Document Recipient (XDR) | Receives and stores |

### Transactions

| Transaction | From → To | Purpose |
|-------------|-----------|---------|
| ITI-41 Provide and Register Document Set-b | Document Source → Document Recipient | Reuses ITI-41 in a point-to-point context |

Required grouping: ATNA + CT on both actors.

---

## XCA Cross-Community Access

### Actors

| Actor | Role |
|-------|------|
| Initiating Gateway | Issues cross-community query / retrieve from one community on behalf of consumers |
| Responding Gateway | Receives cross-community queries; satisfies them from its community's Registry/Repository |

### Transactions

| Transaction | From → To | Purpose |
|-------------|-----------|---------|
| ITI-38 Cross Gateway Query | Initiating Gateway → Responding Gateway | Federated registry query |
| ITI-39 Cross Gateway Retrieve | Initiating Gateway → Responding Gateway | Federated document retrieval |

Required grouping: ATNA + CT. Pair with XCPD (Initiating Gateway grouped with Initiating Gateway from XCPD) for patient discovery.

---

## XCPD Cross-Community Patient Discovery

### Actors

| Actor | Role |
|-------|------|
| Initiating Gateway | Sends cross-community patient discovery request |
| Responding Gateway | Responds with matches in its community |

### Transactions

| Transaction | From → To | Purpose |
|-------------|-----------|---------|
| ITI-55 Cross Gateway Patient Discovery | Initiating Gateway → Responding Gateway | Cross-community patient discovery |
| ITI-56 Patient Location Query | Initiating Gateway → Responding Gateway | Locate communities holding records |

---

## MHD Mobile access to Health Documents

### Actors

| Actor | Role |
|-------|------|
| Document Source (MHD) | Submits documents to a Document Recipient via FHIR |
| Document Recipient (MHD) | Receives FHIR document submissions |
| Document Responder (MHD) | Serves FHIR-based queries/retrievals |
| Document Consumer (MHD) | Queries and retrieves via FHIR |

### Transactions

| Transaction | From → To | Purpose |
|-------------|-----------|---------|
| ITI-65 Provide Document Bundle | Document Source → Document Recipient | POST a FHIR Bundle containing DocumentReference, DocumentManifest (or List), Binary |
| ITI-66 Find Document Manifests / Lists | Document Consumer → Document Responder | FHIR search for DocumentManifest or List |
| ITI-67 Find Document References | Document Consumer → Document Responder | FHIR search for DocumentReference |
| ITI-68 Retrieve Document | Document Consumer → Document Responder | GET Binary / GET Attachment |

Required grouping: ATNA + CT; IUA for auth.

---

## PIX (HL7 v2) / PIXm (FHIR)

### PIX (V2) Actors

| Actor | Role |
|-------|------|
| Patient Identity Source | Feeds patient identifiers (HL7 v2 ADT) |
| PIX Manager | Maintains cross-reference table |
| PIX Consumer | Queries PIX Manager for matches |

### PIX (V2) Transactions

| Transaction | From → To | Purpose |
|-------------|-----------|---------|
| ITI-8 Patient Identity Feed | Patient Identity Source → PIX Manager | HL7 v2 ADT^A04/A08/A40 |
| ITI-9 PIX Query | PIX Consumer → PIX Manager | HL7 v2 QBP/RSP |
| ITI-10 PIX Update Notification | PIX Manager → PIX Consumer | HL7 v2 ADT^A31/A40 notifications |

### PIXm (FHIR) Actors

| Actor | Role |
|-------|------|
| Patient Identifier Cross-reference Manager (PIXm) | Maintains FHIR-based cross-reference |
| Patient Identifier Cross-reference Consumer (PIXm) | Queries via FHIR |
| Patient Identity Source (PIXm) | Feeds patient identifiers via FHIR |

### PIXm Transactions

| Transaction | From → To | Purpose |
|-------------|-----------|---------|
| ITI-83 Mobile Patient Identifier Cross-reference Query | PIXm Consumer → PIXm Manager | `$ihe-pix` operation on Patient |
| ITI-93 Mobile Patient Identity Feed | Patient Identity Source → PIXm Manager | FHIR Patient updates / Bundle |
| ITI-104 Patient Identity Feed FHIR | Patient Identity Source → PIXm Manager | Alternative FHIR-based feed |

---

## PDQ (HL7 v2) / PDQm (FHIR)

### PDQ (V2) Actors

| Actor | Role |
|-------|------|
| Patient Demographics Supplier | Holds the demographic database |
| Patient Demographics Consumer | Queries by demographics |

### PDQ (V2) Transactions

| Transaction | From → To | Purpose |
|-------------|-----------|---------|
| ITI-21 Patient Demographics Query | Consumer → Supplier | HL7 v2 QBP^Q22 |
| ITI-22 Patient Demographics and Visit Query | Consumer → Supplier | HL7 v2 QBP^ZV1 (extends with visit info) |

### PDQm (FHIR) Actors

| Actor | Role |
|-------|------|
| Patient Demographics Supplier (PDQm) | FHIR-based supplier |
| Patient Demographics Consumer (PDQm) | FHIR-based consumer |

### PDQm Transactions

| Transaction | From → To | Purpose |
|-------------|-----------|---------|
| ITI-78 Mobile Patient Demographics Query | Consumer → Supplier | FHIR Patient search |

---

## ATNA Audit Trail and Node Authentication

### Actors

| Actor | Role |
|-------|------|
| Secure Node | A full-blown system component with audit + mTLS support |
| Secure Application | A lightweight variant for applications that don't manage their own network connections |
| Audit Record Repository | Receives audit messages from Secure Nodes/Applications |

### Transactions

| Transaction | From → To | Purpose |
|-------------|-----------|---------|
| ITI-19 Authenticate Node | Secure Node ↔ Secure Node | mTLS during association |
| ITI-20 Record Audit Event | Secure Node → Audit Record Repository | Send audit message (syslog over TLS / RFC 5424) |

ATNA audit messages use the DICOM/IHE audit schema and transport over TLS-protected syslog.

---

## BPPC Basic Patient Privacy Consents

### Actors

| Actor | Role |
|-------|------|
| Content Creator | Authors BPPC consent documents |
| Content Consumer | Reads BPPC consents to enforce policy |

### Transactions

BPPC re-uses XDS.b transactions to publish consent documents. No unique transaction numbers.

---

## IUA Internet User Authorization

### Actors

| Actor | Role |
|-------|------|
| Authorization Client | Requests tokens |
| Authorization Server | Issues tokens |
| Resource Server | Validates tokens and serves protected resources |

### Transactions

| Transaction | From → To | Purpose |
|-------------|-----------|---------|
| ITI-71 Get Access Token | Authorization Client → Authorization Server | OAuth 2.0 / OIDC token request |
| ITI-72 Incorporate Access Token | Authorization Client → Resource Server | Bearer token usage |
| ITI-102 Token Introspection | Resource Server → Authorization Server | OAuth introspection (where supported) |
| ITI-103 Token Revocation | Authorization Client → Authorization Server | Revoke token |

---

## SWF.b Scheduled Workflow (Radiology)

### Actors (selected)

| Actor | Role |
|-------|------|
| ADT Patient Registration | Hospital registration system |
| Order Placer | EHR / CPOE placing the imaging order |
| Order Filler / DSS | RIS — owns the procedure and the schedule |
| Acquisition Modality | The scanner |
| Image Manager / Image Archive | PACS |
| Image Display | Workstation / viewer |

### Transactions (selected)

| Transaction | From → To | Purpose |
|-------------|-----------|---------|
| RAD-1 Patient Registration | Registration → ADT (HL7 v2 ADT) | Register patient |
| RAD-2 Placer Order Management | Order Placer → Order Filler (HL7 v2 ORM/OMI) | Place imaging order |
| RAD-3 Filler Order Management | Order Filler → Order Placer | Fill / update order |
| RAD-4 Procedure Scheduled | Order Filler → many | Communicate scheduled procedure |
| RAD-5 Modality Worklist Provided | Order Filler → Modality (DICOM C-FIND MWL) | Provide MWL |
| RAD-6 Modality Procedure Step In Progress | Modality → many (DICOM MPPS N-CREATE) | MPPS start |
| RAD-7 Modality Procedure Step Completed | Modality → many (DICOM MPPS N-SET) | MPPS done |
| RAD-8 Modality Images Stored | Modality → PACS (DICOM C-STORE) | Store images |
| RAD-13 Image Availability Query | Display → PACS | Confirm images available |
| RAD-14 Query Images | Display → PACS (DICOM C-FIND) | Find images |
| RAD-16 Retrieve Images | Display → PACS (DICOM C-MOVE/C-GET) | Retrieve images |

---

## XDS-I.b Cross-Enterprise Document Sharing for Imaging

### Actors (in addition to XDS.b)

| Actor | Role |
|-------|------|
| Imaging Document Source | Registers KOS manifest + provides image access |
| Imaging Document Consumer | Queries Registry, retrieves KOS, then retrieves images |

### Transactions (in addition to XDS.b)

| Transaction | From → To | Purpose |
|-------------|-----------|---------|
| RAD-54 Provide and Register Imaging Document Set | Imaging Document Source → Document Repository | Submit KOS manifest + image references |
| RAD-55 WADO Retrieve | Consumer → Imaging Document Source | Retrieve images by WADO-URI |
| RAD-69 Retrieve Imaging Document Set | Consumer → Imaging Document Source | Retrieve referenced study (DICOM transfer) |
| RAD-75 Cross Gateway Retrieve Imaging Document Set | Initiating Gateway → Responding Gateway | XCA-I variant |

KOS = Key Object Selection (DICOM SOP Class for manifests of selected images).

---

## Profile Grouping Notes

- Almost every ITI actor MUST be grouped with ATNA (Secure Node or Secure Application) and CT (Time Client).
- IUA Authorization Client / Resource Server typically grouped with FHIR-based actors (MHD, mCSD, mACM).
- BPPC actors are grouped with XDS Document Source / Consumer to capture and enforce consent.
- XUA (SAML) is layered onto cross-community profiles when a user assertion must be conveyed end-to-end.
- For TEFCA, expect XCA + XCPD + XUA grouping today; FHIR Roadmap moves toward MHD / PIXm / PDQm + IUA.

Verify all transaction numbers, actor names, and required-grouping clauses against the current ITI / RAD / PCC Technical Framework volumes before committing to a design.
