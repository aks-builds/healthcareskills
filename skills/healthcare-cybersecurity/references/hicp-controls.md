# HHS 405(d) HICP — Top Control Areas by Organization Size

A working summary of the HHS 405(d) Health Industry Cybersecurity Practices (HICP) control areas, indexed by the practice volume (small / medium / large) that maps to organization size. The HICP publication is updated periodically — verify the current release on the HHS 405(d) site for the exact practice numbering, sub-practice text, and threat mapping. This reference is a working aid, not a substitute for the current HICP document.

## Threats HICP Targets

HICP organizes its practices around the high-impact threats observed in the healthcare sector:

1. Email phishing attack
2. Ransomware attack
3. Loss or theft of equipment or data
4. Insider, accidental or intentional data loss
5. Attacks against connected medical devices that may affect patient safety

A sixth threat (commonly framed around supply chain / attacks against network-connected services) appears in some HICP releases — verify against the current edition.

## Practice Volumes

| Practice volume | Audience |
|---|---|
| Small | Small clinics, small physician practices, FQHCs |
| Medium | Mid-sized hospitals, multi-clinic groups, regional health systems |
| Large | Large health systems, integrated delivery networks, academic medical centers |

The practices below are listed at the domain level. Sub-practices vary by volume — small-practice text typically emphasizes off-the-shelf cloud tools and prescriptive defaults; large-practice text addresses governance, threat hunting, and enterprise-scale architecture.

## Top Control Areas (verify against current HICP)

### 1. Email Protection Systems
- Anti-phishing / DMARC / SPF / DKIM
- Email gateway filtering (attachments, sandboxing, URL rewriting)
- User training (simulated phishing campaigns)
- Reporting workflow (one-click report to security)

### 2. Endpoint Protection Systems
- Centrally managed EDR / next-gen AV
- Hardening baselines (CIS Benchmarks or equivalent)
- Patching cadence and exception management
- Device encryption (laptops, mobiles, USB)
- Mobile device management (MDM) for clinical mobile devices

### 3. Access Management
- Unique user IDs across all systems
- Phishing-resistant MFA where feasible
- Privileged access management (PAM) for admin / break-the-glass
- Joiners-Movers-Leavers (JML) automation
- Periodic access certification (clinical staff, vendors)

### 4. Data Protection and Loss Prevention
- Data classification and inventory (PHI / ePHI / PII / financial)
- Encryption at rest and in transit (see phi-handling encryption-cheatsheet)
- DLP for outbound email and cloud
- Secure remote access (VPN, ZTNA)
- Backup and tested restore (offline / immutable copy)

### 5. Asset Management
- Hardware and software inventory (clinical workstations, medical devices, mobile, cloud assets)
- License and end-of-life tracking
- Decommissioning with NIST SP 800-88 sanitization
- Configuration management database (CMDB) integration

### 6. Network Management
- Network segmentation — clinical, business, medical devices, guest
- Egress filtering (block unnecessary destinations)
- Wireless security (WPA3 / 802.1X / separate guest network)
- DNS filtering / protective DNS
- Zero-trust / micro-segmentation maturity (large practice)

### 7. Vulnerability Management
- Authenticated vulnerability scans on a defined cadence
- Patching SLAs by severity
- Risk-based exception handling
- Penetration testing (annual / on major changes)
- Threat intelligence (sector ISAC participation — H-ISAC)

### 8. Incident Response
- Written plan tailored to clinical operations
- Downtime procedures (paper backup, alternate workflows)
- IR retainer / external counsel / cyber insurance integration
- Tabletop exercises (annual minimum; ransomware-specific recommended)
- Post-incident review and lessons learned
- Reporting (HHS / OCR, FBI / CISA, sector ISAC) per current guidance

### 9. Medical Device Security
- Inventory with vendor, model, OS, network zone, vendor-supplied MDS² form (see medical-device-cybersecurity.md)
- SBOM ingestion from vendors
- Network segmentation aligned to IEC 80001
- Passive monitoring for high-risk devices that cannot be actively scanned
- Procurement requirements (security questionnaire, MDS², SBOM, patching commitments)
- Coordinated vulnerability disclosure path

### 10. Cybersecurity Oversight and Governance
- Named Security Officer / CISO
- Reporting line to executive leadership and board
- Policies and procedures lifecycle
- Workforce training and security awareness
- Risk register tied to the HIPAA Security Rule risk analysis
- Third-party / vendor risk management
- Metrics and KPIs

(Some HICP releases also include a separate practice area for connected medical devices vs. medical device security as part of asset management; verify the current numbering.)

## Mapping to Other Frameworks

HICP is generally cross-walked to:

- HIPAA Security Rule (Administrative / Physical / Technical safeguards — see hipaa-compliance/references/safeguards-checklist.md)
- NIST CSF (Govern / Identify / Protect / Detect / Respond / Recover)
- NIST SP 800-53
- HITRUST CSF (see hitrust-csf)

HHS / OCR has stated that demonstrating use of "recognized security practices" (which include HICP) for the prior 12 months is a factor in penalty determinations under the HITECH-amended HIPAA framework. Document the practices actually in place and the evidence — verify the current rule text and OCR guidance for the exact language.

## Small Practice Quick-Start (Verify Against Current HICP Small Practice Volume)

For a small clinic without a dedicated security team, common defaults:

- Cloud email with built-in phishing protection (and DMARC enforced)
- Endpoint EDR via the MSP
- MFA on email, EHR, and admin accounts (phishing-resistant where supported)
- Full-disk encryption on laptops and mobiles enabled by default
- Cloud backup with tested restore quarterly
- Documented incident response with the EHR vendor's IR contact known
- Annual security awareness training and simulated phishing
- Vendor / BAA inventory in a spreadsheet, reviewed annually

## Final Disclaimers

- Verify all practice numbering, sub-practice text, and threat mapping against the **current** HICP release on the HHS 405(d) site
- Engage your security team, MSP, or qualified assessor for binding decisions
- Consult your privacy/compliance counsel on regulatory tie-in (recognized security practices, OCR penalty calculation)
