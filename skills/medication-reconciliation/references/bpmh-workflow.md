# Best Possible Medication History (BPMH) Workflow

The BPMH is the gold-standard medication history — what the patient is **actually** taking, not just what is in the chart. A high-quality BPMH at admission is the single most important input to safe inpatient prescribing and to accurate discharge med reconciliation.

This reference describes the workflow, the sources, the typical pitfalls, and what a software tool should do to support (not replace) the BPMH interview.

## The Workflow

### Step 1 — Identify the Patient

Two identifiers (name + DOB, MRN, etc.) per the institution's identity policy. Confirm before any med history work.

### Step 2 — Interview the Patient or Caregiver

A structured interview is the irreducible core of the BPMH. Software can pre-populate inputs but cannot replace this conversation.

Probe systematically:

- **Prescription medications.** Name, strength, dose, frequency, route, indication, prescriber.
- **Over-the-counter medications.** Aspirin, NSAIDs, antihistamines, sleep aids, antacids, laxatives.
- **Herbals and supplements.** Vitamins, fish oil, glucosamine, melatonin, St. John's wort, ginkgo, garlic, ginseng — all of these can interact with prescription drugs.
- **PRN medications.** When do they **actually** take them? "Once a month" vs "every day" radically changes risk.
- **Recently stopped medications.** What was stopped, when, and why? Withdrawal risk for some drug classes.
- **Sample medications** from the office. Often missed.
- **Adherence.** Open-ended: "How do you actually take this?" — not "Do you take this every day?"
- **Patient-applied medications.** Eye drops, topicals, transdermal patches, inhalers, injectables.
- **Specialty pharmacy and infusion meds.** Biologics, oncology, IVIG — often dispensed monthly, easily forgotten in the rush.
- **Recent antibiotic or steroid course.** May have ended last week and still be relevant.

### Step 3 — Cross-Reference Against External Sources

Confirm the patient-reported list against at least one additional source:

- The bag-of-pills (patient brings actual bottles).
- Surescripts medication history (1-2 years of fill events across most US pharmacies and many PBMs).
- Prior EHR record (same institution).
- HIE record (other institutions).
- Outpatient provider's most recent note or active med list.
- Long-term-care facility's MAR if the patient comes from a SNF.
- The patient's outpatient pharmacy (phone call if needed).

The cross-reference is where most discrepancies surface. Fills do not prove the patient took the drug; absence of fills does not prove they did not (cash-pay scripts, samples, OTC, or fills outside the network are not in Surescripts).

### Step 4 — Resolve Discrepancies

For each discrepancy between sources:

- Ask the patient or caregiver to clarify.
- Contact the prescribing physician or outpatient pharmacy if needed.
- Document the resolution (which source was correct, who confirmed).

### Step 5 — Document the BPMH

The BPMH is documented **distinctly from** the inpatient orders. In FHIR, the BPMH is a list of `MedicationStatement` resources; the inpatient orders are `MedicationRequest` resources. Reconciliation is the comparison between the two.

Each BPMH entry should carry:

- The drug at SCD or SBD level (RxCUI).
- Dose, frequency, route.
- Status (active, recently stopped, held).
- Source(s) and confidence.
- Provenance (who interviewed, when, against which sources).

## Time Budget

A high-quality BPMH for a complex patient with 15+ medications typically takes **20-40 minutes**. Tools that pre-populate from data sources can shorten the data-gathering portion, but the interview is non-negotiable.

Pharmacist-led BPMH consistently outperforms provider-led or nurse-led BPMH in published literature. Staffing the BPMH with pharmacists is one of the highest-value safety investments a hospital can make.

## Sources — Strengths and Limitations

| Source | Strength | Limitation |
|--------|----------|------------|
| Patient / family interview | Captures actual taking; OTC and herbals; adherence | Recall bias; language barriers; cognitive impairment; reluctance to admit non-adherence |
| Pill bottles / bag | Confirms strengths and prescribers | Misses what's not in the bag (intentional or oversight) |
| Surescripts Medication History | Broad pharmacy coverage; cross-network | Cash-pay scripts may be missing; does not prove taking |
| Pharmacy fill (single chain) | Detailed for that chain | Misses fills elsewhere |
| Prior EHR / HIE | Active outpatient med list | May be stale; quality varies |
| Long-term-care MAR | Ground truth for SNF residents | Hard to obtain quickly, especially after-hours |
| Insurance / PBM claims | Wide coverage | Lag of days to weeks; cash pay missed |

