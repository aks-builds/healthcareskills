# Code System Cheat Sheet

Quick reference to the major healthcare code systems — purpose, governance body, license, FHIR system URI, common pitfalls. Always verify current naming, licensing terms, and FHIR URIs against the authoritative source.

## Contents

- SNOMED CT
- ICD-10-CM (US Diagnoses)
- ICD-10-PCS (US Inpatient Procedures)
- ICD-11 (WHO)
- CPT
- HCPCS Level II
- LOINC
- RxNorm
- NDC
- UCUM
- CVX / MVX
- ICF
- Other Common Code Systems
- Picking the Right System
- Common Gotchas

---

## SNOMED CT

- **Full name**: Systematized Nomenclature of Medicine — Clinical Terms
- **Maintainer**: SNOMED International (formerly IHTSDO)
- **Scope**: clinical findings, disorders, procedures, observable entities, body structures, organisms, substances, qualifier values, situations, events, social context — the broadest clinical terminology in healthcare
- **Identifier**: SCTID — long integer with a check digit
- **Hierarchy**: poly-hierarchical IS-A rooted at SNOMED CT Concept (138875005)
- **Distribution**: RF2 release format
- **Updates**: International edition monthly; US Edition twice yearly (March, September); UK / AU / others on their own cadences
- **License**: SNOMED International member-country license. In the US, NLM provides free use under UMLS license. Non-member-country use requires paid affiliate license.
- **FHIR system URI**: `http://snomed.info/sct`
- **Versioned URI**: `http://snomed.info/sct/{moduleId}/version/{date}` (e.g., `http://snomed.info/sct/731000124108/version/20250901` for US Edition)
- **Common gotchas**:
  - International vs. US Edition — concepts in US Edition may not exist internationally
  - Post-coordination expressions are powerful but many systems can't store them
  - ECL (Expression Constraint Language) is the right way to define value sets
  - Inactivation: concepts can be retired; preserve historical SCTIDs

---

## ICD-10-CM (US Diagnoses)

- **Full name**: International Classification of Diseases, Tenth Revision, Clinical Modification
- **Maintainer**: NCHS (National Center for Health Statistics) / CDC
- **Scope**: diagnoses for US outpatient and inpatient billing
- **Updates**: annual release effective October 1; mid-year addenda possible
- **License**: free in the US
- **FHIR system URI**: `http://hl7.org/fhir/sid/icd-10-cm`
- **Common gotchas**:
  - 3 to 7 characters, with the 7th character sometimes mandatory (e.g., injury episode of care: A=initial, D=subsequent, S=sequela)
  - "Excludes1" vs. "Excludes2" notes — different semantics, often misapplied
  - ICD-10-CM (US) is **different** from ICD-10 (WHO) — codes look similar but are not interchangeable
  - Annual updates break frozen value sets in quality measures

---

## ICD-10-PCS (US Inpatient Procedures)

- **Full name**: International Classification of Diseases, Tenth Revision, Procedure Coding System
- **Maintainer**: CMS
- **Scope**: inpatient hospital procedure coding in the US
- **Structure**: 7-character alphanumeric codes; each position has defined semantics (section, body system, root operation, body part, approach, device, qualifier)
- **License**: free in the US
- **FHIR system URI**: `http://www.cms.gov/Medicare/Coding/ICD10`
- **Common gotchas**:
  - Inpatient only — outpatient procedures bill on CPT/HCPCS
  - Every position must be filled; default to "X" (no qualifier) for unused positions
  - Root operation definitions are precise — coders need training

---

## ICD-11 (WHO)

- **Full name**: International Classification of Diseases, 11th Revision
- **Maintainer**: World Health Organization
- **Scope**: international diagnostic classification with multiple linearizations (Mortality and Morbidity Statistics — MMS — primary)
- **Adoption**: some countries are adopting; not yet in US production billing
- **License**: free (WHO)
- **FHIR system URI**: `http://id.who.int/icd11/mms` (verify current canonical URI)
- **Common gotchas**:
  - Coexists with ICD-10 in many countries during transition
  - Cluster coding (postcoordination) is built in
  - Stem codes vs. extension codes — different roles
  - US adoption timeline unclear

---

## CPT (Current Procedural Terminology)

- **Maintainer**: American Medical Association (AMA)
- **Scope**: physician procedures and services in the US — Evaluation & Management (E/M), Surgery, Radiology, Pathology and Laboratory, Medicine
- **Categories**:
  - Category I: standard procedures
  - Category II: performance measurement (tracking codes)
  - Category III: emerging technology (temporary)
