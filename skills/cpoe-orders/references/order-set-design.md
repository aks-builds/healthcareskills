# Order Set Design

Order sets are the highest-leverage CDS surface in CPOE — they change ordering behavior without ever firing an alert. A well-designed order set encodes evidence-based default care into the click path. A poorly designed one bloats the chart, propagates inappropriate orders, and slows the user down.

## What an Order Set Is

A curated, named group of orders for a clinical scenario, with:

- A clear clinical context (e.g., "Adult Sepsis Bundle — ED", "Total Knee Arthroplasty — Post-op Day 0").
- Pre-selected default orders (those evidence supports defaulting on).
- Optional orders the clinician can add with one click.
- Conditional logic that shows/hides options based on patient or encounter state.
- A pedigree: owner, evidence base, last review date, retirement criterion.

## Order Set vs. Order Panel vs. Pathway

| Construct | What | When to use |
|-----------|------|-------------|
| Order set | Curated multi-section group for a scenario | Standardize complex order placement |
| Order panel | Smaller pre-built group, usually one phase | Pre-op labs, NPO at midnight + meds |
| Order pathway | Multi-day plan with sequencing and conditional logic | Knee replacement, sepsis 1-hour bundle, oncology regimen |

Pick the smallest construct that solves the problem. Use a pathway only when sequencing across days actually matters; otherwise a series of order sets at each phase is simpler to maintain.

## Anatomy of a Well-Designed Order Set

### 1. Sections

Group orders by clinical category, in the order the clinician will think about them:

- Admission orders (admit to, status, code status, isolation)
- Labs
- Imaging
- Medications
  - Routine (scheduled)
  - PRN
  - High-alert (anticoagulants, insulin, opioids) — set off visually
- Nursing
- Diet
- Activity
- Consults / referrals
- Disposition / follow-up

### 2. Defaults

For each order, decide:

- **Default on** — evidence-supported default care for this scenario.
- **Default off but visible** — appropriate in some patients; clinician chooses.
- **Conditionally visible** — only show when a condition is met.

Be ruthless. An order set with 80 items all defaulted on is a billing-fest, a payer-audit risk, and a patient-safety risk.

### 3. Conditional Logic

Use logic to hide irrelevant options rather than relying on the clinician to skip:

- Only show the vasopressor section if MAP < 65.
- Only show DVT prophylaxis options appropriate for the patient's renal function and bleeding risk.
- Only show pediatric dosing options for patients under a certain age.

Encode the logic in the order set itself, not as a separate alert. A hidden option is silent; a fired alert is fatiguing.

### 4. Propagation

When the clinician signs the order set, dependent orders should default in:

- "NPO after midnight" propagates an automatic hold on the morning insulin.
- A surgical admission order propagates the appropriate post-op protocol section.
- An ICU transfer order propagates the ICU monitoring orders.

Document the propagation rules so they are auditable and editable.

### 5. Provenance

Every order set should have a stable record of:

- Owner (a specific person, not a committee).
- Evidence base (guideline, study, internal data).
- Last reviewed date.
- Next review date.
- Retirement criterion ("retire if utilization falls below X uses per month for 3 months").

## Specific Clinical Considerations

### High-Alert Medications

High-alert classes (ISMP list — verify current version: anticoagulants, insulin, opioids, neuromuscular blockers, chemotherapy, concentrated electrolytes) warrant extra design care in any order set:

- Set off visually from routine meds.
- Default off; require deliberate selection.
- Pair with explicit dose ranges and monitoring orders.
- Include the independent-double-check workflow handle for nursing.

### Range Orders and PRN

PRN and range orders need clear indication and threshold language:

- "Acetaminophen 650 mg PO q6h PRN pain >=4/10" — bounded indication.
- "Morphine 2-4 mg IV q4h PRN pain >=7/10" — bounded range.

Joint Commission has specific standards for range-order expression and nursing selection within the range; verify against the current Joint Commission medication management chapter.

### Conditional Orders

"Hold metoprolol if SBP < 100 or HR < 55" requires the EHR to model the condition explicitly so nursing can document the trigger when the dose is withheld. Confirm that the customer's CPOE actually supports conditional orders at this fidelity — not all do.

### IV Infusions

Continuous infusions need rate and concentration, not just dose. Tie the order to the smart-pump drug library so hard/soft dose limits at the bedside match the order. Heparin nomogram and PCA orders need explicit titration tables, bounds, and re-check timing.

### Antibiotic Stewardship

Antibiotic options should reflect the institution's antimicrobial stewardship guidance. Do not hardcode specific regimens that go stale. Pull from the stewardship-curated formulary so that when guidance changes, the order set follows.

## Governance

### Authoring

- Multidisciplinary author group: physicians from the relevant specialty, pharmacy, nursing, informatics.
- Evidence review documented.
- Pilot in one unit or one service line before broad release.
- Comparison to existing order patterns (what are clinicians ordering today?).

### Review Cycle

- Annual review at minimum.
- Trigger-driven review (new guideline, new med added/removed from formulary, ISMP update, safety event involving the order set).
- Each review documented with the date, reviewer, and outcome (keep, edit, retire).

### Retirement

- Order sets that fall below utilization thresholds are retired, not left running.
- Order sets that were authored for a guideline that has been superseded are revised or retired.
- Track the retirement event so the audit trail is intact.

## Common Pitfalls

- **Order-set bloat**: defaulting on too much. The clinician unselects nothing, signs everything, and the chart fills with unneeded orders.
- **Stale evidence**: order sets that codify last decade's guideline.
- **Conflicting order sets**: two sets with overlapping defaults that produce duplicate orders.
- **Hidden propagation**: dependent orders default in without the clinician realizing.
- **No owner**: order sets without an owner go un-reviewed and out of date.
- **Hardcoded vendor-specific IDs**: flowsheet IDs, order IDs, document codes — these vary per customer and break on transfer.
- **Skipping the pilot**: deploying a new order set hospital-wide on day one with no measurement plan.

## Measurement

A healthy order-set program measures:

| Metric | Direction |
|--------|-----------|
| Utilization per order set (uses per week / month) | Stable or up where expected |
| Edit rate (how often clinicians modify the defaulted selection) | Stable or down — high edits suggest defaults are wrong |
| Outcome metrics tied to the clinical scenario (e.g., sepsis bundle compliance, post-op pain scores, time-to-antibiotic) | Up |
| Adverse events related to ordering | Down |
| Number of active order sets in the library | Pruned over time, not just growing |
- A growing library with no retirement signal is a smell. Healthy programs retire as much as they add.
