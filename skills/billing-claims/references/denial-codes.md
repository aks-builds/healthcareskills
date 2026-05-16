# Denial Codes — CARC, RARC, Group Codes

Reference for engineers building denial classification and workqueue routing. Describes the WPC-published CARC and RARC code lists at a structural level, the group codes used on the 835 CAS segment, and common denial categories engineers must support. Does not enumerate specific CARC/RARC numeric values — those are maintained by the Washington Publishing Company (WPC) and update on a published schedule.

> Verify against the **WPC current CARC and RARC code lists** (https://x12.org/codes — WPC code lists are republished on a regular cadence). Do not hard-code specific CARC/RARC numbers from memory.

## Contents

- Group codes (CO, PR, OA, PI, CR)
- CARC — Claim Adjustment Reason Codes
- RARC — Remittance Advice Remark Codes
- Common denial categories and recommended workqueue routing
- Where these codes appear in the 835
- Engineering notes
- Pitfalls

---

## Group Codes

The **group code** sits on the 835 **CAS** segment alongside the CARC. It identifies who is responsible for the adjusted amount.

> Verify the active group code set and any additions/retirements against the WPC current code list.

| Group code | Meaning |
|---|---|
| **CO** | **Contractual Obligation** — provider write-off per payer contract / fee schedule. Not patient liability. |
| **PR** | **Patient Responsibility** — deductible, copay, coinsurance, non-covered services owed by the patient. |
| **OA** | **Other Adjustment** — used when none of the other group codes apply. |
| **PI** | **Payer Initiated Reductions** — payer-initiated, may be appealable. |
| **CR** | **Corrections and Reversals** — typically paired with reversal of a previously-paid amount. Frequent on take-backs and reissues. |

### Why this matters

- **PR** balances roll to patient statements.
- **CO** balances are write-offs and never billed to the patient.
- **OA** / **PI** need investigation to decide whether to appeal or accept.
- **CR** signals a reversal — your cash posting must subtract the prior payment, not double-post.

---

## CARC — Claim Adjustment Reason Codes

CARCs explain **why** an adjustment was made. They are numeric values published by WPC on a regular schedule. Each CARC has:

- A **definition** (human-readable description)
- **Effective** and (optionally) **deactivation** dates
- An indication of whether it is **active** or **deactivated**

Examples of **categories of CARCs** that engineers must support (each category includes many specific CARC numbers — verify current values from WPC):

- **Deductible / coinsurance / copay** — patient-responsibility shifts
- **Non-covered service** — service not a benefit
- **Bundling / inclusive service** — service is included in another paid service
- **Authorization / referral required** — PA missing, expired, or invalid
- **Coverage / eligibility issues** — member not on file, term date, plan exhausted
- **Coordination of benefits** — primary payer issue
- **Timely filing** — claim received after filing deadline
- **Duplicate** — service treated as duplicate of a paid claim
- **Service not consistent with provider type** — credentialing or scope mismatch
- **Documentation / clinical review required**

> Verify specific CARC numbers, definitions, and active status against the WPC current code list. Hard-coding numbers from memory will silently rot.

---

## RARC — Remittance Advice Remark Codes

RARCs provide **additional explanation** alongside CARCs. There are two types:

- **Supplemental** — adds detail to a CARC (e.g., "the procedure code is inconsistent with the provider type/specialty").
- **Informational** — informational only (typically begin with `N` or `M` prefixes — verify current convention).

RARCs alone are usually not actionable without their parent CARC. Build the denial classifier on the **combination** of group code + CARC + RARC + payer-specific context.

> Verify current RARC list and prefix conventions against the WPC code list.

---

## Common Denial Categories and Workqueue Routing

Engineers building denial-management systems should route denials into actionable buckets. The exact CARC/RARC mappings will vary by payer and by year — do not hard-code mappings from memory. The category structure below is durable:

### Eligibility / coverage

- **Signals**: PR or CO group code with eligibility-themed CARC; member not on file; coverage terminated; plan exhausted
- **Action**: Front-end fix at registration / eligibility verification. Confirms whether member has secondary coverage to bill instead.
- **Owner**: Patient access / front office
- **Reference**: `billing-claims` eligibility workflow (270/271)

### Authorization / referral

- **Signals**: PA-themed CARC, often with RARC indicating PA number missing, expired, or mismatched to service
- **Action**: Check whether PA was obtained; check authorization-to-claim alignment (codes, units, dates, providers, place of service); appeal or retroactive PA where possible
- **Owner**: PA team
- **Reference**: `prior-authorization`

### Medical necessity

- **Signals**: CARC/RARC referencing LCD/NCD or payer medical policy; clinical-review denial
- **Action**: Pull clinical documentation; submit appeal with supporting evidence; consider peer-to-peer
- **Owner**: Clinical appeals team / case management
- **Reference**: `medical-coding` for LCD/NCD coding alignment

### Coding

- **Signals**: invalid code; bundling (NCCI); missing or incorrect modifier; code not consistent with provider type; ICD-CPT mismatch
- **Action**: Coder workup; corrected claim with corrected codes/modifiers; refer for documentation improvement if pattern repeats
- **Owner**: Coding team
- **Reference**: `medical-coding`

### Timely filing

- **Signals**: timely-filing-themed CARC
- **Action**: Often non-appealable. Check whether payer received an earlier submission attempt; provide proof of timely submission if available.
- **Owner**: Billing follow-up
- **Reference**: Track filing deadlines per payer in companion-guide registry.

### Duplicate

- **Signals**: duplicate-themed CARC
- **Action**: Determine whether truly duplicate (re-submission of the same claim) or whether payer is incorrectly treating two distinct services as duplicate (e.g., bilateral procedure billed on two lines without proper modifiers).
- **Owner**: Billing follow-up
- **Reference**: Modifier alignment may need `medical-coding`

### Coordination of benefits (COB)

- **Signals**: COB-themed CARC; missing primary EOB; primary not on file
- **Action**: Verify payer order; submit secondary with primary 835 / EOB attached; correct COB sequence at registration
- **Owner**: Patient access + billing
- **Reference**: `billing-claims` COB / MSP section

### Contract / fee schedule

- **Signals**: CO group code with fee-schedule reduction matching contracted rate (this is expected); CO mismatch where the reduction exceeds expected contract terms is a contract-loading issue
- **Action**: Reconcile expected vs. actual paid amount against contracted fee schedule; escalate contract-loading defects to payer
- **Owner**: Underpayment / payer-relations

### Provider / credentialing

- **Signals**: provider-not-on-file; out-of-network when expected in-network; taxonomy / specialty mismatch
- **Action**: Verify credentialing status; correct provider data on claim
- **Owner**: Credentialing + billing

### Documentation / clinical-review pend

- **Signals**: claim pended for additional records
- **Action**: Pull and submit requested documentation within payer-specific deadline; track aging
- **Owner**: HIM / medical records

---

## Where These Codes Appear in the 835

- **CAS** segments at the **claim level** and **service-line level** carry the group code + CARC + amount, with optional adjustment quantity.
- **LQ / MIA / MOA** segments carry RARCs and outlier info.
- A single claim line can have **multiple CAS adjustments** (e.g., a deductible PR adjustment + a contractual CO adjustment on the same line).
- **PLB** (Provider-Level adjustments) carry recoupments, capitation amounts, interest, and refunds at the provider level — these do not appear as CAS on individual claims.

> Verify CAS segment placement and PLB reason codes against TR3 005010X221A1 (or current designated 835 version).

---

## Engineering Notes

### Don't hard-code CARC/RARC numbers

The WPC code lists update on a regular cadence. Hard-coded mappings rot silently. Instead:

1. Load the **current WPC CARC and RARC code lists** as reference data.
2. Refresh on the published update schedule.
3. Build the denial classifier with a rules layer that references the loaded lists by code value, with effective-date awareness.
4. Capture each denial with: group code, CARC list, RARC list, payer, line-specific signals (POS, modifier, service line), and route to a workqueue using rules rather than scattered if/else logic.

### Track aging and KPIs

Core KPIs for the denial-management system:

- **First-pass yield** — percent of claims paid on first submission
- **Denial rate** — denials per claims submitted
- **Net collection rate** — collected vs. allowed
- **Days in A/R**
- **Denial overturn rate** — percent of appealed denials reversed
- **Aging** by bucket (0-30 / 31-60 / 61-90 / 90+) by category

### Audit trail

Every denial / appeal action must be timestamped, attributed to an actor, and linked back to the originating 835 segment. Required for payer disputes and internal QA.

### Multi-CAS handling

A single 835 line can carry multiple CAS adjustments. Cash posting must sum them correctly (CO + PR + OA + PI + CR group code amounts plus paid amount should reconcile to billed amount, subject to PLB and overpayment-recovery logic).

### CR group code is a reversal trap

A `CR` group code typically pairs with the reversal of a prior payment. Cash posting must:

1. Identify the prior posted payment that is being reversed.
2. Subtract the prior payment from cash.
3. Apply any new corrected payment on the same 835 (if present) as a separate posting.

Treating CR as an additional adjustment without reversing the prior payment leads to double-counted cash.

---

## Pitfalls

- Treating a 999 acceptance as "claim accepted" — it only confirms syntactic structure. Wait for 277CA + 835.
- Hard-coding CARC/RARC numbers and mappings rather than refreshing from WPC.
- Ignoring **provider-level adjustments (PLB)** — these can offset claim-level payments and corrupt reconciliation.
- Routing all denials to a single workqueue; the categories above have different owners and SLAs.
- Treating PR group code amounts as collectible without checking secondary coverage that should be billed first.
- Skipping the audit-trail link from each denial action back to the underlying 835 segment.

> Verify CARC, RARC, and group-code current values and definitions against the WPC code list.
