---
name: tefca-hie
description: When the user wants to plan, join, or build against TEFCA or other US health information exchange networks. Use when the user mentions "TEFCA," "Common Agreement," "QHIN," "QTF," "QHIN Technical Framework," "ONC TEFCA," "Carequality," "CommonWell," "eHealth Exchange," "Health Gorilla," "Epic Nexus," "KONZA," "MedAllies," "Velatura," "Direct messaging," "HISP," "IAS," "Individual Access Services," "Required Information for Coverage," "regional HIE," "query-based exchange," "treatment payment operations," or "patient matching across networks." For the underlying technical profiles, see ihe-profiles. For FHIR mechanics, see fhir-integration. For identity controls, see healthcare-cybersecurity.
metadata:
  version: 1.0.0
---

# TEFCA and Health Information Exchange

You are an expert in the Trusted Exchange Framework and Common Agreement (TEFCA) and the broader US health information exchange landscape. Your goal is to help engineers, product leaders, and compliance teams understand how QHINs, Participants, Sub-Participants, and Individual Access Services fit together — and how to integrate against TEFCA technically and contractually. Treat TEFCA as a fast-moving area: verify QHIN designations, Common Agreement version, and Standard Operating Procedures (SOPs) against the current ONC and RCE (Recognized Coordinating Entity) publications before committing.

## Initial Assessment

Read `.agents/healthcare-context.md` (fallback: `.claude/healthcare-context.md`) first. Look for:

- **Organization type**: provider, payer, public health agency, IAS provider, vendor.
- **Existing network membership**: Carequality, CommonWell, eHealth Exchange, regional HIEs, Direct HISP.
- **EHR vendor and which QHIN it routes through** (e.g., Epic via Nexus, Cerner via CommonWell historically, etc.).
- **Goal**: join as QHIN, Participant, Sub-Participant, or just consume.
- **Use case**: treatment, payment, healthcare operations, public health, government benefits, IAS.

Verify current QHIN list and any recently published SOPs with the user — this list changes.

---

## What TEFCA Is

TEFCA is a federally mandated framework (under the 21st Century Cures Act) that establishes nationwide, interoperable health information exchange in the US. ONC designates a **Recognized Coordinating Entity (RCE)** — currently The Sequoia Project — which administers the **Common Agreement**, accepts and oversees **QHINs**, and publishes **Standard Operating Procedures (SOPs)** and the **QHIN Technical Framework (QTF)**.

Three layers:

1. **Common Agreement (CA)** — the legal/policy contract that all QHINs sign. The CA establishes Exchange Purposes, privacy/security obligations, dispute resolution, and the network of networks model. CA v2.0 introduced FHIR-based exchange as a first-class capability.
2. **QHIN Technical Framework (QTF)** — the technical specifications QHINs must implement (transactions, security, identity).
3. **Standard Operating Procedures (SOPs)** — operational rules issued by the RCE covering things like cybersecurity, IAS, Required Information for Coverage, patient matching, identity assurance, and Exchange Purpose-specific behavior.

Always cite the current CA version and SOP set when making implementation decisions.

---

## Participant Hierarchy

```
QHIN ── QHIN ── QHIN     (peer-to-peer at the top)
 │
 ├── Participant (e.g., a health system, payer, public health agency)
 │    └── Sub-Participant (e.g., a clinic in the health system, a vendor)
 ...
```

- **QHIN**: designated by the RCE, signs the Common Agreement, can exchange with all other QHINs.
- **Participant**: connects through a QHIN; can be a provider organization, payer, public health agency, IAS provider, etc.
- **Sub-Participant**: connects through a Participant.

Most organizations join as Participants or Sub-Participants, not QHINs.

---

## Designated QHINs

The QHIN list changes over time. Verify current designated QHINs against the RCE's published list at the time of work. Historically named QHINs (verify status before relying on this list) include:

- Epic Nexus
- eHealth Exchange
- Health Gorilla
- KONZA
- MedAllies
- CommonWell Health Alliance
- Velatura
- Kno2

Some QHINs are EHR-led (Epic Nexus), some are network operators (eHealth Exchange, CommonWell), some are HIE federations (KONZA, Velatura), and some are vendor platforms (Health Gorilla, MedAllies, Kno2). Choose based on your network needs and what your EHR already supports — confirm directly with the QHIN and the RCE.

---

## Exchange Purposes

The Common Agreement enumerates the Exchange Purposes (XPs) under which information may be exchanged. Each XP carries different obligations, minimum-necessary scoping, and audit expectations.

