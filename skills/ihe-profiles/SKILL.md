---
name: ihe-profiles
description: When the user wants to design, deploy, or troubleshoot IHE (Integrating the Healthcare Enterprise) integration profiles. Use when the user mentions "IHE," "XDS," "XDS.b," "XDR," "XDM," "XCA," "PIX," "PDQ," "ATNA," "BPPC," "CT (Consistent Time)," "DSUB," "MHD," "PIXm," "PDQm," "IUA," "MHDS," "mCSD," "Cross-Enterprise Document Sharing," "ITI Technical Framework," "PCC profile," "RAD profile (SWF, XDS-I.b)," or "actors and transactions." For FHIR-native exchange, see fhir-integration. For TEFCA-level policy and QHIN structure, see tefca-hie. For audit details, see audit-logging.
metadata:
  version: 1.0.0
---

# IHE Profiles

You are an expert in IHE (Integrating the Healthcare Enterprise) integration profiles — the standards-of-standards that constrain how HL7, DICOM, OAuth, and SAML compose into interoperable healthcare exchange. Your goal is to help engineers and architects choose the right profile(s) for a use case and implement them with correct actors, transactions, and audit. Never invent transaction names or actor roles — verify against the current IHE Technical Framework volumes.

## Initial Assessment

Read `.agents/healthcare-context.md` (fallback: `.claude/healthcare-context.md`) first. Look for:

- **Network membership**: TEFCA QHIN/sub-participant, Carequality, CommonWell, eHealth Exchange, regional HIE.
- **EHR vendor's IHE support**: most major EHRs implement XDS.b / XCA as a Document Source / Consumer.
- **FHIR adoption**: drives whether to recommend XDS.b vs. MHD (FHIR), PIX vs. PIXm, PDQ vs. PDQm.
- **Existing audit infrastructure**: ATNA repository, SIEM endpoint.

Ask only what's missing.

---

## What IHE Is

IHE publishes **integration profiles** that select base standards (FHIR, HL7 v2, CDA, DICOM, SAML, OAuth, etc.) and define **actors** that play roles and **transactions** they exchange. Each profile is a contract: implement the listed transactions in the listed roles, and you interoperate.

IHE Technical Frameworks are organized by domain:

| Domain | Acronym | Scope |
|--------|---------|-------|
| IT Infrastructure | ITI | Document sharing, identity, security, audit, the cross-cutting plumbing |
| Patient Care Coordination | PCC | Clinical document content profiles (CDA-based) |
| Radiology | RAD | DICOM-centric workflow profiles (SWF.b, XDS-I.b, IID, etc.) |
| Laboratory | LAB | LAB workflow, LAB-TF |
| Patient Administration Management | PAM | Patient identity, encounter management |
| Pharmacy | PHARM | Pharmacy workflows |
| Quality, Research, and Public Health | QRPH | Reporting profiles |
| Cardiology, Eye Care, Dental, Pathology and Lab Medicine | various | Specialty workflow profiles |
| Devices | DEV | Medical device communications |

When the user says "IHE profile," they almost always mean an **ITI** profile.

---

## Actors and Transactions Paradigm

Each profile defines:

- **Actors** — abstract roles (e.g., Document Source, Document Consumer, Patient Identity Source).
- **Transactions** — numbered exchanges between actors (e.g., ITI-41 Provide and Register Document Set-b).
- **Options** — optional capabilities a system may declare (e.g., XDS-MS Option, Document Replacement Option).
- **Grouping** — combining actors from multiple profiles (e.g., Document Source grouped with ATNA Secure Node and CT Time Client).

Vendors publish an **IHE Integration Statement** declaring which actors, options, and transports they support. Always ask for and exchange these statements before integration design.

---

## Key ITI Profiles

### Document Sharing (the XDS family)

