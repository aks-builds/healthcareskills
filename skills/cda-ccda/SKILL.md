---
name: cda-ccda
description: When the user wants to author, parse, validate, or transform Clinical Document Architecture (CDA) or Consolidated CDA (C-CDA) documents. Use when the user mentions "CDA," "C-CDA," "CCD," "Continuity of Care Document," "Discharge Summary," "Operative Note," "Progress Note," "Consultation Note," "History and Physical," "Care Plan," "Transfer Summary," "Referral Note," "Procedure Note," "templateId," "narrative block," "Schematron," "NIST CDA Validator," "ETSA," "C-CDA scorecard," "CDA-on-FHIR," or "FHIR-to-CDA." For non-document FHIR data, see fhir-integration. For HL7 v2 documents transported via MDM, see hl7-v2.
metadata:
  version: 1.0.0
---

# CDA and C-CDA

You are an expert in HL7 Clinical Document Architecture (CDA Release 2) and Consolidated CDA (C-CDA) — the XML-based clinical document format mandated by ONC for US EHR certification and widely used for transitions of care. Your goal is to help engineers author, validate, and transform these documents correctly. Never invent templateIds, OIDs, or codes — verify against the published C-CDA Implementation Guide (HL7 C-CDA R2.1 + Companion Guide) when uncertain.

## Initial Assessment

Check `.agents/healthcare-context.md` (fallback: `.claude/healthcare-context.md`) first. Relevant sections:

- **C-CDA version**: R2.1 is the ONC-required baseline; R3.0 is published; Companion Guide updates are frequent.
- **Document types** in scope (CCD vs. Discharge Summary vs. Referral Note, etc.).
- **EHR vendor** — Epic, Oracle Health (Cerner), Meditech, Athenahealth all generate C-CDAs with vendor quirks.
- **USCDI version** in scope (v3/v4) — drives which data classes must appear.
- **Direction**: producing, consuming, or transforming (CDA↔FHIR).

---

## CDA Document Structure

A CDA document is a single XML file with two halves:

```xml
<ClinicalDocument xmlns="urn:hl7-org:v3">
  <!-- Header -->
  <typeId root="2.16.840.1.113883.1.3" extension="POCD_HD000040"/>
  <templateId root="2.16.840.1.113883.10.20.22.1.1" extension="2015-08-01"/> <!-- US Realm Header -->
  <templateId root="2.16.840.1.113883.10.20.22.1.2" extension="2015-08-01"/> <!-- CCD -->
  <id root="..."/>
  <code code="34133-9" codeSystem="2.16.840.1.113883.6.1" displayName="Summarization of Episode Note"/>
  <title>Continuity of Care Document</title>
  <effectiveTime value="20260301120000-0500"/>
  <confidentialityCode code="N" codeSystem="2.16.840.1.113883.5.25"/>
  <languageCode code="en-US"/>
  <recordTarget>...</recordTarget>
  <author>...</author>
  <custodian>...</custodian>

  <!-- Body -->
  <component>
    <structuredBody>
      <component>
        <section>...</section>
      </component>
      <!-- more sections -->
    </structuredBody>
  </component>
</ClinicalDocument>
```

### Header

The header carries document metadata and participants:

| Element | Role |
|---------|------|
| `recordTarget` | The patient |
| `author` | The person/system who authored the document |
| `dataEnterer` | Transcriptionist or scribe |
| `informant` | Source of information (clinician, patient, family) |
| `custodian` | Organization responsible for the document |
| `legalAuthenticator` / `authenticator` | Signing/co-signing clinicians |
| `participant` | Other participants (caregivers, emergency contacts) |
| `documentationOf` | Service event(s) the document is about |
| `componentOf` | Encompassing encounter |

### Body — Sections and Entries

Each section has:

1. **Narrative block** (`<text>`) — human-readable HTML-like markup. This is the *legal record*.
2. **Structured entries** (`<entry>`) — machine-readable codified data.

Both must convey the same clinical meaning. If they disagree, narrative wins legally, but downstream systems will trust entries.

---

## TemplateIds

Every CDA document, section, and entry is identified by one or more `<templateId>` elements (OID + optional extension date). TemplateIds chain — for example, a Medication Activity entry will assert both the generic Substance Administration template and the C-CDA-specific medication activity template.

When validating, the document's templateIds tell the validator which Schematron / FHIR profile to apply. Missing templateIds is the most common cause of failed validation.

---

## C-CDA Document Types (R2.1)

| Document Type | LOINC Code | Use |
|---------------|------------|-----|
| Continuity of Care Document (CCD) | 34133-9 | Patient summary for handoffs |
| Discharge Summary | 18842-5 | End of inpatient stay |
| Operative Note | 11504-8 | Surgical procedure narrative |
| Progress Note | 11506-3 | Inpatient daily note |
| Consultation Note | 11488-4 | Specialist consult |
| History and Physical | 34117-2 | H&P |
| Referral Note | 57133-1 | Specialist referral |
| Care Plan | 52521-2 | Care plan document |
| Transfer Summary | 18761-7 | Transfer to another facility |
| Procedure Note | 28570-0 | Non-surgical procedure |
| Diagnostic Imaging Report | 18748-4 | Radiology report (often DI document) |
| Unstructured Document | 34133-9 (various) | Scanned / PDF wrapped |

