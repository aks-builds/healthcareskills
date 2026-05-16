---
name: terminology-services
description: When the user wants to choose, map, validate, or serve healthcare code systems and value sets. Use when the user mentions "SNOMED CT," "ICD-10-CM," "ICD-10-PCS," "ICD-11," "CPT," "HCPCS," "LOINC," "RxNorm," "NDC," "UCUM," "CVX," "terminology server," "ValueSet," "ConceptMap," "$expand," "$lookup," "$validate-code," "$translate," "VSAC," "UMLS," "OMOP Athena," "RxNav," or "code mapping." For FHIR resource modeling generally, see fhir-integration. For coding for billing workflow, see medical-coding.
metadata:
  version: 1.0.0
---

# Terminology Services

You are an expert in healthcare terminologies and terminology services. Your goal is to help engineers choose the right code system for a given semantic, bind it correctly in FHIR/CDA/HL7 v2, and operate a terminology server (or consume one) without inventing codes. The rule is absolute: every code you reference in production must be verified against the authoritative source — never fabricate ICD-10-CM, CPT, SNOMED CT, LOINC, RxNorm, or NDC codes.

## Initial Assessment

Read `.agents/healthcare-context.md` (fallback: `.claude/healthcare-context.md`) first. Key items:

- **Country / jurisdiction** — drives which code sets are legally required (e.g., ICD-10-CM is US; other countries use ICD-10 WHO, ICD-10-CA, ICD-10-AM, etc.).
- **Care setting** — ambulatory vs. inpatient vs. payer drives CPT vs. HCPCS vs. ICD-10-PCS.
- **EHR and IGs** — US Core has specific required bindings (e.g., SNOMED CT for problem list, LOINC for labs and vitals, RxNorm for medications).
- **Terminology server / source of truth** in use: VSAC, UMLS, internal terminology server, Snowstorm, Ontoserver, HAPI, etc.

---

## Code Systems Reference

### SNOMED CT

- **Scope**: clinical findings, disorders, procedures, observable entities, body structures, organisms, substances, qualifier values, situations, events, social context.
- **Identifier**: SCTID (Snomed CT Identifier) — long integer with a check digit and partition.
- **Hierarchy**: poly-hierarchical IS-A relationships rooted at 138875005 (SNOMED CT Concept).
- **Concept model**: concepts have descriptions (FSN, synonyms), relationships (IS-A and attribute relationships), and reference-set memberships.
- **Post-coordination**: combine pre-coordinated concepts into expressions (e.g., 73761001 |colonoscopy| : 405813007 |procedure site| = 71854001 |colon|).
- **ECL (Expression Constraint Language)**: query language to define subsets — `<<73761001` returns colonoscopy and descendants.
- **Editions**: International (monthly), US Edition (US-specific extensions, twice yearly), UK Edition, AU, NL, etc.
- **Distribution**: RF2 release format. Licensing via SNOMED International member country licenses (US: NLM provides free use under UMLS license).
- **FHIR system URI**: `http://snomed.info/sct`. Version: `http://snomed.info/sct/{module}/version/{date}`.

### ICD-10-CM (US Diagnoses)

- **Maintainer**: NCHS / CDC.
- **Scope**: diagnoses for outpatient + inpatient encounters in the US.
- **Updates**: annual (Oct 1) with mid-year addenda.
- **FHIR URI**: `http://hl7.org/fhir/sid/icd-10-cm`.

### ICD-10-PCS (US Inpatient Procedures)

- **Maintainer**: CMS.
- **Scope**: inpatient hospital procedure coding (7-character alphanumeric codes).
- **Use**: required for Medicare inpatient billing.
- **FHIR URI**: `http://www.cms.gov/Medicare/Coding/ICD10`.

### ICD-11 (WHO)

- **Maintainer**: WHO.
- **Scope**: international diagnostic classification with linearizations (Mortality and Morbidity Statistics — MMS — among others).
- **Use**: adopted in some countries, not yet in US production billing.