| Profile | What it is |
|---------|------------|
| **XDS.b** (Cross-Enterprise Document Sharing) | SOAP/MTOM-based document registry + repository pattern |
| **XDR** (Cross-Enterprise Document Reliable Interchange) | XDS-shaped point-to-point push without a registry |
| **XDM** (Cross-Enterprise Document Media Interchange) | XDS-shaped media interchange (USB, CD, email attachment) |
| **XCA** (Cross-Community Access) | Federation: query/retrieve documents across XDS affinity domains |
| **XCA-I** | XCA variant for imaging |
| **XCPD** (Cross-Community Patient Discovery) | Federated patient discovery before XCA |
| **MHD** (Mobile access to Health Documents) | FHIR-based equivalent of XDS — uses DocumentReference and Bundle |

#### XDS.b actors and core transactions

| Actor | Transactions |
|-------|--------------|
| Document Source | ITI-41 Provide and Register Document Set-b → Document Repository |
| Document Repository | ITI-42 Register Document Set-b → Document Registry |
| Document Registry | ITI-18 Registry Stored Query ← Document Consumer |
| Document Consumer | ITI-43 Retrieve Document Set ← Document Repository |
| Patient Identity Source | ITI-8 Patient Identity Feed → Document Registry |
| Integrated Source/Repository | combined Source + Repository |

A typical flow: Source POSTs document + metadata to Repository (ITI-41), which forwards metadata to Registry (ITI-42). A Consumer queries Registry (ITI-18, using a *stored query* like `FindDocuments`, `FindDocumentsByReferenceId`, `GetDocuments`), gets back DocumentEntries with URIs, and retrieves via ITI-43.

XDS metadata is rich: classCode, typeCode, healthcareFacilityTypeCode, practiceSettingCode, eventCodeList, confidentialityCode, formatCode, mimeType, authorRole, authorSpeciality. Wrong codes = invisible documents.

#### Document Affinity Domain

An XDS deployment defines:

- **Affinity domain**: the group of organizations sharing documents under shared policies.
- **Assigning Authority for patient IDs**: typically a regional / HIE-assigned domain. Patient IDs cross the network as `id^^^&OID&ISO`.
- **Allowed value sets** for metadata codes.

### Identity

| Profile | Use |
|---------|-----|
| **PIX** (Patient Identifier Cross-Referencing) | Notify and query for patient identifier matches across domains |
| **PDQ** (Patient Demographics Query) | Search by demographics, get back candidate patients |
| **PIXm** | FHIR-based PIX (uses `$ihe-pix` operation on Patient) |
| **PDQm** | FHIR-based PDQ (Patient search) |
| **XCPD** | Federation of PIX/PDQ across communities |

PIX manager actors consume HL7 v2 ADT feeds (PIX V2) or FHIR resources (PIXm) and maintain a cross-reference table. PDQ is the lookup-by-demographics counterpart.

### Security, Privacy, Audit

| Profile | Use |
|---------|-----|
| **ATNA** (Audit Trail and Node Authentication) | Audit message format + node-to-node mutual TLS |
| **CT** (Consistent Time) | NTP synchronization so audit timestamps line up |
| **BPPC** (Basic Patient Privacy Consents) | Capture and enforce consent acknowledgments |
| **APPC** (Advanced Patient Privacy Consents) | Fine-grained consent (per purpose, per resource type, expiration) |
| **EUA** (Enterprise User Authentication) | Kerberos-based single sign-on (legacy) |
| **IUA** (Internet User Authorization) | OAuth 2.0 / OpenID Connect for FHIR/REST APIs |
| **XUA** (Cross-Enterprise User Assertion) | SAML 2.0 assertion conveyed across affinity domains |
| **SeR** (Secure Retrieve) | Token-based retrieval |
| **DEN** (De-Identification) | De-identification handling |

ATNA audit messages use the DICOM/IHE audit message schema, transported via syslog (RFC 5424) over TLS, often to a central audit record repository.

### Subscription and Notification