> Verify Surescripts service definitions and access requirements against current Surescripts documentation. Service terms and PBM coverage evolve.

## How a Tool Should Help

A med rec software tool should:

- **Pre-populate** the BPMH worklist from Surescripts + prior EHR + HIE.
- **Normalize** all inputs to RxCUI at SCD/SBD level so brand vs generic, repackaged NDCs, and synonyms collapse correctly.
- **Preserve provenance** — show which source(s) contributed each entry.
- **Highlight uncertainty** — entries from one source only, conflicting strengths, recent-fills-without-active-orders.
- **Surface OTC and supplement prompts** so the interviewer remembers to ask.
- **Capture adherence** as a structured field, not just free text.
- **Record interview metadata** — who, when, language used, interpreter used.
- **Separate the BPMH list from the inpatient orders** — never auto-merge.
- **Track changes** — when the interview reveals something different from a source, log both.

A tool should **not**:

- Auto-reconcile and present a finished list as if the interview happened.
- Treat a Surescripts fill as proof the patient is taking the drug.
- Collapse different doses or routes of the same drug into one entry.
- Hide low-confidence entries — they need clinician attention.

## Output of the BPMH

The BPMH feeds three downstream activities:

### 1. Admission Reconciliation

Compare BPMH (`MedicationStatement`) to admission orders (`MedicationRequest`) and classify each line as:

- Continue (same drug, dose, frequency, route)
- Modify (same drug, different dose / frequency / route — with reason)
- Discontinue (in BPMH, not ordered — with reason)
- New (ordered, not in BPMH — with reason)
- Hold (in BPMH, intentionally held during stay — with reason)

Each change requires a documented reason and prescriber attestation.

### 2. Inpatient Decision Support

DDI, dose, duplicate, allergy, and renal/hepatic checks should run against the **post-reconciliation** list (active inpatient orders plus continued home meds). Running checks against the pre-rec list produces noise.

### 3. Discharge Reconciliation

At discharge, compare inpatient orders to the planned discharge list. The discharge med list goes to the patient (printed + portal, with plain-language instructions), to the receiving provider (PCP, specialist, SNF), and to the outpatient pharmacy via Surescripts NCPDP SCRIPT.

## High-Alert Medication Classes

The ISMP High-Alert Medications list identifies drug classes that cause disproportionate harm when used in error: anticoagulants, insulin, opioids, neuromuscular blockers, chemotherapy, concentrated electrolytes, and others. At med rec, these classes warrant extra scrutiny — independent double-check by a pharmacist or second prescriber.

> Use the current ISMP High-Alert Medications list — verify against ISMP's published list rather than relying on outdated copies.

## Quality Measures

Med rec quality is tracked by:

- **Joint Commission NPSG 03.06.01** — Maintain and communicate accurate medication information at transitions. Verify current text and applicability.
- **CMS Promoting Interoperability / MIPS** — Med rec at transitions reported electronically. Verify current measure ID and specification.
- **HEDIS Medication Reconciliation Post-Discharge (MRP)** — Outpatient follow-up med rec after acute admission. Verify current MRP specification.

Measure IDs, denominators, and reporting periods change annually — confirm against current TJC, CMS, and NCQA specifications.

## Common Pitfalls

- Treating Surescripts fill history as the med list. Fills are evidence, not proof of taking.
- Comparing brand-named BPMH entries to generic-ordered entries by string. Normalize to RxCUI first.
- Auto-reconciling without prescriber sign-off. Med rec is a clinical act with attestation.
- Skipping the reason field for every change. Auditors and the next clinician need to know why.
- Forgetting OTC, herbals, and supplements. They cause real interactions.
- Re-running med rec at every shift without a discrepancy. Provider fatigue is the enemy.
- Using `MedicationRequest` for the BPMH instead of `MedicationStatement`. Wrong resource model breaks the entire downstream pipeline.
