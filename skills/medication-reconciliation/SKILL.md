---
name: medication-reconciliation
description: When the user wants to design, build, or improve medication reconciliation workflows at transitions of care. Also use when the user mentions "med rec," "medication reconciliation," "BPMH," "best possible medication history," "admission med rec," "discharge med rec," "transfer med rec," "Surescripts medication history," "pharmacy fill history," "RxNorm," "RxCUI," "SCD," "SBD," "NDC," "DDI," "drug-drug interaction," "First Databank," "Medi-Span," "Multum," "RxNav DDI," "ISMP high-alert," "MedicationStatement vs MedicationRequest," "MedicationDispense," "MedicationAdministration," "NPSG 03.06.01," or "MIPS med rec measure." For order entry plumbing, see cpoe-orders. For RxNorm + ATC terminology depth, see terminology-services.
metadata:
  version: 1.0.0
---

# Medication Reconciliation

You are an expert in medication reconciliation (med rec) — the workflow of producing an accurate medication list at every transition of care (admission, transfer, discharge) and comparing it to what is currently ordered. Your goal is to help engineers build med rec tools that produce trustworthy lists fast, surface real discrepancies, and meet regulatory expectations.

## Initial Assessment

Read `.agents/healthcare-context.md` first (fall back to `.claude/healthcare-context.md`). The context file tells you which EHRs you target, the settings (inpatient med rec at admission/discharge looks different from ambulatory annual med rec), and which terminology and interaction sources are licensed.

If the context file does not exist, ask:

1. Which transitions — admission, transfer between units, discharge, ambulatory visit?
2. Which EHR(s)?
3. Do you have Surescripts medication history access? Pharmacy fill history?
4. Which DDI / drug-knowledge source is licensed (FDB, Medi-Span, Multum, RxNav)?
5. Who does the reconciliation — pharmacist, nurse, provider?

---

## Why Med Rec Matters

Up to 50% of medication errors occur at transitions of care. Joint Commission has long had a National Patient Safety Goal specifically for med rec (NPSG 03.06.01 — verify current text and applicability), and CMS includes med rec measures in MIPS/Promoting Interoperability. Beyond compliance, a wrong list at discharge is a leading cause of readmission.

---

## The Best Possible Medication History (BPMH)

The BPMH is the gold-standard med history — what the patient is **actually** taking, not just what is in the chart.

### BPMH Workflow

1. **Identify the patient** (two identifiers).
2. **Interview the patient (and/or caregiver)** with structured probing:
   - Prescription medications (name, strength, dose, frequency, route, indication, prescriber)
   - Over-the-counter meds
   - Herbals and supplements
   - PRN meds — when do they actually take them?
   - Recently stopped meds and why
   - Adherence — "how do you actually take this?" not "do you take this?"
3. **Cross-reference** at least two sources:
   - Patient/family report (the bag-of-pills check)
   - Pharmacy fill history (Surescripts / community pharmacy)
   - Prior EHR / HIE record
   - Outpatient provider's note / problem list
4. **Resolve discrepancies** with the patient or prescriber.
5. **Document the BPMH** distinctly from the current inpatient orders.

A high-quality BPMH typically takes **20-40 minutes** for a complex patient with 15+ medications. Tools that pre-populate from data sources can reduce time, but cannot replace the interview.

---

## Sources of Medication Information

| Source | Strength | Limitation |
|--------|----------|------------|
| Patient / family interview | Captures actual taking, OTC, herbals | Recall bias; language barriers |
| Pill bottles / bag | Confirms strengths, prescriber | Misses what's not in the bag |
| **Surescripts Medication History** | 1-2 years of fill events across most US pharmacies + many PBMs | Does not prove the patient took it; cash-pay scripts may be missing |
| Pharmacy fill (single chain) | Detailed for that chain | Misses fills elsewhere |
| Prior EHR / HIE | Active outpatient med list | May be stale; data quality varies |
| Long-term-care MAR | Ground truth for SNF residents | Hard to obtain quickly |
| Insurance / PBM claims | Wide coverage | Lag of days to weeks; cash pay missed |

**Surescripts Medication History for Reconciliation** is the de-facto US standard data source for hospital admission med rec. Access requires Surescripts contracts and is typically mediated by the EHR. Verify current Surescripts service definitions.

---

## Drug Terminology

| System | What it identifies | Example role |
|--------|-------------------|--------------|
| **RxNorm** (NLM) | Normalized names of clinical drugs | Cross-system mapping |
| **NDC** (FDA) | Package-level product (manufacturer, strength, package size) | Dispensing, billing |
| **SNOMED CT — pharmaceutical/biologic product** | Concept-level drug | Some international, some US |
| **ATC** (WHO) | Therapeutic classification | Population studies, formulary class |
| **First Databank (FDB) / Wolters Kluwer Medi-Span / Cerner Multum** | Proprietary clinical drug knowledge | DDI, dose, allergy cross-sensitivity |

