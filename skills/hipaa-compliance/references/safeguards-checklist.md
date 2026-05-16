# HIPAA Security Rule Safeguards Checklist

Use this as a working checklist when planning, reviewing, or evidencing the Security Rule safeguards. Items marked **Required** must be implemented as specified. Items marked **Addressable** must be implemented as specified, implemented with a documented equivalent alternative, or documented as not reasonable and appropriate (with the rationale and the compensating approach). The "addressable" label does NOT mean optional.

Always verify the current text of the rule and any subsequent HHS modifications; cite section numbers from the source, not from memory. Consult your privacy/compliance counsel and Security Officer for binding interpretation.

## Administrative Safeguards

### Security Management Process
- [ ] **Required** — Risk analysis covering all ePHI, with documented scope, threats, vulnerabilities, likelihood, impact
- [ ] **Required** — Risk management — implement security measures sufficient to reduce risks to a reasonable and appropriate level
- [ ] **Required** — Sanction policy for workforce members who violate policies
- [ ] **Required** — Information system activity review (log review SOP; see audit-logging skill)

### Assigned Security Responsibility
- [ ] **Required** — Designate a Security Official responsible for policies and procedures

### Workforce Security
- [ ] **Addressable** — Authorization and/or supervision of workforce with ePHI access
- [ ] **Addressable** — Workforce clearance procedure (background where appropriate)
- [ ] **Addressable** — Termination procedures (access removal, badge/asset return)

### Information Access Management
- [ ] **Required** — Isolating healthcare clearinghouse functions (if applicable to a hybrid entity)
- [ ] **Addressable** — Access authorization (procedure for granting access)
- [ ] **Addressable** — Access establishment and modification

### Security Awareness and Training
- [ ] **Addressable** — Security reminders (verify current cadence policy)
- [ ] **Addressable** — Protection from malicious software
- [ ] **Addressable** — Log-in monitoring
- [ ] **Addressable** — Password management (or modern equivalents — phishing-resistant MFA, passwordless)

### Security Incident Procedures
- [ ] **Required** — Response and reporting procedures

### Contingency Plan
- [ ] **Required** — Data backup plan
- [ ] **Required** — Disaster recovery plan
- [ ] **Required** — Emergency mode operation plan
- [ ] **Addressable** — Testing and revision procedures (tabletops, restore tests)
- [ ] **Addressable** — Applications and data criticality analysis

### Evaluation
- [ ] **Required** — Periodic technical and non-technical evaluation against the standards (cadence to verify against current HHS expectations)

### Business Associate Contracts
- [ ] **Required** — Written contract or other arrangement (see baa-checklist.md)

## Physical Safeguards

### Facility Access Controls
- [ ] **Addressable** — Contingency operations (access during emergency mode)
- [ ] **Addressable** — Facility security plan
- [ ] **Addressable** — Access control and validation procedures
- [ ] **Addressable** — Maintenance records

### Workstation Use
- [ ] **Required** — Policies for proper functions, manner, and physical attributes of workstations accessing ePHI

### Workstation Security
- [ ] **Required** — Physical safeguards for workstations (clinical floor placement, screen privacy filters, kiosk modes)

### Device and Media Controls
- [ ] **Required** — Disposal (NIST SP 800-88 sanitization)
- [ ] **Required** — Media re-use
- [ ] **Addressable** — Accountability (chain of custody)
- [ ] **Addressable** — Data backup and storage before equipment movement

## Technical Safeguards

### Access Control
- [ ] **Required** — Unique user identification
- [ ] **Required** — Emergency access procedure (the "break-the-glass" path — see phi-handling and audit-logging)
- [ ] **Addressable** — Automatic logoff
- [ ] **Addressable** — Encryption and decryption (addressable here, but encryption to HHS specifications enables the breach-notification safe harbor)

### Audit Controls
- [ ] **Required** — Hardware, software, and/or procedural mechanisms that record and examine activity in systems containing ePHI (see audit-logging skill)

### Integrity
- [ ] **Addressable** — Mechanism to authenticate ePHI (detect improper alteration/destruction)

### Person or Entity Authentication
- [ ] **Required** — Verify the identity of the person/entity accessing ePHI (MFA strongly recommended; verify current HHS guidance on recognized security practices)

### Transmission Security
- [ ] **Addressable** — Integrity controls
- [ ] **Addressable** — Encryption (TLS 1.2+ in practice; verify current expectations)

## Organizational Requirements

- [ ] BAAs and equivalent arrangements (see baa-checklist.md)
- [ ] Requirements for group health plans (if applicable)

## Policies, Procedures, and Documentation

- [ ] **Required** — Maintain written policies and procedures
- [ ] **Required** — Retain documentation for 6 years from creation or last effective date — verify state law overlays
- [ ] **Required** — Availability of documentation
- [ ] **Required** — Updates in response to environmental or operational changes

## Items Commonly Marked "Verify"

These vary or have been the subject of HHS rulemaking and guidance updates — verify current status with the rule text and current HHS guidance rather than memory:

- Specific encryption algorithm requirements and key length expectations
- MFA expectations and "phishing-resistant" language
- Audit log retention floor (HIPAA documentation 6-year rule applies to required documentation; system audit log retention is risk-based — verify against current HHS / OCR guidance and any state/CMS overlays)
- Reasonable response time for ePHI access requests under recent rulemaking
- Modernizations of the Security Rule under recent NPRMs (e.g., proposed encryption-required, MFA-required text — confirm whether finalized)

Consult your privacy/compliance counsel for binding interpretation. The checklist above is a working aid, not legal advice.