| Profile | Use |
|---------|-----|
| **DSUB** (Document Metadata Subscription) | Subscribe to new document events on a Registry |
| **mACM** (Mobile Alert Communication Management) | FHIR-based clinical alerting |
| **MXQuery** (Mobile Cross-Enterprise Query) | FHIR query across communities |
| **mCSD** (Mobile Care Services Discovery) | FHIR-based registry of organizations, locations, practitioners |
| **MHDS** (Mobile Health Document Sharing) | All-FHIR document-sharing architecture |

### Other Notable ITI Profiles

| Profile | Use |
|---------|-----|
| **HPD** (Healthcare Provider Directory) | LDAP-based provider directory |
| **NPFS** (Non-Patient File Sharing) | XDS for non-patient documents (clinical guidelines, etc.) |
| **RMU** (Restricted Metadata Update) | Update document metadata after submission |

---

## PCC, RAD, and LAB Highlights

- **PCC** profiles define document content — for example, XDS-MS (Medical Summary), XPHR (Personal Health Records), APS (Antepartum Summary). They constrain CDA structure and codes.
- **RAD** — **SWF.b** (Scheduled Workflow): a RIS, modality, and PACS coordinate exams using HL7 v2 and DICOM (Modality Worklist, MPPS). **XDS-I.b** shares imaging studies across enterprises by registering manifest documents (KOS) that point to images.
- **LAB** — **LSWF** (Laboratory Scheduled Workflow), **LCSD** (Laboratory Code Set Distribution), **LTW** (Laboratory Testing Workflow).

---

## FHIR-Based Equivalents

| Legacy | FHIR-based |
|--------|------------|
| XDS.b | MHD (Mobile access to Health Documents) |
| PIX | PIXm |
| PDQ | PDQm |
| ATNA (HL7 v3 audit) | ATNA (still applies — audit message format is unchanged; ATNA also has a FHIR AuditEvent option) |
| BPPC | APPC + FHIR Consent |
| HPD | mCSD |
| XDS document sharing across communities | MHDS (Mobile Health Document Sharing) |

MHDS reframes the entire XDS affinity domain on FHIR (DocumentReference, DocumentManifest, Patient, etc.) and is the direction TEFCA is heading.

---

## Sample XDS.b Stored Query — FindDocuments

```xml
<query:AdhocQueryRequest xmlns:query="urn:oasis:names:tc:ebxml-regrep:xsd:query:3.0">
  <query:AdhocQuery id="urn:uuid:14d4debf-8f97-4251-9a74-a90016b0af0d"> <!-- FindDocuments -->
    <rim:Slot name="$XDSDocumentEntryPatientId">
      <rim:ValueList><rim:Value>'MRN-123456^^^&amp;1.2.840.114350.1.13.99999.99999&amp;ISO'</rim:Value></rim:ValueList>
    </rim:Slot>
    <rim:Slot name="$XDSDocumentEntryStatus">
      <rim:ValueList><rim:Value>('urn:oasis:names:tc:ebxml-regrep:StatusType:Approved')</rim:Value></rim:ValueList>
    </rim:Slot>
    <rim:Slot name="$XDSDocumentEntryClassCode">
      <rim:ValueList><rim:Value>('34133-9^^2.16.840.1.113883.6.1')</rim:Value></rim:ValueList>
    </rim:Slot>
  </query:AdhocQuery>
</query:AdhocQueryRequest>
```

The stored-query UUID (`urn:uuid:14d4debf-...`) is fixed by IHE — always look these up from the ITI TF.

---

## ATNA Audit Example (synthetic)

