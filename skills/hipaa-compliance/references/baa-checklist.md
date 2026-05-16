# Business Associate Agreement Checklist

A generic checklist of elements typically required in a HIPAA Business Associate Agreement, drawn from the Omnibus Rule's required-contract provisions and HHS's published sample BAA. This is a working aid for engineering, vendor management, and procurement teams. It is NOT a legal template and is NOT legal advice. Verify every element against the current text of the rule and have your privacy/compliance counsel review any BAA before execution.

## Required Elements (verify against current rule text)

### Definitions and Scope
- [ ] Definition of PHI consistent with the rule
- [ ] Identification of the parties (CE, BA — and subcontractor BA where applicable)
- [ ] Description of the services and the PHI created/received/maintained/transmitted

### Permitted and Required Uses and Disclosures
- [ ] Permitted uses and disclosures by the BA — scoped to the services
- [ ] Prohibition on use or further disclosure other than as permitted or required by law
- [ ] BA's own management and administration / legal responsibilities — verify the carve-out language
- [ ] Data Aggregation services for the CE (if applicable)
- [ ] De-identification — confirm whether the BA is permitted to de-identify and use de-identified data; document any such permission

### Safeguards
- [ ] BA will implement administrative, physical, and technical safeguards reasonably and appropriately to protect ePHI as required by the Security Rule
- [ ] BA will comply with applicable Privacy Rule requirements when carrying out the CE's Privacy Rule obligations

### Reporting
- [ ] Report to the CE any use or disclosure not provided for by the BAA
- [ ] Report Security Incidents (including a definition / threshold for non-significant incidents) — verify acceptable phrasing
- [ ] Report Breaches of unsecured PHI without unreasonable delay and not later than the contractual deadline (often shorter than the rule's 60-day outside limit)

### Subcontractors
- [ ] BA will ensure any subcontractor that creates, receives, maintains, or transmits PHI on behalf of the BA agrees to the same restrictions and conditions (downstream BAA)
- [ ] Maintain a list of subcontractor BAs (operational practice, not always a contract requirement)

### Individual Rights Support
- [ ] Access (provide PHI in the designated record set the BA holds, supporting the CE's response)
- [ ] Amendment
- [ ] Accounting of disclosures
- [ ] Restrictions and confidential communications (as applicable to the BA's role)

### Availability to HHS
- [ ] Make internal practices, books, and records available to HHS for compliance review

### Term and Termination
- [ ] Term aligned with the underlying services agreement
- [ ] Termination for material breach of the BAA
- [ ] Return or destruction of PHI on termination, or extension of protections if return/destruction is infeasible, with the limitations on use spelled out

### General Provisions
- [ ] Regulatory references resolve to the current text (avoid hard-coded cross-references that may drift)
- [ ] Survival of relevant obligations post-termination
- [ ] Order-of-precedence with the underlying agreement (typically, the BAA controls for PHI matters)

## Items to Negotiate or Verify

These commonly surface in BAA negotiation and should be raised with counsel and the security/privacy team:

- **Breach notification timeline** — the rule sets a 60-day outside limit; CEs commonly require much shorter (e.g., 24-72 hours)
- **Security Incident definition** — broad definitions can create reporting noise; narrowed-with-exceptions language is common
- **Indemnification and liability caps** — outside the BAA's regulatory scope but in the underlying contract
- **Insurance requirements** — cyber liability, technology E&O — separate from regulatory text
- **Audit rights** — CEs often request rights to audit the BA; BAs typically counter with SOC 2 / HITRUST / pen-test deliverables
- **Specific safeguards (encryption, MFA, retention)** — many CEs append a security exhibit
- **Use of de-identified data and aggregated data for the BA's own purposes** — confirm whether permitted and on what basis
- **Use of PHI for AI/ML training** — increasingly material; usually requires explicit, narrow permission and is often outside the CE's authority to grant
- **Choice of law and venue**
- **HITECH-style fees for data access requests** — verify the underlying services agreement aligns

## Subcontractor (Downstream) BAA

When the BA engages a subcontractor that touches PHI, a downstream BAA is required. The downstream BAA must impose substantially the same restrictions and conditions on the subcontractor that apply to the BA. Maintain:

- [ ] Inventory of subcontractor BAs with the categories of PHI they touch
- [ ] Renewal/expiration tracking
- [ ] Periodic verification that subcontractor safeguards are still in place
- [ ] Cloud Service Providers that store/process ePHI are BAs even if they never access the data (per HHS Cloud Computing Guidance) — verify CSP has a current BAA

## Cloud Service Provider Considerations

- [ ] CSP is contracted as a BA where it stores/processes ePHI
- [ ] Shared responsibility model is documented (what the CSP secures vs. what the customer secures)
- [ ] CSP regions / data residency match contractual commitments
- [ ] CSP exit / data return / data destruction terms are specified

## Final Disclaimers

- This checklist is a working aid drawn from generic descriptions of the rule's requirements and the HHS sample BAA — verify against the current text of the rule
- Do not execute any BAA without counsel review
- Specific clauses, including breach-notification timelines and security exhibits, should be tailored to the relationship and the risk profile — consult your privacy/compliance counsel
