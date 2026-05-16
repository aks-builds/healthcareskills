# X12 Transaction Cheatsheet

Reference for engineers building HIPAA-mandated transaction handling. Summarizes each common 5010 transaction by purpose, who initiates, and content at a high level. Does not enumerate segment IDs or element values — those are defined in the relevant X12 Technical Report Type 3 (TR3) implementation guide and the trading partner's companion guide.

> Verify segment IDs, qualifiers, loops, and element values against the relevant **X12 5010 TR3** (e.g., 837P 005010X222A1) and the **payer/clearinghouse companion guide**. Do not invent X12 content from memory.

## Contents

- Standards posture and versions
- Transactions covered
- 270 / 271 — Eligibility
- 276 / 277 — Claim status
- 277CA — Claim acknowledgment
- 278 — Services review / Prior Authorization
- 834 — Enrollment
- 835 — Healthcare Claim Payment / Advice
- 837 — Claim submission (P / I / D)
- 820 — Premium payment
- 999 — Functional acknowledgment
- TA1 — Interchange acknowledgment
- NCPDP parallel
- Engineering notes

---

## Standards Posture and Versions

- All transactions below are HIPAA-mandated standard transactions and currently use **X12 005010** (commonly "5010") with the associated **TR3** as the implementation guide.
- NCPDP transactions (pharmacy) run in parallel and are not X12.
- HHS (HIPAA) designates the standard; trading partners must use it for the listed transaction unless an alternative is similarly designated.

> Verify the **current designated version** and any active errata against HHS / CMS HIPAA Administrative Simplification guidance.

---

## Transactions Covered

| Transaction | Purpose | Direction | Notes |
|---|---|---|---|
| **270** | Eligibility / Benefit Inquiry | Provider → Payer | Real-time + batch |
| **271** | Eligibility / Benefit Response | Payer → Provider | Pair of 270 |
| **276** | Claim Status Inquiry | Provider → Payer | Real-time + batch |
| **277** | Claim Status Response | Payer → Provider | Pair of 276 |
| **277CA** | Claim Acknowledgment | Payer/Clearinghouse → Provider | Front-end claim acceptance/rejection |
| **278** | Services Review (Prior Authorization) request / response | Provider ↔ Payer/UMO | See prior-authorization skill |
| **834** | Enrollment / Maintenance | Employer / Exchange → Payer | Member enrollment |
| **835** | Healthcare Claim Payment / Advice (ERA) | Payer → Provider | Remittance |
| **837P** | Professional claim | Provider → Payer | Maps largely to CMS-1500 |
| **837I** | Institutional claim | Provider → Payer | Maps largely to UB-04 / CMS-1450 |
| **837D** | Dental claim | Provider → Payer | Uses CDT codes |
| **820** | Premium Payment | Sponsor → Payer | Group premium remittance |
| **999** | Functional Acknowledgment | Receiver → Sender | Syntactic / structural validation |
| **TA1** | Interchange Acknowledgment | Receiver → Sender | ISA/IEA envelope validation |

---

## 270 / 271 — Eligibility

**Purpose**: Provider asks payer whether a member is eligible and what their benefits look like; payer responds.

**Who initiates**: Provider (270). Payer responds with 271.

**Content (high level)**:
- Information source (payer)
- Information receiver (provider)
- Subscriber and (optionally) dependent identifying info
- Service type code(s) for the benefit being checked
- Response includes coverage status, plan info, benefits by service type, deductible/coinsurance/copay where available

**Notes**:
- **CAQH CORE Operating Rules** mandate real-time response SLAs and connectivity profiles.
- Some benefit detail must still be obtained via supplemental real-time benefit inquiry, member portal, or phone — 271 does not always return every service-type detail.

> Verify content against TR3 005010X279A1 (or current designated version) and the payer companion guide.

---

## 276 / 277 — Claim Status

**Purpose**: Provider asks payer where a previously-submitted claim is in adjudication; payer responds.