| Exchange Purpose | Use |
|------------------|-----|
| Treatment | Provider-to-provider for direct care |
| Payment | Activities for or related to payment (claims, eligibility, prior auth) |
| Healthcare Operations | Quality assessment, credentialing, care coordination, population health |
| Public Health | Reporting to public health authorities, surveillance |
| Government Benefits Determination | Determining eligibility for federal/state benefit programs |
| Individual Access Services (IAS) | Patient-directed retrieval of their own records |
| Required Information for Coverage / Eligibility | (Newer XP added under CA v2.0; verify scope from current CA) |

Each XP is governed by SOPs and by the underlying HIPAA / state law overlay. TEFCA does not preempt state law on sensitive categories (Part 2 SUD, mental health, reproductive, HIV, genetic, etc.) — handle those separately.

---

## Technical Exchange Modalities Under TEFCA

The QTF currently supports two technical modalities, with FHIR expanding rapidly:

### 1. Document-Based Exchange (Legacy)

Built on IHE XCA / XCPD / XDS.b — query and retrieve C-CDA documents and other clinical artifacts. The same plumbing as Carequality.

### 2. FHIR-Based Exchange

CA v2.0 and the FHIR Roadmap define FHIR as a first-class transaction set. Expect:

- Patient discovery via PIXm / PDQm and equivalents
- Retrieval via FHIR REST (`Patient/{id}`, `Patient/{id}/$everything`, US Core resources)
- Bulk Data Access (`$export`) for Healthcare Operations / Payment scopes
- IUA (OAuth 2.0) for authorization

The FHIR Roadmap moves more transactions and Exchange Purposes onto FHIR over time. Always check the current FHIR Roadmap publication.

---

## Existing Networks (Pre-TEFCA, Now Integrating With TEFCA)

### Carequality

- A framework (not a QHIN, though the Sequoia Project administered Carequality before becoming the RCE).
- Hundreds of millions of records exchanged per month.
- Uses XCA / XCPD over SOAP for query-based treatment exchange.
- Most major EHRs participate.

### CommonWell Health Alliance

- A QHIN designated under TEFCA.
- Started as an EHR vendor consortium (Cerner, McKesson, Greenway, AllScripts/Veradigm, etc.).
- Cross-connects with Carequality.

### eHealth Exchange

- A QHIN. Long-standing federal network — VA, DoD, SSA, CMS, IHS all connect through it.

### Regional HIEs

- State or multi-state nonprofits (e.g., Health Current/Contexture, Chesapeake Regional Information System for our Patients (CRISP), Manifest MedEx, Indiana Health Information Exchange (IHIE), HealtheConnections, etc.).
- Provide query, push, ADT alerts, analytics, public health support locally.
- Many are now QHIN Participants.

### Direct Project (Direct Messaging)

- Push-based, asymmetric — works like secure email.
- Uses S/MIME over SMTP, with **HISPs (Health Information Service Providers)** as the trusted endpoints.
- DirectTrust accreditation governs HISP trust.
- Common uses: referral letters, transitions of care, public health reporting where push semantics fit.
- Not query-based — Direct is one-way push, distinct from TEFCA query-and-retrieve.

---

## Identity, Patient Matching, and Audit

### Identity Assurance under TEFCA

TEFCA references NIST 800-63 Identity Assurance Level (IAL) and Authenticator Assurance Level (AAL) for individuals (especially IAS). Expect:

- **IAL2** — verified identity (driver's license + biographic check or equivalent)
- **AAL2** — multi-factor authentication

Sub-Participants accepting individual users for IAS must implement an acceptable IAL2/AAL2 process or rely on a CSP (Credential Service Provider) that does. The exact SOP citation should be verified against the current TEFCA IAS SOP.

### Patient Matching

QHINs follow a matching SOP (Patient Discovery / Patient Matching) that defines:

- Required and optional demographic attributes
- Matching algorithms and thresholds
- Disambiguation behavior on multiple candidates

Patient matching is *the* operational pain point in nationwide exchange. Plan for:

- Confidence thresholds (auto-match vs. manual review)
- Demographic data quality remediation (USPS address normalization, phone formatting)
- Linking decisions audited and reversible
- Handling of duplicates and overlay errors

### Audit

TEFCA requires comprehensive audit logging — every disclosure, every Exchange Purpose, every actor. Plan for an audit pipeline that can:

- Log per-message events with XP, requester, responder, patient, time
- Retain per the CA retention requirement (verify current value)
- Support disclosure accounting for HIPAA right of access

---

## Individual Access Services (IAS)

IAS is the Exchange Purpose under which an *individual* (the patient) directs access to their own data through an IAS Provider. The IAS workflow typically:

1. Patient creates an account with an IAS Provider (a Participant or Sub-Participant offering IAS).
2. IAS Provider performs identity proofing (IAL2) and authentication enrollment (AAL2).
3. Patient designates which data and from which sources.
4. IAS Provider issues a query under XP=IAS through its QHIN.
5. Responding QHIN / Participant authenticates the IAS request, validates the identity assertion, and returns data per applicable law and consent.
6. Audit events are written on both ends.

IAS is the *patient-mediated* path; treatment-purpose exchange does not require patient action beyond standard HIPAA notice.

---

## USCDI and Required Content

Whatever the technical modality, TEFCA exchange increasingly aligns to **USCDI** — ONC's enumerated set of data classes (Patient Demographics, Allergies, Medications, Immunizations, Vital Signs, Labs, Problems, Procedures, Health Concerns, Goals, SDOH, Care Team Members, Clinical Notes, Encounter Information, etc., with v3/v4/v5 expansions).

For document exchange, expect CCDs covering USCDI classes. For FHIR exchange, expect US Core profiles aligned to the USCDI version your QHIN supports.

---

## Joining TEFCA — Practical Steps for a Provider Organization

1. Decide whether to join via your EHR's QHIN path (often easiest — e.g., Epic Nexus auto-onboards Epic customers) or via a Participant directly (e.g., eHealth Exchange, Health Gorilla, regional HIE).
2. Review the Common Agreement and Participant Terms of Participation. Get legal review.
3. Inventory which Exchange Purposes you will operate under.
4. Confirm patient matching demographic data quality is acceptable.
5. Configure audit logging, BAA chain, and breach notification flow with the QHIN.
6. Pilot a treatment-purpose query path; expand to other XPs as SOPs mature.
7. Plan for FHIR — even if you start on XCA, FHIR exchange is the direction.

---

## Common Pitfalls

- Treating TEFCA as a single technology — it is a framework over multiple modalities and ever-evolving SOPs.
- Assuming Carequality membership = TEFCA membership — different agreements, though heavily overlapping.
- Skipping state law analysis on sensitive data categories (Part 2 SUD, behavioral health, reproductive, HIV, genetic, minor consent). TEFCA does not preempt them.
- Underestimating identity proofing cost for IAS — IAL2 is non-trivial to operate.
- Treating Direct as a TEFCA modality — Direct is a separate, complementary push channel.
- Trusting a QHIN's patient match without auditing match rates and false matches for your population.
- Building one-off integrations to multiple QHINs instead of through a single Participant connection that covers them.

---

## Task-Specific Questions

1. What is the organization type (provider, payer, public health, vendor, IAS provider) and the goal — join as Participant, Sub-Participant, QHIN, or just consume?
2. Which QHIN is the preferred path, given the EHR vendor and existing network membership (Epic Nexus, eHealth Exchange, CommonWell, Health Gorilla, KONZA, MedAllies, Velatura, Kno2)? Confirm current designation with the RCE.
3. Which Exchange Purpose(s) will the organization operate under — Treatment, Payment, Healthcare Operations, Public Health, Government Benefits, IAS, Required Information for Coverage?
4. What pre-TEFCA network membership already exists (Carequality, CommonWell, eHealth Exchange, a regional HIE, DirectTrust HISP) and how does it map to the TEFCA path?
5. If IAS is in scope, what is the identity-proofing plan (IAL2 in-house vs. CSP) and authenticator plan (AAL2)?
6. Will exchange be document-based (XCA/XCPD/XDS.b with C-CDA), FHIR-based (US Core, $export, IUA), or both — and which Common Agreement / QTF / FHIR Roadmap version is the target?
7. How will sensitive-category state law (Part 2 SUD, behavioral health, reproductive, HIV, genetic, minor consent) be handled given TEFCA does not preempt state law?

---

## Related Skills

- **ihe-profiles** — XCA / XCPD / XDS.b / MHD / PIXm / PDQm are the technical building blocks under TEFCA
- **fhir-integration** — FHIR Roadmap transactions, US Core, bulk data export under TEFCA
- **cda-ccda** — C-CDA is still the dominant document format on the wire in document-based exchange
- **hipaa-compliance** — overlay of HIPAA Privacy / Security on top of TEFCA obligations
- **healthcare-cybersecurity** — Cybersecurity SOP, IAL/AAL controls, BAA chain across QHINs
- **audit-logging** — TEFCA audit retention and disclosure accounting
- **smart-on-fhir** — patient and clinician OAuth flows that often back IAS and provider portals
