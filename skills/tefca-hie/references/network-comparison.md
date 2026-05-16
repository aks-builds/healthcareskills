# US Health Information Exchange Network Comparison

Reference for the major US-wide health information exchange networks, their relationships to TEFCA, and how to choose among them. **The QHIN list and feature claims change frequently — always verify current designations and capabilities against the RCE's published list and each network's own conformance statement at the time of work.**

## Contents

- Disambiguation: Framework vs. Network vs. QHIN vs. HIE
- TEFCA (the framework)
- Carequality (the framework that pre-dates TEFCA)
- CommonWell Health Alliance (network + QHIN)
- eHealth Exchange (long-standing federal network + QHIN)
- Epic Nexus (EHR-led QHIN)
- Health Gorilla (vendor platform QHIN)
- KONZA (HIE federation QHIN)
- MedAllies (HISP / network QHIN)
- Velatura (HIE federation QHIN)
- Kno2 (vendor platform QHIN)
- DirectTrust / Direct Messaging
- Regional HIEs
- Choosing a Path

---

## Disambiguation: Framework vs. Network vs. QHIN vs. HIE

These terms get conflated. Keep them straight:

- **TEFCA** — a framework with a Common Agreement, SOPs, and a QHIN Technical Framework. It is not itself a network. ONC defines it; the RCE administers it.
- **QHIN (Qualified Health Information Network)** — designated by the RCE under TEFCA to participate at the top of the network-of-networks. Signs the Common Agreement.
- **Network** — a collection of organizations that exchange health information under a shared agreement (e.g., Carequality, CommonWell). A network may or may not be a QHIN.
- **HIE (Health Information Exchange)** — a generic term for any health-information exchange organization or system. Regional HIEs are non-profits operating in a state or multi-state area. Some HIEs are QHIN Participants; some are not.
- **HISP (Health Information Service Provider)** — operates Direct Messaging endpoints (separate from TEFCA exchange).

---

## TEFCA (the framework)

- **Type**: Federal framework
- **Administered by**: ONC (sets policy) and the RCE (currently The Sequoia Project — verify) (administers)
- **Common Agreement**: legal contract signed by QHINs and their participants
- **QHIN Technical Framework (QTF)**: technical spec for QHIN-to-QHIN transactions
- **Standard Operating Procedures (SOPs)**: rules covering cybersecurity, patient matching, IAS, identity assurance, sensitive categories, etc.
- **Modalities**: document-based (XCA / XCPD / XDS.b with C-CDA) and FHIR-based (US Core, $export, IUA — expanding under the FHIR Roadmap)
- **Exchange Purposes**: Treatment, Payment, Healthcare Operations, Public Health, Government Benefits Determination, IAS, Required Information for Coverage (verify current list against current Common Agreement)

Always cite the Common Agreement version and SOP set when making design decisions.

---

## Carequality (the framework that pre-dates TEFCA)

- **Type**: Framework (administered by Sequoia Project; Sequoia is now the RCE under TEFCA but Carequality remains as a separate framework)
- **Scope**: Cross-network query-based document exchange
- **Volume**: Hundreds of millions of records exchanged per month historically — verify current
- **Technical**: XCA / XCPD over SOAP
- **Members**: Major EHRs (Epic, Oracle Health/Cerner, Athenahealth, eClinicalWorks, Meditech, etc.), HIEs, payers
- **Relationship to TEFCA**: Carequality membership and TEFCA participation are distinct legal arrangements. Many organizations are in both. The networks interconnect and many Carequality members are moving traffic to TEFCA paths.

---

## CommonWell Health Alliance

- **Type**: Network + designated QHIN (verify current designation)
- **History**: Founded as an EHR vendor consortium (Cerner, McKesson, Greenway, Allscripts/Veradigm, etc.) circa 2013
- **Scope**: Patient matching and document exchange across member EHRs
- **Technical**: XDS / XCA-style document exchange; expanding FHIR support
- **Relationship to Carequality**: cross-connects, so a CommonWell member can reach Carequality members and vice versa
- **Relationship to TEFCA**: designated QHIN (verify)
- **Typical members**: EHR vendors and their customer organizations

---

## eHealth Exchange

- **Type**: Network + designated QHIN (verify)
- **History**: Long-standing federal exchange, evolved from the Nationwide Health Information Network (NwHIN) Exchange under ONC
- **Scope**: Connects federal agencies (VA, DoD, SSA, CMS, IHS), state HIEs, and large provider organizations
- **Technical**: XCA / XCPD-based, expanding FHIR
- **Distinctive**: the only network with deep federal-agency participation (SSA disability adjudication is a notable use case)
- **Membership**: providers, payers, public health, federal agencies

---

## Epic Nexus

- **Type**: Designated QHIN (verify)
- **Operator**: Epic Systems Corporation
- **Scope**: Auto-onboards Epic customer organizations into TEFCA via Epic's existing infrastructure (Care Everywhere, Share Everywhere)
- **Technical**: XCA / XCPD-based with FHIR support via Epic's APIs
- **Distinctive**: For Epic customers, this is often the path of least resistance — Epic handles much of the technical onboarding automatically
- **Membership**: Epic customer organizations

