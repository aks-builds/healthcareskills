---
name: cpoe-orders
description: When the user wants to design, build, or integrate with computerized provider order entry (CPOE). Also use when the user mentions "CPOE," "order entry," "order set," "order panel," "MedicationRequest," "ServiceRequest," "ORM," "ORM^O01," "OMP^O09," "electronic signature," "duplicate order check," "dose check," "allergy check," "drug-drug interaction at ordering," "verbal order," "telephone order," "range order," "conditional order," "PRN order," "heparin nomogram," "PCA," "IV pump," "downtime ordering," "order pathway," "order propagation to pharmacy," or "Joint Commission ordering standard." For ambient scribe note write-back, see clinical-documentation. For CDS at the moment of ordering, see clinical-decision-support. For medication reconciliation at transitions, see medication-reconciliation.
metadata:
  version: 1.0.0
---

# CPOE Orders

You are an expert in computerized provider order entry (CPOE). Your goal is to help engineers and informaticists design and integrate ordering workflows that are safe, efficient, and faithful to how clinicians actually place orders — including the messy parts (verbal orders, range orders, conditional orders, downtime).

## Initial Assessment

Read `.agents/healthcare-context.md` first (fall back to `.claude/healthcare-context.md`). The context file tells you which EHRs, which clinical settings (inpatient ordering looks very different from ambulatory), and which downstream systems (pharmacy, lab, radiology) need to receive orders.

If the context file does not exist, ask:

1. Which order types — medication, lab, imaging, referral, nursing, diet, other?
2. Inpatient, ambulatory, ED, peri-op, or oncology?
3. Which EHR places the order, and which downstream systems consume it?
4. HL7 v2 (ORM/OMP) or FHIR (`MedicationRequest` / `ServiceRequest`) for propagation?
5. Are you building inside the EHR, embedding via SMART, or routing externally?

---

## Order Types

| Type | FHIR resource | HL7 v2 trigger | Notes |
|------|---------------|-----------------|-------|
| Medication | `MedicationRequest` | OMP^O09 (pharmacy), RDE^O11 (e-Rx) | Inpatient and outpatient prescribing |
| Lab | `ServiceRequest` (category=laboratory) | OML^O21 (lab) or ORM^O01 (legacy) | Routed to LIS |
| Imaging | `ServiceRequest` (category=imaging) | OMI^O23 (imaging) or ORM^O01 | Routed to RIS / PACS |
| Referral | `ServiceRequest` (referral) or `ReferralRequest` (older) | REF^I12 | To external provider or service |
| Nursing | `ServiceRequest` (category=nursing) or `Task` | Often EHR-internal, no v2 | Vitals q4h, ambulate, fall precautions |
| Diet | `NutritionOrder` | OMD^O03 | Routed to dietary |
| Blood product | `ServiceRequest` linked to `Specimen` / blood bank flow | OMB^O27 | Type & screen, crossmatch, transfuse |
| DME / supply | `SupplyRequest` | Varies | Ostomy supplies, walkers, etc. |

Use FHIR R4 resource categorization rigorously — the `category` and `code` together drive routing.

---

## Order Set vs. Panel vs. Pathway

| Construct | What | When |
|-----------|------|------|
| **Order set** | Curated group of orders for a scenario (e.g., "Adult Sepsis Bundle") | Standardize complex order placement |
| **Order panel** | Smaller pre-built group, often a single phase | Pre-op labs, NPO at midnight + meds |
| **Order pathway** | Multi-day plan with sequencing and conditional logic | Total knee replacement, oncology regimen |

Order sets are the highest-leverage CDS surface — they change ordering behavior **without firing an alert**. Invest there before adding interruptive alerts.

---

## Order Lifecycle

```
Drafted → Signed (electronic signature) → Verified (pharmacy / nursing) → Active → Completed | Cancelled | Modified | Discontinued
```

FHIR `MedicationRequest.status`: `active`, `on-hold`, `cancelled`, `completed`, `entered-in-error`, `stopped`, `draft`, `unknown`.

FHIR `ServiceRequest.status`: same enumeration as above (per R4 spec — verify against the current FHIR spec).

A signed order is a legal record. The signing event is auditable and (in modern EHRs) cryptographically attributable.

---

## Safety Checks at Order Entry

Every CPOE system runs a battery of checks **before** the order is signed. These are commonly evaluated in the `order-select` and `order-sign` CDS Hooks events, or in the EHR's native rules engine.

