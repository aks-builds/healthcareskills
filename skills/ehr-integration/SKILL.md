---
name: ehr-integration
description: When the user wants to integrate with a specific EHR vendor's APIs, app program, or sandbox. Also use when the user mentions "Epic," "App Orchard," "Connection Hub," "Cerner," "Oracle Health," "Millennium," "CareAware," "Code Console," "Ignite APIs," "HealtheLife," "Athenahealth," "athenaOne," "MDP," "More Disruption Please," "Meditech," "Greenfield," "eClinicalWorks," "NextGen," "Allscripts," "Veradigm," "OpenEMR," "EHR sandbox," "vendor app review," "Care Everywhere," "Hyperdrive," "Chronicles," "Interconnect," "DocumentReference write-back," or "MDM message." For FHIR-resource-level work, see fhir-integration. For SMART OAuth specifics, see smart-on-fhir. For HL7 v2 messaging, see hl7-v2.
metadata:
  version: 1.0.0
---

# EHR Integration

You are an expert in commercial EHR integration. Your goal is to help engineers ship integrations against specific EHR vendors — navigating each vendor's app program, sandbox, OAuth flows, data scopes, and common write-back patterns — without getting blocked by vendor-specific quirks.

## Initial Assessment

Read `.agents/healthcare-context.md` first (fall back to `.claude/healthcare-context.md`). The context file tells you which EHRs the user's organization (or customer base) is targeting, which versions, and whether you are building inside a covered entity or as a third-party app vendor seeking distribution.

If the context file does not exist, ask a minimal set of questions:

1. Which EHR(s)? (Epic, Oracle Health/Cerner, Athenahealth, Meditech, eClinicalWorks, NextGen, Allscripts/Veradigm, OpenEMR, other)
2. Which deployment(s)? (inpatient, ambulatory, ED, oncology, etc.)
3. Are you a third-party app developer or an internal team at a customer site?
4. Read-only or read/write? Which workflows?
5. Is this for one customer or for distribution across many?

---

## Vendor Decision Matrix

| Vendor | App program | Production OAuth | Primary FHIR profile | Notable write-back |
|--------|-------------|------------------|----------------------|--------------------|
| Epic | Connection Hub (formerly App Orchard, then Showroom) | SMART on FHIR, OAuth 2.0 | US Core on R4, USCDI v1+ | DocumentReference, Flowsheet, Communication |
| Oracle Health (Cerner) | Code Console + Ignite APIs for Millennium | SMART on FHIR, OAuth 2.0 | US Core on R4 | DocumentReference, ServiceRequest |
| Athenahealth | More Disruption Please (MDP) | OAuth 2.0; athenaOne REST + FHIR | US Core on R4 | Document upload via athenaOne REST |
| Meditech | Greenfield + Traverse Exchange | SMART on FHIR, OAuth 2.0 | US Core on R4 (Expanse) | DocumentReference (varies by site) |
| eClinicalWorks | Developer Program / EVA APIs | SMART on FHIR for USCDI | US Core on R4 | Document upload, ServiceRequest |
| NextGen | Connect Marketplace | SMART on FHIR | US Core on R4 | DocumentReference |
| Allscripts / Veradigm | Veradigm Developer Program (UAN, Sunrise, TouchWorks, Pro EHR) | SMART on FHIR; legacy SOAP UAN | US Core on R4 (newer) | Note write-back via UAN or FHIR DocumentReference |
| OpenEMR | Self-hosted; open source | SMART on FHIR module | US Core on R4 | DocumentReference |

Always confirm the current state with the vendor's developer portal — programs get renamed and re-tiered often.

---

## Epic

### App program

- **Connection Hub** is the current name of the program (was App Orchard, then Showroom). Tiered membership levels gate access to APIs, sandbox, and customer distribution.
- Most apps go through **vendor services review** before being installable at a customer site. Each customer site still has its own go-live and security review.
- USCDI on FHIR R4 APIs are available **without** Connection Hub membership for the patient-facing read scopes mandated by ONC, but write APIs and most provider-facing scopes require membership.

### Sandbox

- **fhir.epic.com** has open documentation and a developer sandbox (Open Epic / "Open" endpoints).
- Each Epic customer (organization) has its own production endpoint; you cannot reuse client IDs across customers.

### OAuth flow notes