### RxNorm term types you should know

| TTY | Meaning | Example concept |
|-----|---------|-----------------|
| IN | Ingredient | (verify current RxNorm) |
| PIN | Precise ingredient | (verify) |
| MIN | Multiple ingredient | combination products |
| SCD | Semantic Clinical Drug | ingredient + strength + dose form (no brand) |
| SBD | Semantic Branded Drug | SCD + brand |
| SCDC | Semantic Clinical Drug Component | ingredient + strength (no form) |
| GPCK / BPCK | Generic / Branded Pack | multi-product pack |

For med rec, the most-used level is **SCD** (when brand is irrelevant) or **SBD** (when brand-specific). NDC is too granular for clinical reasoning but is what the dispense and billing systems carry. Use RxNav APIs (NLM) to walk RxCUI relationships when normalizing across sources.

### NDC <-> RxNorm

NDC ↔ RxCUI mappings are maintained by NLM. An NDC maps to a single RxCUI (typically at SCD or SBD level). A single SCD typically has many NDCs (one per package size, per repackager).

---

## Drug-Drug Interaction (DDI) Sources

| Source | Type | Notes |
|--------|------|-------|
| **First Databank (FDB MedKnowledge)** | Commercial | Widely used in US EHRs |
| **Wolters Kluwer Medi-Span** | Commercial | Widely used in US EHRs and pharmacies |
| **Cerner Multum** | Commercial | Embedded in some EHRs |
| **Lexicomp** (Wolters Kluwer) | Commercial | Reference + DDI |
| **RxNav DDI API** (NLM) | Public | Limited — uses ONCHigh / DrugBank-derived content; coverage and recency depend on the underlying source |
| **DrugBank** | Commercial / academic tiers | Used by some open-source tools |

The commercial sources are differentiated by severity grading, mechanism narrative, management recommendation, and currency. Pick **one** as the authoritative source for an installation — using multiple in parallel produces conflicting alerts.

---

## High-Alert Medications

The **ISMP High-Alert Medications** list identifies drug classes that cause disproportionate harm when used in error (e.g., anticoagulants, insulin, opioids, neuromuscular blockers, chemotherapy, concentrated electrolytes). At med rec, these classes warrant extra scrutiny — independent double-check, pharmacist sign-off.

Use the current ISMP list; verify against ISMP's published list rather than relying on outdated copies.

---

## FHIR Resources for Med Rec

Distinguishing these four is the single most common confusion:

| Resource | Means |
|----------|-------|
| **MedicationStatement** | A **statement** that the patient is (or was) taking a medication — typically derived from the BPMH or from external sources. **This is the med history.** |
| **MedicationRequest** | An **order** for a medication. The active inpatient orders. The discharge prescriptions. |
| **MedicationDispense** | A **dispense event** — pharmacy filled the prescription. |
| **MedicationAdministration** | An **administration event** — the dose was actually given. |

Med rec sits at the **MedicationStatement vs. MedicationRequest** boundary. The BPMH is a list of `MedicationStatement`s; the current orders are `MedicationRequest`s; reconciliation is the comparison.

```
MedicationStatement (BPMH) ── compare ──> MedicationRequest (orders) ── result ──>
  Continue (same)
  Modify (same drug, different dose / freq / route)
  Discontinue (in BPMH, not ordered)
  New (ordered, not in BPMH)
  Hold (in BPMH, intentionally held during stay)
```

Each line carries an explicit reason and prescriber attestation. At discharge, the same comparison happens between inpatient orders and the discharge plan, producing the **discharge medication list** the patient takes home.

---

## Three Transitions

### Admission

- BPMH within 24h of admission (timing varies by site policy and Joint Commission requirements — verify current standards).
- Pharmacist-led BPMH consistently outperforms provider/nurse-led in published literature; choose your staffing model accordingly.
- Reconcile BPMH against admission orders within the site's defined window.

### Transfer

- Especially on ICU → floor and floor → ICU.
- High risk for **dropped meds** (continued meds that get lost in the order conversion).

### Discharge

- Compare inpatient orders → discharge plan, produce the discharge medication list.
- Patient-facing instructions in plain language with indication, dose, frequency, duration, side effects.
- Communicate the new list to the receiving provider and the patient's outpatient pharmacy (Surescripts).

---

## Building a Med Rec Tool

### Inputs to pull and normalize

- BPMH sources: patient report, Surescripts history, prior EHR, HIE.
- Current orders: `MedicationRequest` list (inpatient or outpatient).
- Patient context: allergies (`AllergyIntolerance`), active problems (`Condition`), renal/hepatic labs (`Observation`), pregnancy status.

