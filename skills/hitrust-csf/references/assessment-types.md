# HITRUST Assessment Types — e1 vs. i1 vs. r2

A working comparison of the three primary HITRUST validated assessment types. HITRUST has periodically refined its assessment portfolio — verify the current portfolio, control counts, cycle, and scoring rules against the current MyCSF documentation and HITRUST's published guidance. Specific control counts evolve with each CSF version; the numbers in this reference should be confirmed before relying on them. Consult an Authorized External Assessor for binding interpretation.

## At a Glance

| Topic | e1 | i1 | r2 |
|---|---|---|---|
| Full name (verify current branding) | Essentials, 1-year | Implemented, 1-year (threat-adaptive) | Risk-based, 2-year |
| Driver | Foundational / entry-level | Strong baseline; threat-adaptive | Comprehensive, tailored, highest assurance |
| Control scope | Small static set (verify current count) | Mid-sized set focused on cyber hygiene (verify) | Large tailored set scoped to organization (verify) |
| Tailoring | Static, no tailoring | Limited tailoring | Full tailoring via MyCSF factors |
| Maturity scoring | Implementation-focused | Implementation-focused | PRISMA maturity (Policy / Process / Implemented / Measured / Managed) — verify current model |
| Validity period | 1 year | 1 year | 2 years with interim assessment at year 1 |
| Audience | Small organizations, startups, lower-risk vendors | Mid-market, organizations under threat-adaptive assurance | Large healthcare orgs, payers, CSPs, vendors selling to enterprise health systems |
| Inheritance | Limited | Partial | Full inheritance program eligible |

## When Each Fits

### e1 — Foundational

- New organizations or first-time HITRUST assessees
- Small vendors selling into healthcare for whom enterprise certification is overkill
- Stepping stone toward i1 or r2
- Quick "we have a HITRUST assessment" signal to customers who accept it as a baseline

**Limits**: Lower assurance signal. Not always accepted by large enterprise health-system buyers who expect r2.

### i1 — Implemented, Threat-Adaptive

- Mid-market organizations that need a credible cyber posture certification on a 1-year cycle
- Organizations where the threat landscape is moving and the control set should adapt
- A practical step between e1 and r2 — many organizations land here as their target
- Suitable for many digital health SaaS vendors selling to mid-size health systems

### r2 — Risk-Based, 2-Year

- Enterprise health systems, major payers, large vendors, CSP-tier participants
- Customers contractually require r2
- High-assurance position that's tailored to the organization's actual risk factors
- Multi-regime evidence story (HIPAA + HITECH + GDPR + state laws + sectoral)

**Cost and effort**: r2 is the most expensive and longest assessment. Plan multi-quarter implementation, evidence collection, and assessor engagement.

## Scoping in MyCSF

The MyCSF questionnaire collects organizational factors that drive control applicability for r2 (and to a lesser extent for the other types). Factors typically include:

- Organization size and type (CE, BA, vendor, CSP)
- Data types (PHI, PII, financial, PCI)
- Regulatory drivers (HIPAA, GDPR, NIST 800-171, CMMC, state laws, sectoral)
- Geography
- Volume of records
- External-facing services
- Cloud service usage and CSP relationships

The output is a tailored control set. **Scope is the most powerful lever** for managing assessment effort — over-scoping (e.g., declaring the entire enterprise instead of the certified product boundary) inflates work without commensurate benefit. Engage the External Assessor early on scoping.

## PRISMA Maturity Model (r2)

HITRUST r2 scores each control on a maturity scale (verify the current named levels and weights):

| Level | What it asks |
|---|---|
| Policy | Is the requirement documented in policy? |
| Process | Is there a procedure operationalizing the policy? |
| Implemented | Is the requirement actually deployed? |
| Measured | Is the implementation measured (KPIs, metrics, monitoring)? |
| Managed | Is the measurement used to drive corrective action and continual improvement? |

Each level has a weighting; the final score per control is the weighted sum. To achieve r2 certification, controls must reach a threshold score. Verify the current pass thresholds and weights with HITRUST documentation.

## Assessment Lifecycle (Generalized)

1. **Decision and scoping** — pick assessment type, define scope, engage External Assessor
2. **Readiness / gap assessment** — internal self-assessment in MyCSF (or with assessor support) to surface gaps
3. **Remediation** — implement controls, gather evidence, mature processes
4. **Validated assessment** — External Assessor reviews and tests evidence per control
5. **Quality assurance** — HITRUST itself performs QA on the assessor's work
6. **Certification** — HITRUST issues the certification letter and report
7. **Interim assessment (r2 only)** — at year 1 of the 2-year cycle, a subset re-tested
8. **Re-certification** — next full cycle

## Inheritance and Cross-Mapping

HITRUST publishes Authoritative Source mappings — controls cross-walked to:

- HIPAA Security Rule (the primary mapping for U.S. healthcare)
- HIPAA Privacy and Breach Notification
- NIST CSF
- NIST 800-53
- ISO 27001/27002
- PCI DSS
- GDPR
- CCPA / CPRA
- CMMC
- FedRAMP (for CSP-tier)
- State laws (varies)

This is part of the value proposition: one r2 assessment produces evidence usable across multiple customer questionnaires and regulatory positions.

CSP inheritance is its own program — see inheritance-program.md.

## Common Decision Patterns

| Situation | Likely choice |
|---|---|
| First-time, small org, prove baseline to customers | e1 |
| Want a credible 1-year threat-adaptive cert | i1 |
| Enterprise health-system customer demands r2 | r2 |
| Already SOC 2 Type 2 and customers ask for HITRUST | i1 or r2 depending on customer expectations |
| Acquired a company; need to integrate into your r2 boundary | Add scope; engage assessor |
| Multi-regime (HIPAA + GDPR + state) signal | r2 with the right factors selected |

## Final Disclaimers

- Verify current control counts, scoring rules, and assessment-portfolio names against current HITRUST documentation
- Verify the current PRISMA model and pass thresholds
- Engage an Authorized External Assessor for binding scoping and gap-assessment decisions
- Consult leadership and counsel on contractual / customer commitments to specific assessment types