**Who initiates**: Provider (276). Payer responds with 277.

**Content (high level)**:
- Payer claim control number, provider claim ID, member info, service dates
- Response includes status category, status code, action codes, payment info if adjudicated

**Notes**:
- The **277CA** is a separate transaction sent from the payer/clearinghouse front end after initial claim receipt and is distinct from the 277 used to respond to 276 inquiries.

> Verify against TR3 005010X212 (or current designated version).

---

## 277CA — Claim Acknowledgment

**Purpose**: Payer or clearinghouse acknowledges receipt of a claim and reports front-end accept/reject at the claim level.

**Who initiates**: Payer or clearinghouse, in response to receiving a 837.

**Content (high level)**:
- Per-claim status (accepted / rejected / pending)
- Front-end edit failures (missing data, format errors, member-not-on-file, etc.)
- Payer claim control number for accepted claims

**Engineering note**: Treat **277CA acceptance** as confirmation the payer has received and accepted the claim for adjudication — **not** that it will be paid. Final adjudication status comes on the 835.

---

## 278 — Services Review (Prior Authorization)

**Purpose**: Provider asks payer/UMO to review a planned service for authorization or certification.

**Who initiates**: Provider (278 request). Payer/UMO responds with 278 response.

**Content (high level)**:
- Requested service codes (CPT/HCPCS/NDC), units, dates, place of service
- Ordering and rendering providers
- Diagnosis
- Attachments via PWK or separate 275 / out-of-band channel

**Notes**:
- See the **prior-authorization** skill for the full PA workflow.
- **278N** (Notification) is a related variant used where notification is required rather than approval.
- The HL7 Da Vinci **PAS** IG defines a FHIR-based front end that is converted to/from X12 278 by the payer's intermediary.

> Verify against TR3 005010X217 (or current designated version) and the prior-authorization skill.

---

## 834 — Enrollment / Maintenance

**Purpose**: Employer or Exchange tells payer about member enrollment, changes, and terminations.

**Who initiates**: Employer / Exchange / TPA (acting for the plan sponsor).

**Content (high level)**:
- Member demographics and identifiers
- Plan selection, effective and termination dates
- Coverage tier (single, family, etc.) and other attributes

**Engineering note**: 834 is a **member-master feed**, not a claim. Build a slowly-changing-dimension data store and reconcile changes carefully; downstream eligibility (271) and accumulators depend on it.

> Verify against TR3 005010X220A1 (or current designated version).

---

## 835 — Healthcare Claim Payment / Advice (ERA)

**Purpose**: Payer remits payment and adjudication detail to provider.

**Who initiates**: Payer.

**Content (high level)**:
- **BPR** — payment method, amount, EFT trace
- **TRN** — reassociation trace number that matches the EFT/820 sent by the payer's bank
- **CLP** — claim-level paid amount, status, payer claim control number
- **CAS** — claim- or service-line-level adjustments: group code (CO / PR / OA / PI / CR) + CARC + amount + optional adjustment quantity
- **LQ / MIA / MOA** — remark codes (RARC) and outlier info
- **SVC** — service-line detail
- **PLB** — provider-level adjustments (recoupments, capitation, interest, refunds)

**Engineering note**: Reassociate 835 cash with the EFT/820 via the TRN reassociation trace. Posting payment from an 835 without confirming the EFT arrival leads to unposted cash.

> Verify against TR3 005010X221A1 (or current designated version) and the WPC CARC/RARC code lists.

---

## 837 — Claim Submission

Professional, institutional, and dental claim transactions:

| Variant | TR3 (verify current) | Maps to | Code sets |
|---|---|---|---|
| **837P** Professional | 005010X222A1 | CMS-1500 | CPT/HCPCS + ICD-10-CM + modifiers |
| **837I** Institutional | 005010X223A2 | UB-04 / CMS-1450 | CPT/HCPCS + ICD-10-CM + ICD-10-PCS (inpatient) + revenue codes + bill type |
| **837D** Dental | 005010X224A2 | ADA claim form | CDT |

