# RCKMS Reportable Conditions — Pointer

This is intentionally a pointer, not a static list. The Reportable Conditions Knowledge Management System (RCKMS) is updated continuously by CSTE in partnership with APHL, and the per-jurisdiction reportable conditions lists are maintained by each state, local, territorial, and tribal public health authority. **Always pull the current list from RCKMS and the receiving jurisdiction's published statute or rule — do not hard-code any list in an EHR or copy it into a static document.**

## Contents

- Why this is not a static list
- Where to look
- Categories of reportable conditions (illustrative)
- How RCKMS uses these
- Verification expectations

---

## Why this is not a static list

- **Conditions are added and modified continuously**. New pathogens, outbreak conditions (e.g., novel coronaviruses, mpox, avian influenza), antimicrobial-resistance markers, and emerging hazards are added by jurisdictions as warranted. A snapshot taken on any given date will be stale within months.
- **Jurisdictions differ**. The CSTE-recommended list of nationally notifiable conditions is one input; each state, local, territorial, and tribal jurisdiction has its own statute, rule, or regulation that defines what is reportable, by whom, on what timeline, and to whom. There is no single national "reportable conditions list" that applies everywhere.
- **Trigger codes evolve**. Even when the condition is unchanged, the underlying LOINC / SNOMED CT / ICD-10-CM / RxNorm trigger codes in the RCTC value sets are updated as terminologies evolve.
- **Operational logic is in RCKMS**. The whole point of RCKMS is to centralize jurisdiction-aware reportability decisioning so EHRs do not need to maintain per-state condition lists.

---

## Where to look

| Source | What it has | Why use it |
|--------|-------------|-----------|
| **RCKMS** (rckms.org) | Jurisdiction-aware rules engine and condition coverage | Authoritative for "is this reportable here" |
| **CSTE Position Statements** | National notifiable conditions list and case definitions | Tracks nationally notifiable conditions over time |
| **CDC NNDSS** (National Notifiable Diseases Surveillance System) | Conditions CDC collects nationally | National-level scope |
| **VSAC RCTC value sets** (vsac.nlm.nih.gov) | Trigger codes used by EHRs for eCR | Required for triggering — verify current publication date |
| **Receiving state PHA statute / rule** | Legally binding reportable list for that jurisdiction | Required for clinician-facing legal compliance |

---

## Categories of reportable conditions (illustrative, non-exhaustive)

These are broad categories that most jurisdictions have reportable conditions in. The specific conditions, codes, timelines, and recipients vary. **Verify against current RCKMS and the receiving jurisdiction.**

- **Vaccine-preventable diseases** (e.g., measles, mumps, rubella, pertussis, varicella, polio, diphtheria).
- **Foodborne and waterborne diseases** (e.g., Salmonella, Shigella, E. coli O157:H7, Listeria, Vibrio, Cryptosporidium).
- **Vector-borne diseases** (e.g., West Nile virus, dengue, Zika, Lyme disease, Rocky Mountain spotted fever, malaria).
- **Sexually transmitted infections** (e.g., chlamydia, gonorrhea, syphilis, congenital syphilis).
- **HIV and AIDS** (with enhanced confidentiality).
- **Viral hepatitis** (A, B, C, D, E — acute and chronic).
- **Tuberculosis** and latent TB infection where applicable.
- **Healthcare-associated infections and antimicrobial resistance markers** (e.g., CRE, MRSA bloodstream, C. difficile — many through NHSN rather than direct ELR/eCR).
- **Respiratory pathogens** (e.g., influenza, novel coronaviruses, RSV in some jurisdictions, Legionella).
- **Outbreak / emerging infections** (e.g., mpox, avian influenza in humans).
- **Vaccine adverse events** (VAERS).
- **Lead poisoning and environmental exposures** (e.g., elevated blood lead, pesticide exposure).
- **Cancer** (to state central cancer registry; separate stream from communicable disease reporting).
- **Bioterrorism agents** (e.g., anthrax, smallpox, plague, tularemia, viral hemorrhagic fevers) — often with shortened reporting timelines.
- **Maternal and child health conditions** in some jurisdictions (e.g., neonatal abstinence syndrome, certain birth defects).
- **Occupational and environmental conditions** in some jurisdictions.

This list is illustrative only — do not implement against it. Pull the current reportable list from RCKMS and the receiving jurisdiction.

---

## How RCKMS uses these

RCKMS evaluates incoming eICRs against jurisdiction-specific rules that encode:

- Which condition(s) the trigger codes implicate.
- Whether the patient's residence, encounter facility, or specimen jurisdiction makes the case reportable in this PHA's territory.
- Whether case characteristics (age, sex, pregnancy status, etc.) meet that jurisdiction's case definition.
- The reporting timeline expected.
- Any supplemental data the jurisdiction wants returned in the Reportability Response.

EHRs and sponsors implementing eCR do not need to recreate this logic — they need to keep their RCTC subscriptions current and their eICR generation faithful.

---

## Verification expectations

Before any production go-live or any release that touches reportability:

1. Verify current RCTC value-set publication date on VSAC.
2. Verify current RCKMS rule coverage for each target jurisdiction.
3. Verify the receiving state PHA's published reportable conditions list for legal / clinician-facing reference.
4. Verify any conditions on shortened timelines (often bioterrorism, novel pathogens, outbreaks) have the right routing.
5. Verify CDC NNDSS scope if national rollup is in scope.

Always pull from current sources — do not hard-code, do not snapshot, do not copy this content into production logic.