- **Updates**: annual (January 1)
- **License**: AMA-licensed. Commercial / clinical use requires an AMA license.
- **FHIR system URI**: `http://www.ama-assn.org/go/cpt`
- **Common gotchas**:
  - Licensing — cannot redistribute CPT codes commercially without AMA license
  - Modifiers (two-character suffixes) change billing semantics significantly (LT/RT, 50, 59, 25, etc.)
  - E/M codes (99201–99499) underwent major restructuring in recent years

---

## HCPCS Level II

- **Full name**: Healthcare Common Procedure Coding System Level II
- **Maintainer**: CMS
- **Scope**: products, supplies, drugs, and non-physician services — DME (durable medical equipment), ambulance, injectable drugs (J-codes), prosthetics, orthotics, transportation
- **Structure**: one alphabetic character followed by four digits (A0000–V9999), plus modifiers
- **Updates**: quarterly
- **License**: free
- **FHIR system URI**: `https://www.cms.gov/Medicare/Coding/MedHCPCSGenInfo`
- **Common gotchas**:
  - HCPCS Level I is CPT; HCPCS Level II is this. "HCPCS" in common parlance usually means Level II.
  - J-codes for injectable drugs are essential for outpatient drug billing
  - Modifiers overlap with CPT modifiers but have distinct ones (KX, GA, GZ, etc.)

---

## LOINC

- **Full name**: Logical Observation Identifiers Names and Codes
- **Maintainer**: Regenstrief Institute
- **Scope**: laboratory tests, clinical observations, vital signs, surveys, documents, panels
- **Structure**: six-part fully specified name (FSN): Component:Property:Time:System:Scale:Method
- **Identifier**: numeric LOINC code (e.g., 8480-6 for systolic BP)
- **Updates**: biannual (June, December)
- **License**: free under the LOINC license
- **FHIR system URI**: `http://loinc.org`
- **Common gotchas**:
  - The six-part name is the structural truth — match codes on the axes, not on display
  - Same test in different specimens = different LOINC codes
  - Panel codes contain other LOINC codes; capture both panel and components
  - Document codes (in the document-type LOINC set) are used as the C-CDA document `code`
  - RELMA is Regenstrief's mapping tool for local-to-LOINC mapping

---

## RxNorm

- **Maintainer**: NLM (National Library of Medicine)
- **Scope**: clinical drugs in the US, plus relationships to brand names, ingredients, drug classes, and external code systems (NDC, SNOMED CT, ATC, MED-RT)
- **Updates**: weekly
- **License**: free (US, public)
- **FHIR system URI**: `http://www.nlm.nih.gov/research/umls/rxnorm`
- **Term Types (TTYs)**:

| TTY | Meaning |
|-----|---------|
| IN | Ingredient |
| PIN | Precise Ingredient |
| MIN | Multiple Ingredients |
| BN | Brand Name |
| SCD | Semantic Clinical Drug (generic, fully specified — ingredient + strength + dose form) |
| SBD | Semantic Branded Drug |
| SCDF | Semantic Clinical Drug Form |
| SCDG | Semantic Clinical Drug Group |
| SCDC | Semantic Clinical Drug Component |
| GPCK / BPCK | Generic / Brand Pack |

- **US Core (verify version)**: typically requires SCD/SBD/GPCK/BPCK for MedicationRequest.medication
- **API**: RxNav (`https://rxnav.nlm.nih.gov/REST/`)
- **Common gotchas**:
  - Frequent updates mean concept ids can change semantics over weekly releases
  - Pack vs. drug — GPCK/BPCK for combination packs
  - NDC-to-RxNorm mapping is many-to-many; use RxNav `/ndcstatus.json`

---

## NDC (National Drug Code)

- **Full name**: National Drug Code
- **Maintainer**: FDA
- **Scope**: identifies a specific manufacturer's drug package (labeler-product-package)
- **Structure**: 10-digit (with variants 4-4-2, 5-3-2, 5-4-1) or 11-digit padded form
- **Updates**: continuous as manufacturers register
- **License**: free
- **FHIR system URI**: `http://hl7.org/fhir/sid/ndc`
- **Common gotchas**:
  - NDC identifies a **package**, not a concept — for clinical decision support, resolve to RxNorm
  - 10-digit vs. 11-digit forms cause matching errors; canonicalize with leading-zero padding
  - NDCs expire; the FDA Orange/Purple Book lists active products
  - The same drug under different manufacturers has different NDCs

---

## UCUM