---

## Health Gorilla

- **Type**: Vendor platform + designated QHIN (verify)
- **Operator**: Health Gorilla, Inc.
- **Scope**: Aggregation network and FHIR API platform
- **Technical**: FHIR-forward; supports XCA / XCPD as well
- **Distinctive**: Strong FHIR API offering; popular with digital-health startups and apps that need a single integration point to many sources
- **Membership**: Providers, apps, payers

---

## KONZA

- **Type**: HIE federation + designated QHIN (verify)
- **Operator**: KONZA National Network
- **Scope**: Federation of regional HIEs and rural health networks
- **Distinctive**: HIE-centric focus; participants are often HIEs themselves
- **Membership**: HIEs, providers in non-Epic-dominant markets

---

## MedAllies

- **Type**: HISP + designated QHIN (verify)
- **Operator**: MedAllies
- **Scope**: Originally a DirectTrust HISP; now operates as a QHIN
- **Distinctive**: Bridges Direct messaging and TEFCA query-based exchange
- **Membership**: Providers using Direct messaging plus TEFCA query

---

## Velatura

- **Type**: HIE federation + designated QHIN (verify)
- **Operator**: Velatura (formerly Michigan Health Information Network and others)
- **Scope**: Federation of regional HIEs
- **Distinctive**: HIE federation focus similar to KONZA
- **Membership**: HIEs and their participants

---

## Kno2

- **Type**: Vendor platform + designated QHIN (verify)
- **Operator**: Kno2
- **Scope**: Document exchange platform with QHIN designation
- **Distinctive**: Strong in post-acute, long-term care, and behavioral health connectivity
- **Membership**: Post-acute providers, behavioral health organizations, primary care groups

---

## DirectTrust / Direct Messaging

- **Type**: Push-based messaging framework (not a QHIN, not a TEFCA modality)
- **Scope**: Secure email-style push between DirectTrust-accredited HISPs
- **Technical**: S/MIME over SMTP
- **Use cases**: Referral letters, transitions of care, public health reporting where push semantics fit
- **Relationship to TEFCA**: Direct messaging is complementary to TEFCA query-based exchange — they coexist; some QHINs (MedAllies, Kno2) bridge both

Direct is a one-way push channel; TEFCA is primarily query-and-retrieve. Different problems, different tools.

---

## Regional HIEs

State and multi-state non-profits providing exchange, ADT alerts, analytics, and public-health support locally.

Examples (each operates in its own region; verify current naming and reach):

- CRISP (Chesapeake Regional Information System for our Patients) — DC, MD, WV, AK, CT
- Manifest MedEx — California
- Indiana Health Information Exchange (IHIE) — Indiana
- HealtheConnections — Central NY
- HEALTHeLINK — Western NY
- Bronx RHIO — NYC
- Contexture (Arizona / Colorado / etc.)
- Great Lakes Health Connect / Velatura — Michigan
- East Tennessee Health Information Network (etHIN) — Eastern TN
- Healthix — NYC

Many regional HIEs are now QHIN Participants or Sub-Participants, providing the local on-ramp.

---

## Choosing a Path

| Situation | Recommended approach |
|-----------|---------------------|
| Provider organization on Epic | Epic Nexus (QHIN) is usually the simplest path |
| Provider organization on Oracle Health (Cerner) | Historically CommonWell-mediated; verify current QHIN routing |
| Provider organization on athenahealth / eClinicalWorks / Meditech | Check vendor's QHIN routing; eHealth Exchange or Health Gorilla often relevant |
| Federal agency | eHealth Exchange |
| Digital health app / startup needing a single integration | Health Gorilla (or a similar aggregator) |
| Post-acute / behavioral health / long-term care | Kno2 |
| Regional HIE participant | Connect through your regional HIE's QHIN affiliation |
| State public health agency | Connect through eHealth Exchange or the state's HIE; verify Public Health XP coverage |
| Payer for HEDIS / risk adjustment | A QHIN that supports Healthcare Operations bulk export (verify current capability) |
| Want to operate as a QHIN | Engage with the RCE; significant operational, financial, and legal undertaking |

For most provider organizations, the right answer is: **join the QHIN your EHR vendor recommends, plus your regional HIE for local data flow, and use Direct messaging for push-style transitions.**

---

## Verification Reminders

- The list of designated QHINs **changes over time**. Verify against the RCE's published list before relying on any specific QHIN being designated.
- Common Agreement and SOP versions update. Pin your design to a specific version.
- Feature claims (FHIR support, Bulk Data, specific Exchange Purposes) vary by QHIN and version — read the QHIN's published Designation conformance / capabilities, not marketing material.
- Network membership changes — organizations join, leave, merge.
- Regulatory environment shifts — ONC and CMS rules around interoperability, information blocking, and TEFCA participation continue to evolve.

When in doubt, contact the RCE directly and require written confirmation of capabilities from any QHIN being considered.