### CPT (Current Procedural Terminology)

- **Maintainer**: American Medical Association (AMA). **Licensed** — using CPT codes commercially requires an AMA license.
- **Scope**: physician procedures and services (Evaluation & Management, Surgery, Radiology, Pathology, Medicine).
- **FHIR URI**: `http://www.ama-assn.org/go/cpt`.

### HCPCS Level II

- **Maintainer**: CMS.
- **Scope**: products, supplies, drugs, and non-physician services (DME, ambulance, injectable drugs, prosthetics). Plus modifiers (e.g., LT/RT for left/right).
- **FHIR URI**: `https://www.cms.gov/Medicare/Coding/MedHCPCSGenInfo`.

### LOINC (Logical Observation Identifiers Names and Codes)

- **Maintainer**: Regenstrief.
- **Scope**: laboratory tests, clinical observations (vitals, surveys, documents, panels).
- **Structure**: each LOINC code has 6 parts — Component, Property, Time, System, Scale, Method. Plus Class, ranking, units.
- **Parts**: addressable identifiers for the axes (Component LP-xxx, etc.).
- **Hierarchy**: multi-axial (search by Class, by Method, etc.).
- **FHIR URI**: `http://loinc.org`.

### RxNorm

- **Maintainer**: NLM.
- **Scope**: clinical drugs in the US, plus relationships to brand names, ingredients, drug classes, and external code systems (NDC, SNOMED CT, ATC).
- **Term Types (TTYs)** to know:

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

US Core 6+ for `MedicationRequest.medicationCodeableConcept` requires RxNorm SCD/SBD/GPCK/BPCK (with TTY rules — check the specific US Core version).

- **FHIR URI**: `http://www.nlm.nih.gov/research/umls/rxnorm`.

### NDC (National Drug Code)

- **Maintainer**: FDA.
- **Scope**: identifies manufacturer's exact drug package (labeler-product-package).
- **Use**: dispensing, billing (with B-segment formats); pair with RxNorm for clinical reasoning.
- **FHIR URI**: `http://hl7.org/fhir/sid/ndc`.

### UCUM (Unified Code for Units of Measure)

- **Maintainer**: Regenstrief.
- **Scope**: machine-parseable units (e.g., `mg/dL`, `mmol/L`, `10*9/L`, `mm[Hg]`, `[degF]`).
- **FHIR URI**: `http://unitsofmeasure.org`.
- **Rule**: prefer UCUM over free-text units everywhere a quantity is recorded.

### CVX (Vaccine Administered)

- **Maintainer**: CDC.
- **Scope**: vaccines administered (paired with MVX manufacturer codes).
- **FHIR URI**: `http://hl7.org/fhir/sid/cvx`.

### ICF (International Classification of Functioning, Disability and Health)

- **Maintainer**: WHO.
- **Scope**: function, disability, participation. Used in rehab, disability, social work.

### Other Common Code Systems

| System | Use | URI |
|--------|-----|-----|
| ICD-O-3 | Oncology morphology/topography | `http://terminology.hl7.org/CodeSystem/icd-o-3` |
| NDF-RT / DrugBank / ATC | Drug classification | various |
| HGNC | Gene symbols | `http://www.genenames.org/geneId` |
| SOP / SNOMED VMP / dm+d (UK) | Drugs outside US | various |
| HCPCS-II modifiers | Two-character billing modifiers | same as HCPCS |

---

## Where Each Code System Lives in FHIR (US Core baseline)

| Data | Binding (typical) |
|------|-------------------|
| `Condition.code` (problem list) | SNOMED CT (US Core) |
| `Condition.code` (encounter dx) | ICD-10-CM (US Core encounter-diagnosis) |
| `Observation.code` (lab, vital, social hx) | LOINC |
| `Observation.valueCodeableConcept` | SNOMED CT (typical) |
| `Observation.valueQuantity.code` | UCUM |
| `Procedure.code` | SNOMED CT or CPT or HCPCS (depending on profile) |
| `Procedure.code` (inpatient) | ICD-10-PCS or CPT |
| `MedicationRequest.medicationCodeableConcept` | RxNorm |
| `Immunization.vaccineCode` | CVX |
| `AllergyIntolerance.code` | SNOMED CT or RxNorm (for drug allergens) |
| `Encounter.type` | CPT E/M or SNOMED CT |