CCD is the workhorse for HIE and TEFCA exchange — when in doubt, default to CCD.

---

## Common Sections (with required/recommended status in CCD)

| Section | LOINC | Status in CCD |
|---------|-------|---------------|
| Allergies and Intolerances | 48765-2 | Required |
| Medications | 10160-0 | Required |
| Problem List | 11450-4 | Required |
| Procedures | 47519-4 | Required (R2.1 v3 v4) |
| Results | 30954-2 | Required |
| Vital Signs | 8716-3 | Required |
| Encounters | 46240-8 | Recommended |
| Plan of Treatment | 18776-5 | Recommended |
| Immunizations | 11369-6 | Recommended |
| Social History | 29762-2 | Recommended |
| Functional Status | 47420-5 | Recommended |
| Mental Status | 10190-7 | Recommended |
| Health Concerns | 75310-3 | Recommended |
| Goals | 61146-7 | Recommended |
| Assessment | 51848-0 | Recommended |
| Assessment and Plan | 51847-2 | Alternative to separate Assessment + Plan |
| Reason for Referral | 42349-1 | Required in Referral Note |
| Hospital Course | 8648-8 | Required in Discharge Summary |

Each section has an "Entries Required" and an "Entries Optional" variant — pick based on whether you have structured data.

---

## Structured Entries — Common Templates

| Domain | Entry Template (R2.1) |
|--------|------------------------|
| Allergy | Allergy - Intolerance Observation, Allergy Concern Act |
| Medication | Medication Activity, Medication Information |
| Problem | Problem Observation, Problem Concern Act |
| Procedure | Procedure Activity Procedure / Observation / Act |
| Result | Result Organizer, Result Observation |
| Vital Sign | Vital Signs Organizer, Vital Sign Observation |
| Encounter | Encounter Activity |
| Immunization | Immunization Activity, Immunization Refusal Reason |
| Plan of Treatment | Planned Observation / Procedure / Medication / Act / Encounter |
| Social History | Social History Observation, Smoking Status — Meaningful Use, Birth Sex |
| Functional Status | Functional Status Observation, Functional Status Result Observation |
| Health Concern | Health Concern Act |
| Goal | Goal Observation |

Each entry has required coded elements (typically with SNOMED CT, LOINC, RxNorm, ICD-10-CM, or CVX bindings) — see the C-CDA value set catalog for the authoritative bindings.

---

## Narrative Block Rules

- Narrative `<text>` mirrors the entries.
- Reference entries from narrative using `<text><reference value="#allergy-1"/></text>` and `<entry><...><reference value="#allergy-1"/>...</></entry>`.
- The narrative must render without external dependencies (no scripts, external CSS).
- Tables in narrative use CDA-flavored markup (`<table>`, `<thead>`, `<tbody>`, `<tr>`, `<td>`).

Receivers MUST render the narrative; senders MUST ensure narrative and entries match.

---

## Validation

Several layers, each catching different errors:

| Layer | What it catches |
|-------|-----------------|
| XML well-formedness | Parser-level errors |
| CDA XSD schema | Element structure |
| C-CDA Schematron (HL7-published) | Template conformance (cardinality, value set bindings) |
| NIST CDA Validator / Lantana ETSA | C-CDA conformance + Meaningful Use / ONC rules |
| Vendor-specific Schematron | EHR-specific requirements (Epic, Cerner, etc.) |
| C-CDA Scorecard (ONC) | Best-practice scoring — usability beyond conformance |

A document can pass schema and Schematron yet still be clinically poor — scorecard helps catch under-coded narratives, missing reconciled lists, and brittle templates.

---

## Common C-CDA Errors

| Symptom | Root cause |
|---------|------------|
| "templateId for section not asserted" | Missing or misspelled OID/extension |
| "code SHALL be selected from ValueSet X" | Wrong code system or out-of-band code |
| Narrative and entry diverge | Generator built them in separate passes without cross-references |
| Missing `<reference>` from entry to narrative | Breaks rendering pipelines |
| Wrong `effectiveTime` precision | Section requires datetime, entry only has date (or vice versa) |
| `<originalText><reference value="#x"/></originalText>` pointing to nonexistent narrative ID | Validator passes only if reference resolves |
| `<unsupportedTrait>nullFlavor="UNK"</...>` instead of allowed `nullFlavor` per cardinality | `nullFlavor` allowed only where IG permits |

---

## Transformation: C-CDA ↔ FHIR

Two HL7-published bridges:

- **CDA-on-FHIR** — represents CDA constructs using FHIR resources (still feels CDA-shaped).
- **FHIR-to-CDA** Implementation Guide — defines how to construct C-CDA documents from FHIR resources.

For US Core ↔ C-CDA mappings, ONC's HL7 C-CDA ↔ US Core Mappings (companion documents) is the canonical reference. Common landings:

| FHIR resource | C-CDA target |
|---------------|--------------|
| Patient | recordTarget |
| Practitioner / PractitionerRole | author / authenticator / participant |
| Encounter | componentOf / encounters section |
| AllergyIntolerance | Allergy - Intolerance Observation |
| MedicationStatement / MedicationRequest | Medication Activity |
| Condition (problem-list) | Problem Observation in Problem Concern Act |
| Procedure | Procedure Activity Procedure |
| Observation (lab) | Result Observation in Result Organizer |
| Observation (vital signs) | Vital Sign Observation in Vital Signs Organizer |
| Immunization | Immunization Activity |
| CarePlan / Goal | Plan of Treatment / Goal Observation |

Round-tripping is lossy — design integrations with the canonical format in mind.

---

## USCDI Alignment

USCDI v3 and v4 enumerate data classes (e.g., Patient Summary, Medications, Problems, Procedures, Health Concerns, Goals, SDOH, Care Team Members, Encounter Information, Clinical Notes, Labs, Vital Signs, Immunizations). Each class lands in specific C-CDA sections; CCD is the document type the ONC certification criteria require an EHR to be able to produce on demand.

When auditing a C-CDA for USCDI compliance, walk class-by-class and confirm each is represented either as a structured entry or, at minimum, in narrative.

---

## Minimal CCD Skeleton (synthetic)

```xml
<ClinicalDocument xmlns="urn:hl7-org:v3">
  <typeId root="2.16.840.1.113883.1.3" extension="POCD_HD000040"/>
  <templateId root="2.16.840.1.113883.10.20.22.1.1" extension="2015-08-01"/>
  <templateId root="2.16.840.1.113883.10.20.22.1.2" extension="2015-08-01"/>
  <id root="2.16.840.1.113883.19.5.99999.1" extension="DOC-00001"/>
  <code code="34133-9" codeSystem="2.16.840.1.113883.6.1" displayName="Summarization of Episode Note"/>
  <title>Continuity of Care Document</title>
  <effectiveTime value="20260301120000-0500"/>
  <confidentialityCode code="N" codeSystem="2.16.840.1.113883.5.25"/>
  <languageCode code="en-US"/>
  <recordTarget>
    <patientRole>
      <id root="2.16.840.1.113883.19.5.99999.2" extension="MRN-123456"/>
      <patient>
        <name><given>Jane</given><family>Doe</family></name>
        <administrativeGenderCode code="F" codeSystem="2.16.840.1.113883.5.1"/>
        <birthTime value="19850203"/>
      </patient>
    </patientRole>
  </recordTarget>
  <!-- author, custodian, body omitted for brevity -->
</ClinicalDocument>
```

OIDs shown are illustrative — replace with your organization's registered OIDs and never reuse another organization's root OID.

---

## Common Pitfalls

- Building documents that pass XSD but fail Schematron — always validate against the published C-CDA Schematron.
- Encoding clinical content only in narrative — downstream parsers cannot reconcile.
- Using wrong value sets (e.g., ICD-10-CM where SNOMED CT is required for problem list in CCD).
- Sending unstructured PDFs wrapped as C-CDA — legal under ONC but defeats the point.
- Skipping `componentOf` for documents tied to an encounter.
- Letting EHRs ship z-extensions that other consumers ignore.

---

## Task-Specific Questions

1. Which document type are we authoring or consuming — CCD, Discharge Summary, Referral Note, Operative Note, H&P, Consultation Note, Progress Note, Care Plan, Transfer Summary, or Procedure Note?
2. What is the source EHR (Epic, Oracle Health/Cerner, Meditech, Athenahealth, etc.) and which of its quirks should we expect?
3. Which C-CDA template version applies — R2.1 (the ONC baseline) or R3.0 — and which Companion Guide revision?
4. Which USCDI version (v3 / v4 / v5) drives the required data classes, and is this CCD intended for ONC certification or TEFCA exchange?
5. What validation tooling is in scope — XSD only, HL7 C-CDA Schematron, NIST CDA Validator, Lantana ETSA, vendor Schematron, ONC C-CDA Scorecard?
6. Is FHIR mapping part of the deliverable — CDA-on-FHIR, FHIR-to-CDA IG, or US Core ↔ C-CDA round-tripping — and what direction(s)?
7. How is the document transported (XDS.b, XDR, MHD, MDM^T02, Direct messaging, DocumentReference attachment)?

---

## Related Skills

- **fhir-integration** — `DocumentReference` carries C-CDAs by reference or as base64 attachments; CDA-on-FHIR
- **hl7-v2** — `MDM^T02` and `MDM^T08` transport C-CDAs in OBX-5 (typically base64 inside `ED`)
- **terminology-services** — value set bindings (SNOMED, LOINC, RxNorm, CVX) for C-CDA sections
- **ihe-profiles** — XDS.b / XDR / XDM / MHD push and pull C-CDAs across the network
- **tefca-hie** — TEFCA exchange uses C-CDA today and increasingly FHIR
- **ehr-integration** — vendor-specific C-CDA generation endpoints and known quirks
- **audit-logging** — disclosure events for clinical documents
