# EHR Vendor Comparison Matrix

A side-by-side reference for the five US EHR vendors third-party integrators most commonly target. Always confirm against the vendor's current developer portal — programs are renamed and re-tiered often.

## Capability Matrix

| Capability | Epic | Oracle Health (Cerner) | Athenahealth | Meditech | eClinicalWorks |
|------------|------|------------------------|--------------|----------|----------------|
| **OAuth profile** | SMART on FHIR (v1 + v2 scopes); JWT client assertion for confidential / Backend Services | SMART on FHIR (v1 + v2); JWT client assertion for Backend Services | OAuth 2.0; authorization code with PKCE for user-facing; client_credentials for system-to-system | SMART on FHIR on Expanse | SMART on FHIR (USCDI scope) |
| **FHIR R4 (US Core)** | Yes; production endpoint per customer | Yes; tenant-aware base URL | Yes; USCDI + growing write coverage | Yes on Expanse; older Magic / C-S installs have no modern FHIR | Yes for USCDI |
| **FHIR bulk export ($export)** | Group-level supported; system-level varies by customer | Group-level supported; system-level varies | Available; coverage growing | Available on Expanse | Available |
| **Sandbox process** | Open Epic / Open endpoints at fhir.epic.com (no membership required); production needs Connection Hub | Code Console developer portal; public R4 sandbox | More Disruption Please (MDP) program; sandbox issued per partner | Greenfield program; sandbox per partner | Developer Program enrollment; sandbox usually behind partner agreement |
| **App program / distribution** | Connection Hub (was App Orchard, then Showroom); vendor services review; per-customer install + security review | Code Console + Ignite APIs for Millennium; app review for distribution; each tenant has its own setup | More Disruption Please (MDP); production keys per partner-customer pairing | Greenfield for Expanse apps; Traverse Exchange is interop, not Greenfield | Developer Program; partner enrollment for sandbox + production |
| **Note write-back** | DocumentReference (FHIR) for scribe / consult apps; Communication for In Basket; HL7 v2 MDM at older sites | DocumentReference write (user/DocumentReference.write or system scope) | athenaOne REST /clinicaldocument endpoints (not FHIR DocumentReference) | DocumentReference on Expanse (varies by site) | Document upload via FHIR DocumentReference where supported |
| **Order write-back** | OrderEntry via Hyperdrive web SDK or FHIR ServiceRequest / MedicationRequest where supported | ServiceRequest, MedicationRequest per Ignite catalog | athenaOne REST APIs (broad coverage including scheduling, billing, clinical) | FHIR write where supported | ServiceRequest where supported |
| **Patient portal launch** | MyChart (patient-facing) | HealtheLife | Patient-facing portal | Patient and Consumer Health Portal | Patient portal |
| **Embedded app surface** | Hyperdrive embed (Chromium-based; replaces Hyperspace thick client) — same SMART launch flow | SMART launch from Millennium | SMART launch from athenaOne | SMART launch from Expanse | SMART launch |
| **Notable quirks** | Each customer has its own endpoint; flowsheet IDs, order IDs, document codes vary per site; Care Everywhere is the exchange network (not the API) | Tenant ID in FHIR base URL; CareAware iBus for device data | Practice ID required in every call; tenant-aware; document upload via REST not FHIR | Confirm customer is on Expanse before assuming FHIR access | Smaller practices often have MIRTH-based legacy integrations |

## Other Vendors (Quick Reference)

| Vendor | App program | Notes |
|--------|-------------|-------|
| NextGen | Connect Marketplace | SMART on FHIR; legacy MIRTH-based integrations common at smaller practices |
| Allscripts / Veradigm | Veradigm Developer Program | Multiple product lines (Sunrise inpatient, TouchWorks, Pro EHR, Practice Fusion); older UAN (Unified Action Network) SOAP APIs still in heavy use alongside newer FHIR endpoints |
| OpenEMR | Self-hosted; open source | SMART on FHIR module enabled per-install; you own the sandbox and the prod instance |

## Cross-Vendor Themes

### What is consistent

- SMART on FHIR is the OAuth profile every modern EHR supports for user-facing apps.
- US Core on FHIR R4 is the data model for USCDI read.
- Bulk FHIR `$export` is supported broadly for ONC-mandated USCDI data.
- AuditEvent (or equivalent vendor audit) is required for HIPAA accounting of disclosures.

### What varies per vendor (and per customer site)

- Which write scopes are actually granted (vendor catalog vs. customer policy).
- How notes are written back (FHIR DocumentReference vs. proprietary REST vs. HL7 v2 MDM).
- How flowsheet rows, order IDs, and document type codes are coded — these are routinely customized per customer.
- The path from sandbox to production (review artifacts, security questionnaire, customer go-live).

### What you should never assume

- "The EHR's FHIR API" is one thing. Each customer site has its own endpoint, version, and enabled scopes.
- Write scopes that work in sandbox will work in production. They often need per-customer approval.
- Hardcoded flowsheet, order, or document type codes will travel between sites. They will not.
- A customer's FHIR API has every US Core profile fully populated. Coverage varies wildly, especially for social history, family history, and care-team data.

## Pre-Integration Checklist (Per Vendor)

- [ ] Confirm current program name and membership requirements on the vendor's developer portal
- [ ] Read the vendor's current capability statement at `<iss>/.well-known/smart-configuration`
- [ ] Inventory which FHIR resources, profiles, and scopes the vendor publishes
- [ ] Confirm whether write APIs need extra agreement or per-customer approval
- [ ] Confirm the customer's deployment version supports the resources you need (especially Meditech Expanse vs. Magic/C-S)
- [ ] Document each customer's endpoint, client ID, redirect URI, and enabled scopes separately
- [ ] BAA executed with each customer before any production PHI flows

> Verify each vendor's program name, sandbox URL, and current capability against the vendor's developer portal. Names and tier structures change.