- Epic supports SMART App Launch (EHR launch and standalone launch).
- Confidential clients use JWT client assertion (per SMART Backend Services for system-to-system) or client secret. Public clients use PKCE.
- Scopes follow SMART v1 and v2 syntax. Epic supports `patient/*.read`, `user/*.read`, `system/*.read`, and selected write scopes.
- The token endpoint returns a context object with `patient`, `encounter`, and other launch parameters.

### Hyperdrive, Interconnect, Chronicles

- **Hyperdrive** is the Chromium-based replacement for the Hyperspace thick client. Embedded web apps in Hyperdrive use the same SMART launch flow as Hyperspace.
- **Interconnect** is Epic's integration server — APIs (FHIR, web services) are exposed through Interconnect at each customer site.
- **Chronicles** is the underlying database. You do not query Chronicles directly; you go through Interconnect (FHIR / web service / Kuiper for analytics).

### Care Everywhere

- Epic's exchange network. Now connected to TEFCA via Epic Nexus QHIN. Useful for retrieving outside records, not for app distribution.

### Common write-backs

- **DocumentReference** (FHIR) for note write-back from scribe / consult apps. Requires write scopes and customer approval.
- **Communication** for messages back into In Basket.
- **Flowsheet rows** via FHIR `Observation` (with category `vital-signs` or custom) — but flowsheet ID mapping is per-customer.
- **Order Composer / OrderEntry** integration uses Hyperdrive web SDK or FHIR `ServiceRequest`/`MedicationRequest` where supported.

---

## Oracle Health (Cerner Millennium)

### App program

- **Code Console** is the developer portal for Cerner / Oracle Health.
- **Ignite APIs for Millennium** is the umbrella for the FHIR R4 APIs (read + write where supported), Open Developer APIs, and Provider APIs.
- **CareAware** is the device-connectivity platform; CareAware iBus handles medical device data ingestion.
- Apps go through Cerner's app review for customer distribution; each tenant still has its own setup.

### Sandbox

- The Cerner FHIR R4 sandbox is publicly documented at the Code Console developer portal.
- Tenants are identified by tenant ID in the FHIR base URL.

### OAuth flow notes

- SMART App Launch (EHR + standalone). Confidential and public client support.
- Cerner historically supported a system-level OAuth flow via JWT assertion for SMART Backend Services.

### HealtheLife

- Cerner's patient portal. Patient-facing apps can launch from HealtheLife using SMART standalone with patient login.

### Common write-backs

- DocumentReference write for note write-back (requires `user/DocumentReference.write` or system scope).
- ServiceRequest, MedicationRequest where supported per the published Ignite catalog.

---

## Athenahealth

### App program

- **More Disruption Please (MDP)** is Athenahealth's partner program. Production access requires MDP enrollment.
- Production keys are issued per partner-customer pairing.

### APIs

- **athenaOne REST APIs** are the proprietary REST APIs (older, broad coverage including scheduling, billing, clinical).
- **FHIR R4 APIs** cover USCDI and a growing set of write operations.
- Document upload is typically done via the athenaOne REST `/clinicaldocument` endpoints rather than FHIR `DocumentReference`.

### OAuth flow notes

- OAuth 2.0 (client credentials for system-to-system; authorization code with PKCE for user-facing apps).
- Practice ID is required in every call — context is tenant-aware.

---

## Meditech

### App program

- **Greenfield** is Meditech's developer program for Expanse (the modern web-based EHR).
- **Traverse Exchange** is Meditech's interop / HIE platform; not the same as Greenfield.
- Most Greenfield apps target the FHIR R4 APIs against Expanse.

### Quirks

- Older Magic / Client-Server (C/S) Meditech installs do not have a modern FHIR API. You may end up using HL7 v2 over MLLP for those sites.
- Confirm the customer is on Expanse before assuming FHIR access.

---

## Other vendors (quick reference)

| Vendor | Notes |
|--------|-------|
| eClinicalWorks | Developer Program offers SMART on FHIR + EVA APIs. Sandbox access usually requires partner enrollment. |
| NextGen | Connect Marketplace; SMART on FHIR; legacy MIRTH-based integrations are common at smaller practices. |
| Allscripts / Veradigm | Multiple product lines (Sunrise inpatient, TouchWorks, Pro EHR, Practice Fusion). Older UAN (Unified Action Network) SOAP APIs still in heavy use alongside newer FHIR endpoints. |
| OpenEMR | Open source; SMART on FHIR module is enabled per-install. You own the sandbox and the prod instance. |

---

## Cross-Vendor Integration Workflows

### Read clinical data

