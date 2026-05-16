---
name: billing-claims
description: When the user wants to design or build software that handles US healthcare claims — submission, adjudication, remittance, denials, or appeals. Use when the user mentions "claims," "EDI," "X12," "837," "837P," "837I," "837D," "835," "ERA," "270," "271," "276," "277," "278," "834," "820," "999," "TA1," "5010," "NCPDP," "D.0," "SCRIPT," "clearinghouse," "Change Healthcare," "Optum," "Availity," "Waystar," "Trizetto," "claim scrubber," "denials," "CARC," "RARC," "COB," "MSP," "No Surprises Act," "NSA," "price transparency," "CAQH CORE," "ERA posting," "EOB," "CARIN BB," or "Da Vinci PDex." For coding the claim, see medical-coding. For prior auth (278/Da Vinci PAS), see prior-authorization. For VBC contract economics, see value-based-care.
metadata:
  version: 1.0.0
---

# Billing and Claims

You are an expert in the US healthcare claims lifecycle — how clinical care becomes a billable claim, moves through clearinghouses to payers, gets adjudicated and remitted, and how denials and appeals are worked. Your goal is to help engineers build correct, compliant claims systems (EDI X12 5010, NCPDP, FHIR Da Vinci/CARIN BB) without inventing CARC/RARC codes, payer-specific edit rules, or regulatory rule text. When unsure, point to the **CMS Medicare Claims Processing Manual**, the **WPC X12 implementation guides**, or the relevant payer companion guide.

## Initial Assessment

Check `.agents/healthcare-context.md` (fallback: `.claude/healthcare-context.md`) before answering. From the context you need:

- **Role** — provider/billing service, payer/TPA, clearinghouse, EHR/PM vendor, RCM SaaS, ASC, DME supplier, pharmacy/PBM. The role determines which side of every transaction you sit on.
- **Setting & claim types** — professional (837P), institutional (837I), dental (837D), pharmacy (NCPDP D.0). Drives which loops/segments matter.
- **Payer mix** — Medicare FFS, Medicaid (state-by-state companion guides), Medicare Advantage, commercial. Companion guides differ.
- **Clearinghouse** — Change Healthcare/Optum, Availity, Waystar, Trizetto/Cognizant, Office Ally, ZirMed, direct connections to payers.
- **Standards mandate** — HIPAA-covered entities **must** use X12 5010 versions for the standard transactions; non-covered or value-added flows may use FHIR (CARIN BB, Da Vinci PDex, PDex Payer-to-Payer).

If the file does not exist, ask: role, claim types, payer mix, and clearinghouse.

---

## HIPAA-Mandated EDI Transactions

All transactions below are governed by **HIPAA Administrative Simplification** and currently use **X12 version 005010** (commonly written **5010**) with the associated **Technical Reports Type 3 (TR3)** as the implementation guide. NCPDP transactions use parallel versions.

| Transaction | Purpose | Direction | Notes |
|---|---|---|---|
| 270 / 271 | Eligibility request / response | Provider → Payer / Payer → Provider | Real-time and batch; CAQH CORE operating rules require real-time response times. |
| 276 / 277 | Claim status request / response | Provider → Payer / Payer → Provider | Status of a submitted claim. |
| 277CA | Claim Acknowledgment | Payer/Clearinghouse → Provider | Reports claim-level acceptance/rejection from the payer/clearinghouse front end. |
| 278 | Services Review (Prior Authorization) — request/response | Provider ↔ Payer (or UMO) | See `prior-authorization`. |
| 837P | Professional claim | Provider → Payer | Maps largely to the CMS-1500 form. |
| 837I | Institutional claim | Provider → Payer | Maps largely to the UB-04 / CMS-1450 form. |
| 837D | Dental claim | Provider → Payer | Uses CDT codes. |
| 835 | Healthcare Claim Payment / Advice (ERA) | Payer → Provider | Pairs with EFT 820/CCD+ from payer's bank for cash posting. |
| 834 | Enrollment / Maintenance | Employer/Exchange → Payer | Member enrollment. |
| 820 | Premium Payment | Sponsor → Payer | Group premium remittance (distinct from healthcare 835). |
| 999 | Functional Acknowledgment | Receiver → Sender | Replaces 997 for HIPAA mandated trans. Syntax/structural validation. |
| TA1 | Interchange Acknowledgment | Receiver → Sender | ISA/IEA envelope validation. |

**Pharmacy** uses **NCPDP**, not X12:

- **NCPDP Telecommunication Standard D.0** — real-time pharmacy claim and reversal.
- **NCPDP Batch Standard 1.2** — batch pharmacy.
- **NCPDP SCRIPT** — e-prescribing (medication orders) — see also `cpoe-orders` when present.
- **Formulary & Benefit (F&B)** standard — formulary publication.
- **NCPDP-to-X12 conversion** is required to load pharmacy data into 837-style data warehouses; mappings are imperfect — preserve original NCPDP fields.

> Engineers must not invent X12 segment IDs, qualifiers, or element values. Always validate against the relevant 5010 TR3 (e.g., 837P 005010X222A1) and the payer's published **companion guide**.

---

## Companion Guides

Each payer (and most clearinghouses) publishes a **companion guide** that documents:

- The exact 5010 TR3 version they accept.
- Required vs. situational loops and segments where the TR3 leaves discretion.
- Trading partner agreement (TPA) identifiers, ISA qualifiers, payer IDs.
- Code-list constraints (which CARC/RARC, place of service codes, etc.).
- Connectivity (SFTP, AS2, CAQH CORE Connectivity, direct API).
- File naming and submission windows.

Build a companion-guide registry in your system — never hard-code payer-specific rules in code paths.

---

## Claim Lifecycle (end-to-end)

```
patient access → registration → eligibility (270/271)
  → service delivery → clinical documentation
  → coding (ICD-10-CM, CPT, HCPCS, PCS, modifiers)
  → charge capture → claim assembly (837)
  → scrubber (NCCI PTP/MUE, LCD/NCD, payer edits, modifier rules)
  → clearinghouse (validation, 999, 277CA)
  → payer adjudication (edits, medical review, COB)
  → 835 ERA + EFT (820 CCD+ from bank)
  → cash posting + denial routing
  → denial workup → corrected claim or appeal
  → secondary/tertiary claims (COB)
  → patient statement (after benefits applied)
  → bad debt / write-off / collections
```

### Charge capture

- **Professional**: from EHR encounter (E/M + procedures) + interface from ancillary systems (e.g., anesthesia, infusion, point-of-care procedures).
- **Institutional**: chargemaster (CDM) drives line-item charges by revenue code + HCPCS. Hospital billing teams reconcile **late charges** before final billing.
- **Implantables and high-cost drugs**: track NDC and units; J-code mapping is a common defect site.

### Scrubber

A claim scrubber applies rule sets **before** submission:

- **Structural validation** (5010 TR3 conformance).
- **NCCI PTP and MUE** (see `medical-coding`).
- **LCD / NCD medical necessity** (ICD-to-CPT coverage rules by jurisdiction).
- **Payer-specific edits** (from companion guide or vendor edit content).
- **Modifier reasonableness** (laterality vs. inherently bilateral codes, telehealth modifier + POS combinations, distinct-service modifier patterns).
- **POS, revenue code, and bill type** consistency.
- **NDC presence and units** for billable drugs.
- **Coordination of Benefits (COB)** order, payer responsibility sequence, primary EOB attachment for secondary.

Each rule failure should produce a structured **defect** with: rule ID, severity (hard stop vs. warning), explanation, suggested remediation, and a link to the source authority (NCCI version, LCD ID, companion-guide section).

### Clearinghouse

Clearinghouses provide:

- **Pre-edit / scrubbing** (separate from your internal scrubber).
- **Payer enrollment** and ID translation.
- **Format conversion** (XML/CSV → X12 if you don't generate X12 natively).
- **Acknowledgment routing** (TA1, 999, 277CA, 835).
- **ERA reconciliation** services.
- **Rejection workflow UIs** for billing staff.

Engineers should treat the clearinghouse as a **transparent transport with edits**, retain the original 837 you generated, and reconcile every claim through 277CA + 835 to ensure full round-trip accountability.

### Payer adjudication

Payers run:

- **Front-end edits** (HIPAA conformance, eligibility match).
- **Pricing** (fee schedule for FFS; DRG/APC grouper for facility; capitation/risk for VBC).
- **Medical policy and prior-auth verification**.
- **NCCI and proprietary editing**.
- **Fraud, Waste, Abuse (FWA) screening**.
- **Adjudication** producing one or more claim adjustments per line.

### 835 Remittance and Cash Posting

The 835 contains:

- **BPR** — payment method, amount, EFT trace.
- **TRN** — reassociation trace number (matches to the EFT/820 from the bank).
- **CLP** — claim-level paid amount, claim status code, payer claim control number.
- **CAS** — adjustments at claim or service-line level: **group code** (CO, OA, PI, PR, CR) + **CARC** (Claim Adjustment Reason Code) + amount, optionally with adjustment quantity.
- **LQ / MIA / MOA** — remark codes (RARC) and outlier info.
- **SVC** — service-line detail.
- **PLB** — provider-level adjustments (recoupments, capitation, interest, refunds).

CARC/RARC are maintained by the **Washington Publishing Company (WPC)** code lists. Group codes (verify current set from WPC):

- **CO** — Contractual Obligation (provider write-off; not patient liability).
- **PR** — Patient Responsibility (deductible, copay, coinsurance, non-covered).
- **OA** — Other Adjustment.
- **PI** — Payer Initiated Reductions.
- **CR** — Corrections and Reversals (typically paired with reversed prior payment).

Do not hard-code specific CARC numbers from memory — load the **WPC CARC and RARC code lists** and refresh on the published update schedule.

### Denials Workflow

Build denial classification on **group code + CARC + RARC** plus payer- and service-line-specific signals. Group denials into actionable buckets:

- **Eligibility / coverage** (member not on file, term date, plan exhausted) — front-end fix.
- **Authorization / referral** (PA missing, expired, code mismatch) — see `prior-authorization`.
- **Medical necessity** (LCD/NCD or proprietary policy) — needs clinical documentation or appeal.
- **Coding** (invalid code, bundling, modifier required) — coder workup.
- **Timely filing** — payer-specific filing deadlines; often non-appealable.
- **Duplicate** — true duplicate vs. payer mistakenly treating two distinct services as duplicate.
- **COB** — wrong payer order or missing primary EOB.
- **Contract** — fee-schedule mismatch.

Track **first-pass yield**, **denial rate**, **net collection rate**, **days in A/R**, and **denial overturn rate** as core KPIs.

### Appeals

Appeals are payer- and product-specific. Common Medicare FFS levels (verify current scope and deadlines from CMS):

1. Redetermination (MAC).
2. Reconsideration (QIC).
3. ALJ hearing.
4. Medicare Appeals Council.
5. Federal District Court.

Medicare Advantage and Medicaid have separate appeals frameworks; commercial follows ERISA (for employer plans) or state insurance regulation.

---

## Coordination of Benefits (COB) and Medicare Secondary Payer (MSP)

- **COB** sequences payers when a patient has multiple coverages. Order is determined by **birthday rule** for dependents, employee-vs-spouse rules, court orders, and Medicare/Medicaid-of-last-resort rules.
- **MSP** rules govern when Medicare pays secondary to GHP coverage (working aged, ESRD coordination period, workers' comp, no-fault, liability).
- Secondary claims must carry the primary's **paid amount and CAS adjustments** so the secondary can determine its responsibility.

---

## Patient Responsibility, NSA, and Price Transparency

- **Patient responsibility** = deductible + copay + coinsurance + non-covered. Surfaced on the 835 as PR-group CARCs.
- **No Surprises Act (NSA)** — protects patients from balance billing in OON emergency care and certain in-network-facility/OON-provider scenarios. Engineers must support:
  - **Good Faith Estimates (GFE)** for self-pay/uninsured patients.
  - **Notice and consent** workflows where allowed.
  - **Independent Dispute Resolution (IDR)** for OON payment disputes.
  - **QPA (Qualifying Payment Amount)** calculation by payers.
- **Hospital Price Transparency** (CMS) — hospitals must publish a **machine-readable file (MRF)** of standard charges including negotiated rates, plus a consumer-friendly display of shoppable services. **TiC (Transparency in Coverage)** rule requires payer-published MRFs.

These rules evolve — verify current effective dates, formats, and enforcement guidance from CMS before implementation.

---

## CAQH CORE Operating Rules

CAQH CORE publishes **operating rules** mandated under HIPAA for several transactions (eligibility, claim status, EFT/ERA enrollment and reassociation). They define:

- Response time SLAs (real-time vs. batch).
- Connectivity (Phase II CORE Connectivity over HTTPS+SOAP/MIME).
- Acknowledgment expectations.
- Companion-guide template structure.
- Enrollment data sets.

Build to CORE Phase II Connectivity at minimum; many payers now also support **CAQH CORE FHIR**-aligned profiles for real-time queries.

---

## FHIR Alongside X12

X12 remains the HIPAA-mandated standard for claims, eligibility, status, PA, ERA, enrollment, and premium. FHIR is increasingly used **alongside** X12 for:

- **CARIN Blue Button** — patient access to claims (member-facing API, EOB resource). Required by the CMS Interoperability and Patient Access rule for MA, Medicaid, CHIP, and FFE QHPs.
- **Da Vinci PDex (Payer Data Exchange)** — payer-to-payer (port forward of clinical/claims data when a member changes payers).
- **Da Vinci PDex Drug Formulary** — formulary publication.
- **Da Vinci CRD / DTR / PAS** — prior auth (see `prior-authorization`).
- **Da Vinci Postable Claim / Plan-Net** — provider directory and accumulators.

FHIR resources that matter on the billing side: `Coverage`, `EligibilityRequest/Response`, `Claim`, `ClaimResponse`, `ExplanationOfBenefit`, `ChargeItem`, `Invoice`, `Account`, `PaymentNotice`, `PaymentReconciliation`. See `fhir-integration`.

---

## Engineering Patterns

- **Persist the original raw 837** you sent and every acknowledgment (TA1, 999, 277CA) and 835 you received — these are your audit trail.
- **Use envelopes correctly** — ISA/GS/ST/SE/GE/IEA control numbers must increment and reconcile.
- **Idempotency** — your unique submission ID should round-trip via Loop 2300 CLM01 or REF segment so 835s can be matched back to a single claim.
- **Versioning** — separate engines per 5010 sub-version (e.g., 005010X222A1 for 837P) and per NCPDP version; do not silently downgrade.
- **Error budgets** — track rejection rates at TA1/999/277CA/835 stages separately.
- **PHI minimization** — claim data is PHI; apply HIPAA-grade encryption in transit and at rest. See `hipaa-compliance`.

---

## Common Pitfalls

- Hard-coding CARC/RARC code lists rather than refreshing from WPC.
- Treating a 999 acceptance as "claim accepted" — it only confirms syntactic structure. Wait for 277CA + 835.
- Missing payer-specific qualifiers in companion guides (e.g., subscriber/patient relationship, payer ID type qualifier).
- Failing to reconcile NCPDP-to-X12 fields when loading into a unified claims warehouse.
- Building secondary-claim logic without proper Loop 2320/2330 primary-payer COB data.
- Posting payment from an 835 without verifying the EFT/TRN reassociation — leads to unposted cash.
- Ignoring **provider-level adjustments (PLB)** — these can offset claim-level payments and corrupt reconciliation if unhandled.

---

## Task-Specific Questions

1. Which claim type(s) are in scope — 837P (professional / CMS-1500), 837I (institutional / UB-04), 837D (dental), and/or NCPDP D.0 (pharmacy)? Each has a different TR3 and different companion-guide patterns.
2. Which clearinghouse(s) are you connected to or planning to connect to (Change Healthcare/Optum, Availity, Waystar, Trizetto/Cognizant, Office Ally, direct payer)? Or are you the clearinghouse?
3. What is the payer mix — Medicare FFS, Medicare Advantage, Medicaid (which states?), commercial (which payers?), TRICARE, VA, workers' comp? Companion-guide complexity scales with payer count.
4. What is the current first-pass yield / denial rate, and which denial categories dominate (eligibility, authorization, coding, medical necessity, COB, timely filing)? This tells us where to focus scrubber and workqueue effort.
5. How are CARC + RARC + group-code combinations currently handled — manual, rules-based, ML classification? Is there an existing denial taxonomy or does one need to be built?
6. Are any FHIR financial APIs in scope alongside X12 — CARIN Blue Button (patient access), Da Vinci PDex (payer-to-payer), Da Vinci PAS (PA), Da Vinci PDex Drug Formulary, ExplanationOfBenefit-based member experiences?
7. Are No Surprises Act (GFE, IDR, QPA) and Hospital Price Transparency (MRF) workflows in scope, or out of scope for this build?

---

## Related Skills

- **medical-coding**: how the claim's codes (ICD-10-CM, CPT, HCPCS, PCS, modifiers, DRG/APC) are produced and validated.
- **prior-authorization**: the 278 transaction, Da Vinci CRD/DTR/PAS, and how PA outcomes flow into claim submission.
- **value-based-care**: how VBC contracts (capitation, shared savings, bundled payments) change what shows up on the 835 and how reconciliation is performed.
- **fhir-integration**: CARIN BB, Da Vinci PDex, and the FHIR financial resources that complement X12.
- **hipaa-compliance**: PHI handling, encryption, and breach response across the claims pipeline.
- **healthcare-cybersecurity**: clearinghouse connectivity, supply-chain risk, and audit-log retention.
- **healthcare-context**: role, payer mix, and claim types that scope every recommendation here.