Always confirm the binding strength (required vs. extensible) in the specific IG version in scope.

---

## FHIR Terminology Operations

The FHIR Terminology Service spec defines operations on `CodeSystem`, `ValueSet`, and `ConceptMap`:

| Operation | Purpose | Example |
|-----------|---------|---------|
| `$lookup` | Get display + properties for a code | `GET /CodeSystem/$lookup?system=http://loinc.org&code=85354-9` |
| `$validate-code` | Check a code is valid in a system or value set | `GET /ValueSet/{id}/$validate-code?code=8480-6&system=http://loinc.org` |
| `$expand` | Expand a value set to a flat list of codes | `GET /ValueSet/{id}/$expand?filter=blood&count=20` |
| `$translate` | Map a code via a `ConceptMap` | `GET /ConceptMap/$translate?source=...&code=I10&system=http://hl7.org/fhir/sid/icd-10-cm&target=http://snomed.info/sct` |
| `$subsumes` | Is code A subsumed by code B? | `GET /CodeSystem/$subsumes?system=http://snomed.info/sct&codeA=...&codeB=...` |
| `$closure` | Compute transitive closure for a set of codes | server-only operation, less common in clients |

`ValueSet.expansion.contains` returns expanded concepts; `ValueSet.compose` defines the rules (include/exclude by system, by filter, by concept list).

`ConceptMap.group.element.target` carries mappings with `equivalence` (`equivalent`, `equal`, `wider`, `subsumes`, `narrower`, `specializes`, `inexact`, `unmatched`, `disjoint`) — preserve equivalence in your code, do not silently treat `wider` as `equal`.

---

## Public Terminology Sources

| Source | What's there |
|--------|--------------|
| **VSAC (Value Set Authority Center, NLM)** | CMS / ONC value sets for quality measures, CDS, USCDI |
| **UMLS Metathesaurus (NLM)** | Cross-system mappings across 200+ source vocabularies |
| **RxNav (NLM)** | RxNorm browser + APIs (`/REST/rxcui.json`, `/REST/ndcstatus.json`, `/REST/interaction/interaction.json`) |
| **OMOP Athena (OHDSI)** | OMOP CDM concept vocabulary downloads, with standard/non-standard mappings |
| **NCI Thesaurus / NCIm / NCImeta** | Oncology and biomedical |
| **LOINC search / Regenstrief** | Authoritative LOINC search |
| **SNOMED International Browser / Snowstorm** | Authoritative SNOMED CT browse and ECL evaluation |
| **NLM SNOMED CT browser (US Edition)** | US-edition browser |

VSAC and UMLS require a free UMLS license (NLM); SNOMED CT requires being in a member country (NLM provides US access under UMLS).

---

## Terminology Server Options

Open-source and commercial:

| Server | Notes |
|--------|-------|
| Ontoserver (CSIRO) | Commercial; SNOMED-aware, full FHIR terminology service |
| Snowstorm (SNOMED International) | Open source; SNOMED CT-focused; FHIR + native APIs |
| HAPI FHIR JPA terminology | OSS; loads LOINC, SNOMED CT, ICD-10-CM, RxNorm with effort |
| HL7 Terminology Service / tx.fhir.org | Public reference server (do not use for PHI) |

Plan for substantial RAM and disk: a full SNOMED CT + LOINC + ICD-10-CM + RxNorm load is many GB.

---

## Mapping Pitfalls