| Check | What | When to fire |
|-------|------|--------------|
| Allergy | Ordered med vs. patient `AllergyIntolerance` list | Always; modal for severe |
| Drug-drug interaction (DDI) | Ordered med vs. active med list | Severe interruptive; moderate passive |
| Drug-disease | Med vs. active problem (e.g., NSAID + CKD) | Per severity |
| Dose check | Dose vs. age, weight, renal function, hepatic function | Interruptive if exceeds maximum |
| Duplicate therapy | Same drug or same therapeutic class already active | Almost always passive (override common and valid) |
| Pregnancy / lactation | Med vs. pregnancy status | Interruptive for category D / X equivalents |
| Renal / hepatic adjust | Recommend dose adjust based on eGFR / Child-Pugh | Passive suggestion |
| Formulary | On / off formulary, prior auth required | Passive |
| Cost / OOP | Patient out-of-pocket at this pharmacy | Passive (RxBenefit) |

**Tune aggressively.** Most CPOE installs override 90%+ of moderate-severity DDI alerts. Either raise the threshold or move them to passive.

---

## Specialized Medication Workflows

### High-alert classes

The Institute for Safe Medication Practices (ISMP) maintains the **High-Alert Medication list**. These classes (e.g., anticoagulants, insulin, opioids, chemotherapy, neuromuscular blockers) warrant extra checks — double-signature, independent double-check, smart-pump integration.

Use the current ISMP list — verify against ISMP's published list, do not rely on outdated examples.

### IV / infusion orders

- Continuous infusions (e.g., norepinephrine, heparin, propofol) require **rate** and **concentration**, not just dose.
- Smart-pump integration via the **drug library** ties the order to the pump's hard/soft dose limits.
- Heparin **nomogram** orders adjust rate based on aPTT — encoded as a conditional/standing order or as nursing-driven titration with bounds.
- PCA (patient-controlled analgesia) orders specify basal rate, demand dose, lockout interval, max per hour.

Never invent specific concentrations or rates — verify against the institution's medication formulary.

### Range orders

"Morphine 2-4 mg IV q4h PRN pain" — bounded by a range. Joint Commission has specific standards for how range orders must be expressed and how nursing chooses within the range; verify against the current Joint Commission medication management chapter.

### Conditional / standing / PRN orders

| Order type | Example |
|-----------|---------|
| PRN (as needed) | Acetaminophen 650 mg PO q6h PRN pain ≥4/10 |
| Conditional | Hold metoprolol if SBP < 100 or HR < 55 |
| Standing | All patients on this unit get DVT prophylaxis unless contraindicated |

Each requires the EHR to model the condition explicitly so nursing can document the trigger when the dose is given.

---

## Electronic Signature

- The signing event is the **provider's legal authorization** for the order.
- Federal e-prescribing of controlled substances (EPCS) requires **two-factor authentication** at the signing event (DEA EPCS rule — verify current requirements).
- Each signed order has an audit record: who, when, from which workstation, with which authentication.
- Co-signature for residents, students, and APPs follows site policy and state scope-of-practice rules.

---

## Verbal and Telephone Orders

Despite CPOE, verbal and telephone orders persist — emergencies, sterile field, scrubbed providers.

- Read-back / repeat-back is the standard safety mechanism (Joint Commission NPSG — verify current standard).
- The order is entered by the nurse / unit clerk with the provider's name; the provider co-signs within a defined window (often 24-48h per site policy).
- "Standing verbal" orders are heavily restricted by Joint Commission. Confirm site policy and current standards.

---

## Order Propagation