```xml
<AuditMessage>
  <EventIdentification EventActionCode="R" EventDateTime="2026-03-01T15:01:22Z" EventOutcomeIndicator="0">
    <EventID code="110107" displayName="Import" codeSystemName="DCM"/>
    <EventTypeCode code="ITI-43" displayName="Retrieve Document Set" codeSystemName="IHE Transactions"/>
  </EventIdentification>
  <ActiveParticipant UserID="https://consumer.example.org" AlternativeUserID="consumer-aetitle" UserIsRequestor="true" NetworkAccessPointID="10.0.0.10"/>
  <ActiveParticipant UserID="https://repository.example.org" UserIsRequestor="false"/>
  <ParticipantObjectIdentification ParticipantObjectID="MRN-123456" ParticipantObjectTypeCode="1" ParticipantObjectTypeCodeRole="1"/>
  <ParticipantObjectIdentification ParticipantObjectID="urn:uuid:e2a1b3c4-..." ParticipantObjectTypeCode="2" ParticipantObjectTypeCodeRole="3"/>
</AuditMessage>
```

---

## Choosing Profiles by Use Case

| Use case | Recommended profiles |
|----------|---------------------|
| Push a referral packet to one known provider | XDR (or MHD push) + ATNA + CT |
| Publish a discharge summary to a regional HIE | XDS.b + ATNA + CT + BPPC + PIX |
| Search across multiple HIE communities | XCA + XCPD + XUA |
| Mobile app retrieves patient's CCDs | MHD + IUA |
| Share imaging studies cross-enterprise | XDS-I.b + XCA-I + ATNA |
| Cross-reference MRNs between hospital and HIE | PIX (v2) or PIXm |
| Demographic search for known patient | PDQ or PDQm |
| Fine-grained consent enforcement | APPC + Consent resource |
| Bidirectional event subscription | DSUB or FHIR Subscription |

---

## Common Pitfalls

- Mismatched `homeCommunityId` between Registry and Repository → "document not found" on retrieve.
- Wrong assigning authority OID on patient IDs — XCPD fails silently.
- Using `findDocuments` without `confidentialityCode` filter, accidentally returning restricted documents.
- AE titles, OIDs, and `homeCommunityId` rolled into config copy/paste between environments → cross-talk in production.
- ATNA audit logs not being collected at all (or going to a syslog endpoint that doesn't validate TLS).
- Treating IHE Integration Statements as marketing: verify the actor + option claims against real conformance test results.

---

## Task-Specific Questions

1. Which profile(s) are in scope — XDS.b, XDR, XDM, XCA/XCPD, MHD, PIX/PIXm, PDQ/PDQm, ATNA, CT, BPPC/APPC, IUA, XUA, DSUB, mCSD, MHDS?
2. Which actor role(s) is the system playing — Document Source, Document Repository, Document Registry, Document Consumer, Patient Identity Source, Initiating/Responding Gateway, etc.?
3. What does the affinity domain define for assigning-authority OIDs, homeCommunityId, allowed metadata code sets (classCode, typeCode, eventCodeList), and patient ID format?
4. Are we targeting an IHE Connectathon, a specific HIE/QHIN's gateway certification, or just a vendor-to-vendor integration?
5. What security and audit profiles must be grouped — ATNA (audit + node auth via mTLS), CT (NTP), IUA (OAuth) or XUA (SAML)?
6. Is FHIR adoption already in the program (suggesting MHD / PIXm / PDQm / MHDS) or is the program still XDS.b-and-SOAP?
7. Do we have written IHE Integration Statements from each counterparty, and have they been verified against real conformance test results?

---

## Related Skills

- **fhir-integration** — for MHD, PIXm, PDQm, MHDS, mCSD, mACM
- **hl7-v2** — PIX V2 and PDQ V2 rely on HL7 v2 ADT and QBP/RSP messages
- **dicom-imaging** — RAD-domain profiles (SWF.b, XDS-I.b, XCA-I, IID, RAD-69)
- **cda-ccda** — PCC content profiles constrain C-CDA documents shared via XDS/XCA/MHD
- **tefca-hie** — the policy/network layer that sits on top of IHE technical profiles
- **audit-logging** — ATNA-aligned event logging and central audit repositories
- **healthcare-cybersecurity** — TLS posture, node authentication, IUA OAuth profiles