- **1:N mappings** — one source code maps to several target codes (e.g., one ICD-10-CM diagnosis to several SNOMED CT findings depending on specificity). Decide and document the disambiguation policy.
- **Lossy mappings** — ICD-10-CM `I10 Essential (primary) hypertension` ≈ SNOMED CT `59621000 |Essential hypertension|`, but ICD-10-CM `R03.0` (elevated BP without dx of HTN) is **not** the same concept. Do not equate.
- **Version drift** — ICD-10-CM updates yearly; RxNorm weekly; SNOMED CT twice yearly (US). Pin the version on every record (`coding.version`).
- **Local code creep** — vendor "local" codes leak into integrations; map them or reject them, do not pass through unnoted.
- **Display strings as identifiers** — never match on `display`; always match on `system + code`.
- **Post-coordination round-tripping** — many systems cannot store SNOMED CT expressions; collapse to pre-coordinated equivalents only when safe.
- **UCUM mistakes** — `mg` and `mg/dL` are different units; reject quantities without `system="http://unitsofmeasure.org"` and a valid `code`.

---

## Worked Example: Validating a Lab Result

`Observation.code` = `85354-9 |Blood pressure panel with all children optional|` from LOINC.
`Observation.component[i].code` for systolic = `8480-6`, diastolic = `8462-4`.
`Observation.component[i].valueQuantity.code` = `mm[Hg]` (UCUM).

To validate:

```
GET /ValueSet/$validate-code?
  url=http://hl7.org/fhir/us/core/ValueSet/us-core-vital-signs
  &system=http://loinc.org
  &code=8480-6
```

Response (synthetic):
```json
{ "resourceType": "Parameters", "parameter": [
  { "name": "result", "valueBoolean": true },
  { "name": "display", "valueString": "Systolic blood pressure" }
] }
```

---

## Common Pitfalls

- Storing a `code` without a `system` URI — the code is meaningless without the system.
- Storing free-text display strings instead of codes ("flu shot" instead of CVX `141`).
- Mixing CPT and ICD-10-PCS for the same procedure across records — pick a binding per setting.
- Treating NDC as clinical-decision-grade — NDCs are package codes, not concepts; resolve to RxNorm for CDS.
- Forgetting to update value sets when a new ICD-10-CM annual release ships — quality measures break.

---

## Task-Specific Questions

1. Which code system(s) are in scope — SNOMED CT, ICD-10-CM, ICD-10-PCS, ICD-11, CPT, HCPCS, LOINC, RxNorm, NDC, UCUM, CVX, ICF, or others?
2. Which specific versions and editions are pinned (e.g., SNOMED CT US Edition release date, ICD-10-CM FY year, LOINC release, RxNorm weekly release, US Core IG version)?
3. Where do value sets come from — VSAC, UMLS, a vendor-provided pack, an internal terminology team, the IG's compose, or ad-hoc?
4. Are mappings needed — and if so which source→target pairs (e.g., ICD-10-CM → SNOMED CT, NDC → RxNorm, local code → standard)?
5. What terminology server are we consuming or operating — Ontoserver, Snowstorm, HAPI FHIR JPA terminology, tx.fhir.org, internal — and is it the source of truth or a cache?
6. Which FHIR terminology operations are required ($lookup, $validate-code, $expand, $translate, $subsumes), and at what request volume?
7. Are there licensing constraints (CPT/AMA, SNOMED non-member country, proprietary vocabularies) that affect what we can ship or expose?

---

## Related Skills

- **fhir-integration** — bindings live in profiles; this skill explains the systems behind those bindings
- **medical-coding** — coding workflow for revenue cycle (CPT, HCPCS, ICD-10-CM/PCS modifiers)
- **cda-ccda** — section bindings (problem list = SNOMED CT, results = LOINC, etc.)
- **hl7-v2** — OBX-3 (LOINC), DG1 (ICD-10-CM), RXE/RXC (RxNorm/NDC) field bindings
- **clinical-decision-support** — value sets and ECL drive CDS Hooks logic
- **public-health-reporting** — VSAC value sets drive eCQM and case reporting
- **ehr-integration** — vendor terminology mapping layers (e.g., Epic's master code tables)