- **Full name**: Unified Code for Units of Measure
- **Maintainer**: Regenstrief Institute
- **Scope**: machine-parseable units (e.g., `mg/dL`, `mmol/L`, `10*9/L`, `mm[Hg]`, `[degF]`)
- **License**: free
- **FHIR system URI**: `http://unitsofmeasure.org`
- **Common gotchas**:
  - Brackets distinguish "customary" units (e.g., `[degF]`, `[in_i]`)
  - Multiplication is `.` (period); division is `/`
  - Powers use `**` or curly-brace counts (`10*9` for 10^9, `{cells}/uL`)
  - `mg` vs. `mg/dL` are different units — pass-through validation will catch this
  - Always include `system="http://unitsofmeasure.org"` AND `code` on any quantity

---

## CVX / MVX

- **CVX**: Vaccine Administered code set
  - **Maintainer**: CDC
  - **License**: free
  - **FHIR URI**: `http://hl7.org/fhir/sid/cvx`
- **MVX**: Manufacturer of Vaccine code set
  - **Maintainer**: CDC
  - **License**: free
  - **FHIR URI**: `http://hl7.org/fhir/sid/mvx`
- **Common gotchas**:
  - CVX is the standard for FHIR Immunization.vaccineCode
  - Pair CVX (the vaccine) with MVX (the manufacturer); both are required for full IIS reporting
  - "Unknown" and "no vaccine administered" have specific CVX values — don't reinvent

---

## ICF

- **Full name**: International Classification of Functioning, Disability and Health
- **Maintainer**: WHO
- **Scope**: function, disability, participation, environmental factors
- **Use**: rehab medicine, disability assessment, social work, occupational therapy
- **License**: free (WHO)
- **Common gotchas**:
  - Limited US adoption outside specialty settings
  - Pair with ICD-10-CM (diagnosis) — ICF describes the impact, not the disease

---

## Other Common Code Systems

| System | Use | FHIR URI (verify) |
|--------|-----|------------------|
| ICD-O-3 | Oncology morphology/topography | `http://terminology.hl7.org/CodeSystem/icd-o-3` |
| NDF-RT / MED-RT | Drug classification (US, NLM/VA) | `http://hl7.org/fhir/sid/ndfrt` (legacy) |
| ATC | Drug classification (WHO) | `http://www.whocc.no/atc` |
| HGNC | Gene symbols | `http://www.genenames.org/geneId` |
| dm+d | UK drug dictionary | `https://dmd.nhs.uk` (verify) |
| OPCS-4 | UK procedures | UK-specific |
| TNM | Cancer staging | various |
| HPO | Human Phenotype Ontology (rare disease) | `http://purl.obolibrary.org/obo/hp.owl` |
| HL7 v2 / v3 tables | Code tables embedded in HL7 (e.g., 0001 administrative gender) | `http://terminology.hl7.org/CodeSystem/v2-XXXX` or `v3-XXXX` |

---

## Picking the Right System

| Data | Recommended code system |
|------|------------------------|
| Problem list condition (US) | SNOMED CT |
| Encounter diagnosis (US billing) | ICD-10-CM |
| Outpatient procedure (US professional) | CPT |
| Inpatient procedure (US hospital) | ICD-10-PCS |
| DME / supplies / injectable drugs | HCPCS Level II |
| Lab test code | LOINC |
| Lab value unit | UCUM |
| Vital sign | LOINC code + UCUM unit |
| Medication (clinical) | RxNorm |
| Medication (dispensed/billed package) | NDC |
| Drug allergen | RxNorm |
| Non-drug allergen | SNOMED CT |
| Vaccine administered | CVX |
| Vaccine manufacturer | MVX |
| Disease/finding (international) | ICD-10 (WHO) or ICD-11 |
| Function/disability | ICF |
| Gene | HGNC |
| Documents (typing) | LOINC |

---

## Common Gotchas

- **Code without system** is meaningless — every coding MUST carry `system + code`
- **Display string is informational** — never match on `display`; always match on `system + code`
- **Version drift** breaks value sets — pin `coding.version` and update value sets each release
- **Local codes leak** into "standard" feeds — detect and reject or map them
- **Licensing** matters — CPT, SNOMED CT (in non-member countries), and proprietary terminologies have real legal constraints
- **Same code different system** — NDC `12345-678-90` and an internal ID `12345678` look similar but are not interchangeable
- **Wrong code system for the binding** — using ICD-10-CM where SNOMED CT is required for problem list is the #1 US Core failure
- **Display drift** — the display string for a code can change between versions; the SCTID/code stays stable but the human-readable label shifts
