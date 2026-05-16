# Medical Device Cybersecurity Reference

A working summary of the regulatory and standards landscape for cybersecurity of connected medical devices, intended for healthcare delivery organizations (HDOs) and for manufacturers preparing premarket submissions. Verify each topic against the current FDA guidance documents and standards releases — this area evolves rapidly. Consult your regulatory affairs team and counsel for binding interpretation.

## FDA Premarket Guidance Topics

FDA has issued cybersecurity guidance for medical devices in premarket review. The current guidance (verify the latest version on FDA.gov, which has gone through several iterations and a 2023 FDORA-driven statutory anchor) typically addresses:

- **Cybersecurity in device design (Secure Product Development Framework / SPDF)** — security built into the SDLC, not bolted on
- **Threat modeling** — STRIDE, attack trees, or equivalent; document threats and mitigations
- **Cybersecurity risk management** — integrated with safety risk per ISO 14971; AAMI TIR57 is a common framework
- **Software Bill of Materials (SBOM)** — generated per device, in a machine-readable format (SPDX, CycloneDX, SWID); kept current with components and versions
- **Vulnerability management** — coordinated vulnerability disclosure (CVD) program, patching capability, post-market monitoring
- **Cybersecurity testing** — security architecture documentation, third-party testing, penetration test results
- **Labeling / documentation** — security documentation for the operator (HDO), including configuration, hardening guides, monitoring expectations
- **Architecture views** — Global System View, Multi-Patient Harm View, Updateability/Patchability View (commonly required in premarket submissions — verify against current guidance)

FDA's authority for cybersecurity expectations in premarket submissions was strengthened by the Food and Drug Omnibus Reform Act (FDORA) provisions in 2023; verify the current text of FD&C Act 524B and FDA implementing guidance.

## Postmarket Cybersecurity

FDA also has postmarket cybersecurity guidance covering:

- Ongoing vulnerability surveillance
- Coordinated vulnerability disclosure
- Risk assessment of newly-discovered vulnerabilities
- Compensating controls during a remediation window
- When changes constitute a "recall" vs. routine update
- Reporting obligations under MDR (medical device reporting)

## MDS² (Manufacturer Disclosure Statement for Medical Device Security)

The MDS² form is a HIMSS / NEMA / IEC standardized questionnaire that manufacturers fill out and provide to healthcare delivery organizations. It documents the device's security characteristics:

- Authentication (local, network, biometric, none)
- Audit controls (events captured, format, export)
- Authorization (RBAC, account management)
- Configuration management (hardening defaults, change-control)
- Data backup and disaster recovery
- Emergency access (break-the-glass)
- Health data de-identification capabilities
- Encryption (data at rest, in transit, key management)
- Endpoint protection (anti-malware, application allow-listing)
- System and information integrity (patching, BIOS protection)
- Malware detection
- Mobile device security
- Network connectivity (ports, protocols, services)
- Other security considerations
- Physical security
- Person authentication
- Personnel authentication remote service
- Roadmap for third-party components in device life cycle
- Transmission confidentiality
- Service / maintenance / vendor remote access

HDOs use the MDS² as a procurement screening tool and as input to their risk register. Procurement should require an MDS² before purchase.

## SBOM Formats

| Format | Notes |
|---|---|
| **SPDX** | ISO/IEC 5962 standard; broad tooling support; verify current version |
| **CycloneDX** | OWASP project; rich vulnerability and risk metadata; common in DevSecOps tooling |
| **SWID** | ISO/IEC 19770-2; software identification tags; less common in medical devices |

SBOMs should:

- Be machine-readable
- Include components, versions, suppliers, dependency relationships
- Be updated with each release
- Be ingested by the HDO into a tool that maps to known vulnerabilities (CVE, KEV)

CISA publishes SBOM guidance and minimum elements; verify against the current CISA SBOM minimum elements document.

## IEC 80001 Series — Risk Management of Medical IT Networks