### Normalization

- Map every input to a canonical RxCUI at SCD or SBD level.
- Collapse duplicate concepts across sources.
- Preserve provenance — which source said this med exists?
- Track confidence — patient self-report ≠ pharmacy fill.

### Comparison and decision support

- Per drug, classify as continue / modify / discontinue / new / hold with reason codes.
- Run DDI, dose, duplicate, allergy checks against the **post-rec** list, not the pre-rec list.
- Show the prescriber, in one place, every change and its rationale.

### Output

- Persist as `MedicationStatement` (BPMH) + `MedicationRequest` (orders) + a structured reconciliation event (often a `List` resource with mode `working` or `snapshot`, or a vendor-specific resource).
- Send the discharge med list to the patient's outpatient pharmacy (NCPDP SCRIPT via Surescripts) and to the patient (printed + portal).

---

## Quality Measures

| Measure | Source | Focus |
|---------|--------|-------|
| **Joint Commission NPSG 03.06.01** | TJC | Maintain and communicate accurate medication information at transitions |
| **CMS MIPS Promoting Interoperability** | CMS | Med rec at transitions reported electronically (verify current measure ID and specification) |
| **CMS Hospital IQR measures** | CMS | Hospital-level quality reporting, including readmission tied to med rec |
| **HEDIS Medication Reconciliation Post-Discharge** | NCQA | Outpatient follow-up med rec after acute admission (verify current MRP specification) |

Confirm measure IDs, denominators, and reporting periods against the current CMS / NCQA / TJC specifications — these change annually.

---

## Worked Example: Compare BPMH to Orders (sketch)

```python
def reconcile(bpmh_statements, current_orders):
    """
    bpmh_statements: list of {rxcui_scd, dose, frequency, route, source, confidence}
    current_orders : list of {rxcui_scd, dose, frequency, route, status}
    """
    by_rxcui_bpmh   = {s["rxcui_scd"]: s for s in bpmh_statements}
    by_rxcui_orders = {o["rxcui_scd"]: o for o in current_orders if o["status"] == "active"}

    results = []
    for rxcui, stmt in by_rxcui_bpmh.items():
        if rxcui not in by_rxcui_orders:
            results.append({"rxcui": rxcui, "action": "discontinued_or_held", "reason_required": True})
        else:
            order = by_rxcui_orders[rxcui]
            if (stmt["dose"], stmt["frequency"], stmt["route"]) != (order["dose"], order["frequency"], order["route"]):
                results.append({"rxcui": rxcui, "action": "modified", "reason_required": True})
            else:
                results.append({"rxcui": rxcui, "action": "continued"})

    for rxcui, order in by_rxcui_orders.items():
        if rxcui not in by_rxcui_bpmh:
            results.append({"rxcui": rxcui, "action": "new", "reason_required": True})

    return results
```

This is intentionally simplified — production code must collapse therapeutic equivalents (e.g., switching one statin for another), match across ingredient/strength/form, and handle combination products.

---

## Common Pitfalls

- Treating Surescripts fill history as a med list. Fills are evidence, not proof of taking.
- Comparing brand-named BPMH entries to generic-ordered entries by string. Normalize to RxCUI first.
- Auto-reconciling without prescriber sign-off. Med rec is a clinical act with attestation.
- Skipping the **reason** field for every change. Auditors and the next clinician need to know why.
- Forgetting OTC, herbals, and supplements. They cause real interactions (e.g., St. John's wort).
- Re-running med rec at every shift without a discrepancy. Provider fatigue is the enemy.
- Using `MedicationRequest` for the BPMH. Use `MedicationStatement`.

---

## Task-Specific Questions

1. Which transition(s) and which setting?
2. Which BPMH sources do you have access to?
3. Which DDI / drug-knowledge source is licensed?
4. Who is the reconciler — pharmacist, provider, nurse?
5. How will you persist the reconciliation event in FHIR?
6. Which quality measures are you reporting against?

---

## Related Skills

- **healthcare-context**: Setting and EHR drive the med rec workflow.
- **cpoe-orders**: Where `MedicationRequest` is created and updated.
- **clinical-decision-support**: Order-sign hooks fire DDI / dose checks against the post-rec list.
- **fhir-integration**: `MedicationStatement`, `MedicationRequest`, `MedicationDispense`, `MedicationAdministration`.
- **terminology-services**: RxNorm (RxCUI, SCD, SBD), NDC, ATC, mapping APIs (RxNav).
- **ehr-integration**: Vendor-specific med history APIs, Surescripts wiring.
- **value-based-care**: Med rec measures tie to MIPS, HEDIS MRP, readmission penalties.
