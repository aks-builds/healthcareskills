# Clearinghouse Comparison

Reference for engineers selecting or integrating with a US healthcare clearinghouse. Summarizes the major clearinghouses by typical capability profile. Does not enumerate pricing, payer connection counts, or proprietary features — those change continuously.

> Verify current capabilities, payer connection lists, connectivity options, and contract terms directly with the clearinghouse before architectural commitments. Clearinghouse capabilities and corporate structures change.

## Contents

- What a clearinghouse does
- Typical capability dimensions
- Major clearinghouses (high-level overview)
- Direct payer connections vs. clearinghouse
- Connectivity patterns
- Engineering integration notes
- Selection considerations

---

## What a Clearinghouse Does

A clearinghouse provides at minimum:

- **Trading partner connectivity** — established connections to a large fleet of US payers.
- **Format normalization** — accept claims/transactions in various formats (X12, proprietary, XML/JSON, CSV) and emit X12 5010 to the payer (or accept X12 5010 from payer and translate inbound for the provider).
- **Front-end edits** — syntactic and basic content validation before forwarding to the payer; rejections returned to the submitter.
- **Acknowledgment routing** — TA1, 999, 277CA, 835 routed back to the submitter and tied to the original submission.
- **Payer enrollment** — manage the per-payer / per-provider enrollment paperwork required for ERA / EFT and 837 routing.
- **Workflow UIs** — rejection-management dashboards, claim resubmission, eligibility lookup, claim status.
- **Reporting** — submission, rejection, and payment-cycle analytics.

Some clearinghouses also offer revenue-cycle services (denial management, A/R follow-up, patient statements, billing services) as separate products.

---

## Typical Capability Dimensions

When evaluating a clearinghouse for an integration, compare on:

- **Payer reach** — number and identity of payers reachable (Medicare, Medicaid by state, MA plans, commercial, dental, workers' comp, government).
- **Transaction support** — 837P / 837I / 837D, 835, 270/271, 276/277, 278, NCPDP D.0 (pharmacy via separate clearinghouses typically), 834.
- **Connectivity options** — SFTP, AS2, CAQH CORE Phase II Connectivity, REST/SOAP APIs, browser-based portal.
- **Edit content** — depth of pre-submission scrubbing, NCCI inclusion, payer-specific edits, customizable rules.
- **Real-time vs. batch** — 270/271 and 276/277 real-time response; batch claim submission cadence.
- **Acknowledgment timing** — turnaround for 999, 277CA, 835 from each payer.
- **ERA enrollment management** — automation of payer-by-payer ERA / EFT enrollment.
- **Reporting and APIs** — programmatic access to status, rejections, and analytics.
- **PHI / HIPAA controls** — BAA, encryption in transit and at rest, audit logging, breach response.
- **Pricing model** — per-transaction, per-provider, per-claim, bundled, hybrid.

---

## Major Clearinghouses (High-Level Overview)

The list below is non-exhaustive and reflects the US clearinghouse landscape at a general level. Corporate ownership and brands change.

> Verify current ownership, capability profile, payer reach, and product lines directly with each vendor.

### Change Healthcare / Optum

- One of the largest US clearinghouses by transaction volume.
- Wide payer reach across Medicare, Medicaid, MA, and commercial.
- Broad capability set across X12 transactions, ERA enrollment, revenue-cycle services, and analytics.
- Now part of **Optum (UnitedHealth Group)** following the 2022 acquisition.
- Note: a high-impact cybersecurity incident in 2024 disrupted national claims flow and reshaped contingency planning across the industry. Verify current operational and security posture before architectural commitments.

> Verify current Optum/Change Healthcare product and capability mapping directly with the vendor.

### Availity

- Provider-payer collaboration network; historically founded by health plans.
- Strong payer-facing capabilities including provider portal services and real-time eligibility / claim status.
- Broad clearinghouse role; widely used by providers for eligibility and claim submission.

> Verify current Availity capabilities and payer reach directly with the vendor.

### Waystar

- US clearinghouse / revenue-cycle technology vendor.
- Capabilities across claim submission, denial management, eligibility, prior auth, patient payments, analytics.

> Verify current Waystar capabilities and payer reach directly with the vendor.

### Trizetto Provider Solutions / Cognizant

- Provider-side clearinghouse and revenue-cycle product line; part of Cognizant.
- Note: Cognizant also operates a payer-side platform (TriZetto Facets / QNXT for payers) under the same brand heritage — these are different products. Verify which is in scope.

> Verify current Trizetto Provider Solutions capabilities directly with the vendor.

### Office Ally

- US clearinghouse historically focused on smaller-practice and physician markets.
- Offers EHR, claim submission, and revenue-cycle services bundled.

> Verify current Office Ally capabilities directly with the vendor.

### Other / niche

- **Dental clearinghouses** — DentalXChange, Tesia/EDI Dental, others — for 837D and dental-payer-specific connectivity.
- **Pharmacy switches** — for NCPDP transactions, the "switch" (RelayHealth, Surescripts, others) plays a clearinghouse-equivalent role between pharmacies and PBMs.
- **State Medicaid direct** — some state Medicaid agencies accept direct connections, bypassing a commercial clearinghouse.
- **Workers' comp / auto medical** — separate clearinghouse fleets (e.g., Jopari, P2P Link) handle the non-HIPAA WC and auto medical bill flows.

> Verify niche clearinghouse selection against the specific payer mix in scope.

---

## Direct Payer Connections vs. Clearinghouse

Some payers offer **direct connections** for high-volume submitters. Tradeoffs:

| Aspect | Clearinghouse | Direct |
|---|---|---|
| Onboarding effort | Low — one connection covers many payers | High — per-payer enrollment, connectivity, testing |
| Edit consistency | Single edit layer across payers | Per-payer companion-guide handling |
| Cost | Per-transaction fee | Connection cost; may avoid per-transaction fee |
| Resilience | Single point of failure if clearinghouse degrades | Distributed risk across payers |
| Acknowledgment tracking | Clearinghouse normalizes | Per-payer formats and timelines |

Most providers use a primary clearinghouse for the bulk of payers and direct connections for a small set of high-volume payers (Medicare FFS via MAC, large state Medicaid) where the volume justifies the engineering cost.

---

## Connectivity Patterns

- **SFTP** — common for batch 837 submission, batch 835 retrieval, and high-volume file exchange.
- **AS2** — encrypted file transfer common for B2B EDI.
- **CAQH CORE Phase II Connectivity** — HTTPS-based connectivity profile mandated for some real-time HIPAA transactions (eligibility, claim status). Some clearinghouses support CORE-aligned APIs.
- **REST / SOAP APIs** — increasingly offered for real-time eligibility, claim status, and FHIR-aligned profiles.
- **Browser-based portals** — manual entry / lookup; not suitable for high-volume programmatic use but common for exception handling.

> Verify which connectivity options the clearinghouse offers for each transaction.

---

## Engineering Integration Notes

### Treat the clearinghouse as a transparent transport with edits

- Generate your **own X12 5010** internally. Do not depend on the clearinghouse to construct X12 from a proprietary format — companion-guide nuances are easier to handle when you control the source.
- Retain the **original 837** you sent. The clearinghouse may transform or re-envelope; your audit trail is your generated transaction.
- Reconcile every claim end-to-end through **TA1 → 999 → 277CA → 835**, not just at the clearinghouse front door.

### Companion-guide registry

Even with a clearinghouse, each payer has its own companion guide. The clearinghouse may abstract some details, but **payer-specific rules** (relationship codes, payer ID type qualifiers, situational segments) still leak through. Build a companion-guide registry in your system.

### Failure modes

- Clearinghouse outage — design for a secondary path (different clearinghouse or direct connection for critical payers).
- Acknowledgment latency — track stage-by-stage SLA breaches; some 277CAs lag for days for certain payers.
- Format drift — when the clearinghouse rolls out new edits, your rejection rate may shift; instrument for change detection.

### PHI and contracts

- **BAA** with the clearinghouse is required (HIPAA).
- Encryption in transit (SFTP / TLS) and at rest as specified in the BAA / contract.
- Logging and retention per your compliance program.

---

## Selection Considerations

Engineers should evaluate (in addition to the capability dimensions above):

- **Payer reach for your specific payer mix** — not total payer count.
- **Real-time eligibility SLAs** for the workflows that need real-time (registration, scheduling).
- **ERA / EFT enrollment automation** — manual per-payer enrollment is a hidden cost.
- **API surface area** — programmatic submission, status, reporting.
- **Operational resilience** — recent incident history, redundancy, restoration time.
- **Pricing model** — clearinghouse fees can be a meaningful operating cost at scale.
- **Vendor lock-in** — switching clearinghouses requires re-enrollment with payers; structure data exports and integration patterns to make switching feasible.

> Verify all of the above directly with the candidate clearinghouses and against current customer references before committing.