IEC 80001 is the standard family addressing the risk management of IT networks that incorporate medical devices. Key parts:

- **IEC 80001-1** — Application of risk management for IT networks incorporating medical devices; defines responsibilities of the HDO, medical device manufacturer, and IT provider
- **Other parts (technical reports / supporting standards)** — security, wireless networks, disclosure of security needs and risks

Operationally, IEC 80001 places responsibility on the HDO to maintain a Medical IT Network Risk Manager role (or equivalent function) and to perform risk management activities throughout the network lifecycle: planning, design, implementation, operation, maintenance, decommissioning.

## IEC 62443 — Industrial Cybersecurity

IEC 62443 is the industrial automation cybersecurity standard family (originally ISA/IEC 62443). It is increasingly referenced in medical device cybersecurity for:

- Foundational requirements (FRs): identification & authentication, use control, system integrity, data confidentiality, restricted data flow, timely response to events, resource availability
- Security levels (SL 1 through SL 4)
- Zones and conduits — segmentation model that maps cleanly to medical-device network architecture
- Component / system / process requirements

For HDOs, IEC 62443 informs network segmentation patterns (clinical zone, medical-device zone, business zone, with conduits / gateways between them). For manufacturers, it informs secure development practices.

## Network Segmentation Patterns

Common HDO segmentation patterns aligned to IEC 80001 / 62443:

| Zone | Examples | Restrictions |
|---|---|---|
| Medical device — patient-connected | Infusion pumps, ventilators, monitors, anesthesia, imaging modalities | No direct internet; whitelisted vendor cloud only; passive monitoring; clinical priority routing |
| Medical device — diagnostic | PACS, lab analyzers, EKG carts | Vendor-defined patching windows; segmented from business |
| Clinical workstations | Nursing workstations, physician workstations | EDR, application allow-list, full-disk encryption |
| Business / corporate | Admin laptops, finance, HR | Standard enterprise controls |
| Guest / patient Wi-Fi | Patient and visitor devices | Fully isolated from clinical and business |

Avoid flat networks where medical devices share the broadcast domain with general endpoints — this is one of the highest-impact findings in healthcare cybersecurity assessments.

## Procurement and Lifecycle Controls

For HDOs, security in procurement and lifecycle should include:

- Security questionnaire (MDS² + supplemental)
- SBOM delivery and refresh commitment
- Patching cadence in the contract (vendor SLAs)
- Coordinated vulnerability disclosure (CVD) point of contact
- End-of-life / end-of-support communication
- Pre-deployment hardening verification
- Acceptance testing including security configuration
- Connectivity review (which ports, protocols, services; which destinations)
- Ongoing posture monitoring (medical-device security platforms — passive)

## Coordinated Vulnerability Disclosure (CVD)

Manufacturers should publish a CVD policy aligned to ISO/IEC 29147 and ISO/IEC 30111 (verify current versions). HDOs should know:

- Who at the vendor to contact when a vulnerability is discovered
- The vendor's SLA for acknowledgement and remediation
- Whether MITRE will assign CVEs
- The vendor's notification mechanism for fielded devices

## Patient Safety as the First Constraint

Unlike enterprise IT, the highest priority constraint on medical device cybersecurity is patient safety. A control that breaks a clinical workflow can endanger patients. Engineering implications:

- Tabletop with clinical leadership; segmentation cannot break the standard of care
- Patching windows aligned to clinical workflows, not IT maintenance windows
- Compensating controls when patching is infeasible (passive monitoring, isolation, vendor risk acceptance)
- Documented downtime procedures for medical-device incidents

## Final Disclaimers

- Verify all FDA guidance references against the current document on FDA.gov — premarket and postmarket cybersecurity guidance has been updated multiple times
- Verify IEC 80001 and IEC 62443 part numbers and current versions
- Verify CISA SBOM minimum elements and current CISA HPH-sector guidance
- Consult regulatory affairs and counsel for premarket submission decisions and postmarket reporting obligations