1. Establish OAuth (SMART App Launch or Backend Services).
2. Resolve patient context (`launch/patient` or search by demographics + identifier).
3. Fetch the US Core profiles you need: `Patient`, `Condition`, `Observation` (labs + vitals), `MedicationRequest`, `AllergyIntolerance`, `Procedure`, `Immunization`, `DocumentReference`, `Encounter`.
4. Handle paging (`Bundle.link.next`) — do not assume the first page is complete.
5. Cache by `Resource.meta.versionId` + `lastUpdated` to detect changes.

### Bulk export

- All major vendors support **Bulk FHIR** (`$export`) for USCDI per ONC rules. Group-level export is common; system-level may not be.
- Bulk export is async — kick off the job, poll the status endpoint, then download NDJSON files.
- Plan for files in the 1-100 GB range for large groups; stream the NDJSON, do not load into memory.

### Appointments

- FHIR `Appointment` + `Slot` is increasingly supported but vendor-by-vendor. Athenahealth and Epic have rich scheduling APIs; some vendors require their proprietary REST APIs.

### Document insertion (note write-back)

The most common write-back pattern across vendors:

```http
POST /FHIR/R4/DocumentReference
Authorization: Bearer <token>
Content-Type: application/fhir+json
{
  "resourceType": "DocumentReference",
  "status": "current",
  "type": {
    "coding": [{
      "system": "http://loinc.org",
      "code": "11506-3",
      "display": "Progress note"
    }]
  },
  "subject": { "reference": "Patient/example-1" },
  "context": { "encounter": [{ "reference": "Encounter/example-enc-1" }] },
  "content": [{
    "attachment": {
      "contentType": "text/plain",
      "data": "<base64-encoded-note-body>"
    }
  }]
}
```

If the vendor only supports HL7 v2, use an **MDM^T02** (original document) or **MDM^T08** (document edit) message instead. Confirm which document codes (LOINC) the customer's site accepts — verify against the live LOINC release.

---

## Sandbox to Production Checklist

- [ ] Sandbox client ID issued, OAuth flow works end-to-end
- [ ] All read scopes exercised against synthetic patients
- [ ] All write scopes exercised; confirm rollback behavior
- [ ] App review submitted with required artifacts (privacy policy, support model, security questionnaire)
- [ ] Each customer site assigns its own client ID — registration process documented
- [ ] HIPAA BAA in place with each customer (you are a Business Associate for them)
- [ ] Audit logging of every API call (AuditEvent) retained per HIPAA retention rules
- [ ] Token storage encrypted; refresh strategy defined
- [ ] Error handling for 401 (expired token), 403 (scope), 429 (rate limit), 5xx
- [ ] Backoff + retry strategy for transient errors

---

## Common Pitfalls

- Treating "the EHR's FHIR API" as one thing — each customer site has its own endpoint, version, and enabled scopes.
- Assuming write scopes work in sandbox = work in production. They often require per-customer approval.
- Ignoring vendor-specific extensions (Epic and Cerner both publish extension catalogs).
- Hardcoding flowsheet IDs, order IDs, or document type codes — these vary per customer.
- Forgetting that **Hyperdrive embedded apps still launch via SMART** — there is no "Hyperdrive-only" API.
- Confusing **Care Everywhere** (Epic's exchange network) with the Epic FHIR API. Care Everywhere is for outside-records retrieval, not app development.

---

## Task-Specific Questions

1. Which vendor(s), which products, and which customer sites?
2. Which user-facing surface (Hyperdrive embed, web app, mobile, server-to-server)?
3. Which FHIR resources or proprietary endpoints?
4. Read-only or read/write? If write, which workflows and clinical signoff path?
5. Sandbox status — do you already have credentials?
6. Distribution model — internal use, one customer, or many?

---

## Related Skills

- **healthcare-context**: Tells you which EHR(s) and modules to target before you start.
- **smart-on-fhir**: SMART App Launch and Backend Services OAuth flows used by every modern EHR.
- **fhir-integration**: Resource-level FHIR R4 / US Core profile details.
- **hl7-v2**: For sites stuck on legacy v2 messaging (ADT, ORM, MDM, ORU).
- **hipaa-compliance**: BAA, access control, audit requirements when integrating with a covered entity.
- **terminology-services**: Mapping vendor-specific codes (Epic flowsheet IDs, Cerner CKIs) to standard terminologies.
- **audit-logging**: AuditEvent capture for every API call, required for HIPAA accounting of disclosures.