When the order is signed in the EHR, it must reach the executing system (pharmacy, lab, radiology, EHR's own nursing flowsheets).

### HL7 v2

| Trigger | Use |
|---------|-----|
| ORM^O01 | Legacy generic order message (still common) |
| OMP^O09 | Pharmacy order |
| OML^O21 | Lab order |
| OMI^O23 | Imaging order |
| OMG^O19 | General order (replaces some ORM usage) |
| RDE^O11 | e-Rx (NCPDP for retail; in-hospital uses pharmacy v2 family) |

For acknowledgments and status updates, use ORR^O02 (response), OMx-specific acks, and ORM updates (NW = new, CA = cancel, XO = change, DC = discontinue).

### FHIR

`MedicationRequest` and `ServiceRequest` carry the order; downstream:

- `MedicationDispense` — pharmacy filled the order
- `MedicationAdministration` — nurse gave the dose
- `DiagnosticReport` + `Observation` — lab result
- `ImagingStudy` — DICOM study against the imaging request
- `Task` — to track who is doing what across the lifecycle

`ServiceRequest.basedOn` and `MedicationDispense.authorizingPrescription` thread the lineage.

### Surescripts (outpatient e-Rx)

Outpatient prescriptions in the US flow through Surescripts via NCPDP SCRIPT, not HL7. Receipt of the prescription, medication history, formulary, and benefit checks all go through the Surescripts network. Confirm current NCPDP SCRIPT version supported.

---

## Downtime Ordering

Every CPOE-dependent operation needs a downtime plan.

- **Planned downtime** (maintenance): pre-print paper order forms, designate a downtime workstation with read-only access.
- **Unplanned downtime**: paper orders, paper MAR (medication administration record), runner-based delivery to pharmacy and lab.
- **Recovery**: re-enter all paper orders into the EHR after recovery, with explicit timestamps for when the order was actually written.
- Downtime drills are a regulatory expectation in most accreditation regimes.

---

## Role-Based Ordering Authority

| Role | Typical authority |
|------|-------------------|
| Attending physician | Full ordering authority within license + privileges |
| Resident | Per supervision rules; controlled substance orders may need attending co-sign |
| APP (NP, PA) | Per state scope-of-practice + collaborative agreement |
| Pharmacist | Verify, suggest substitutions; some states allow protocol-based prescribing |
| Nurse | Enter verbal/telephone orders; titrate per protocol within bounds |
| Medical student | Enter draft orders for attending signature (per CMS billing rules — verify current) |

EHRs enforce this through role-based access and conditional UI; check the customer's privileging file.

---

## Common Pitfalls

- Modeling every medication order as a single dose. Tapers, scheduled vs. PRN, range, conditional, and continuous infusions all need richer representations.
- Treating "the order is signed" as terminal. The order is only useful once it has propagated to pharmacy/lab/etc. and the receiving system has acknowledged.
- Ignoring `entered-in-error`. Orders entered on the wrong patient happen; the system must support clean correction with audit.
- Forgetting downtime. Every CPOE deployment needs paper backup workflows.
- Hardcoding allergy / DDI thresholds. Tune per service line; expect to retire low-acceptance alerts.
- Verbal orders without read-back. Regulatory and patient-safety risk.

---

## Worked Example: New Medication Order (FHIR)

```http
POST /FHIR/R4/MedicationRequest
Authorization: Bearer <token>
Content-Type: application/fhir+json
{
  "resourceType": "MedicationRequest",
  "status": "active",
  "intent": "order",
  "category": [{
    "coding": [{
      "system": "http://terminology.hl7.org/CodeSystem/medicationrequest-category",
      "code": "inpatient"
    }]
  }],
  "medicationCodeableConcept": {
    "coding": [{
      "system": "http://www.nlm.nih.gov/research/umls/rxnorm",
      "code": "<verify against current RxNorm release>",
      "display": "<medication display name>"
    }]
  },
  "subject": { "reference": "Patient/example-1" },
  "encounter": { "reference": "Encounter/example-enc-1" },
  "authoredOn": "2026-01-15T09:30:00Z",
  "requester": { "reference": "Practitioner/example-prov-1" },
  "dosageInstruction": [{
    "text": "Per institution formulary — verify before use",
    "timing": { "repeat": { "frequency": 1, "period": 12, "periodUnit": "h" } },
    "route": { "coding": [{ "system": "http://snomed.info/sct", "code": "<verify SNOMED CT route code>" }] }
  }]
}
```

Never invent RxNorm codes, doses, or routes — those must come from the verified formulary and a current terminology release.

---

## Task-Specific Questions

1. Which order types and which settings?
2. Which EHR and which downstream systems?
3. HL7 v2, FHIR, or both?
4. Which safety checks are in scope, and which do you tune (vs. inherit from the EHR)?
5. Are controlled substances in scope (EPCS)? Outpatient e-Rx (Surescripts)?
6. What is your downtime plan?

---

## Related Skills

- **healthcare-context**: Setting and EHR drive ordering UX and propagation.
- **clinical-decision-support**: `order-select` and `order-sign` hooks fire at exactly this moment.
- **medication-reconciliation**: What happens to the active med list at admission, transfer, discharge.
- **fhir-integration**: `MedicationRequest`, `ServiceRequest`, `MedicationDispense`, `MedicationAdministration`.
- **hl7-v2**: Pharmacy / lab / imaging order trigger families (ORM, OMP, OML, OMI).
- **terminology-services**: RxNorm, SNOMED CT route, LOINC for orderable tests.
- **ehr-integration**: Vendor-specific order placement and order-set tooling.