**Who initiates**: Provider (or biller / clearinghouse on behalf of provider).

**Content (high level)**:
- Billing provider, pay-to provider, rendering provider, referring provider
- Subscriber + patient demographics
- Service lines with codes, modifiers, units, charge amounts, place of service, dates of service
- Diagnosis pointers from service lines to claim-level diagnoses
- Coordination of benefits (COB) info from prior payers in Loop 2320/2330 for secondary/tertiary claims
- Attachments via PWK control numbers

**Engineering note**: Persist the **raw 837 you sent** along with every TA1 / 999 / 277CA / 835 received. This is the audit trail.

> Verify against the relevant 5010 TR3 and the payer companion guide.

---

## 820 — Premium Payment

**Purpose**: Sponsor (employer / TPA) pays premiums to payer.

**Note**: Distinct from the **820 CCD+** transaction used by banks for EFT remittance reassociation with the healthcare 835. The financial 820 from the bank and the HIPAA premium 820 are different transactions in different contexts.

> Verify against TR3 005010X218 (or current designated version).

---

## 999 — Functional Acknowledgment

**Purpose**: Receiver acknowledges receipt and syntactic/structural validation of a functional group.

**Note**: 999 **replaced 997** for HIPAA-mandated transactions. A 999 acceptance confirms structure only — **not** business acceptance. Wait for 277CA + 835 before treating a claim as paid.

> Verify against TR3 005010X231A1 (or current designated version).

---

## TA1 — Interchange Acknowledgment

**Purpose**: Receiver acknowledges the ISA/IEA envelope only (e.g., bad interchange control number, bad sender/receiver IDs).

**Note**: TA1 happens before 999. A TA1 reject means the entire interchange was not processed; 999 happens after the envelope passes.

---

## NCPDP Parallel

Pharmacy claims do not use X12. They use **NCPDP** standards:

- **NCPDP Telecommunication Standard D.0** — real-time pharmacy claim and reversal (the dominant retail/specialty Rx claim transaction)
- **NCPDP Batch Standard 1.2** — batch pharmacy
- **NCPDP SCRIPT** — e-prescribing (medication orders), including SCRIPT ePA for prior auth
- **NCPDP Formulary and Benefit (F&B)** — formulary publication

NCPDP-to-X12 mappings are imperfect; when loading pharmacy data into a unified claims warehouse, preserve original NCPDP fields rather than only the X12-equivalent translation.

> Verify NCPDP version against the current HIPAA-designated standard and the PBM trading partner agreement.

---

## Engineering Notes

### Envelope control

- ISA / GS / ST control numbers must increment and reconcile between sender and receiver.
- IEA / GE / SE reflect counts and must match the corresponding ISA / GS / ST.
- TA1, 999, 277CA, and 835 all reference back via envelope or claim-level identifiers.

### Acknowledgment ladder

A correctly-handled inbound transaction passes through (at minimum):
- TA1 — envelope ok
- 999 — structure ok
- 277CA — claim accepted at the front end (for 837)
- 835 — adjudication and payment (for 837)

Treat each step as a separate state; expose per-stage rejection rates as a KPI.

### Idempotency

Your unique submission ID should round-trip via the claim-level identifier (CLM01 in 837 / payer claim control number returned in 277CA and 835) so 835s can be matched back to the original claim.

### Versioning

Maintain separate engines per 5010 sub-version (e.g., 005010X222A1 for 837P, 005010X223A2 for 837I) and per NCPDP version. Do not silently downgrade or upgrade — companion guides differ.

### Audit retention

Retain the original wire-format transactions (not just parsed data) for the retention period required by HIPAA, state law, payer contracts, and your compliance program. Storage compression and immutable object stores are common patterns.

> Verify retention requirements against your compliance program and applicable state/federal law.
