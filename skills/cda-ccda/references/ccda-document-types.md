# C-CDA Document Types

Quick reference to the document types defined in Consolidated CDA Release 2.1. **All templateId values listed below must be verified against the current C-CDA IG release** before use — OID extensions are date-tagged and update with each Companion Guide.

## Contents

- How TemplateIds Work at the Document Level
- US Realm Header
- CCD (Continuity of Care Document)
- Discharge Summary
- Operative Note
- History and Physical
- Consultation Note
- Progress Note
- Procedure Note
- Referral Note
- Transfer Summary
- Care Plan
- Diagnostic Imaging Report
- Unstructured Document
- Document-Level Required Elements
- Choosing the Right Document Type
- Verifying the IG Version

---

## How TemplateIds Work at the Document Level

A C-CDA document carries one or more `<templateId>` elements at the `<ClinicalDocument>` level:

```xml
<typeId root="2.16.840.1.113883.1.3" extension="POCD_HD000040"/>
<templateId root="2.16.840.1.113883.10.20.22.1.1" extension="..."/> <!-- US Realm Header -->
<templateId root="2.16.840.1.113883.10.20.22.1.2" extension="..."/> <!-- CCD -->
```

Convention is to chain template assertions: the generic US Realm Header is always present, plus the specific document type's templateId. **The `extension` (date suffix) changes between C-CDA versions** — always pin to your IG's published value.

The document's `<code>` (typically a LOINC code) classifies the document type for catalogs, registries, and XDS metadata.

---

## US Realm Header

- **Template root**: `2.16.840.1.113883.10.20.22.1.1`
- **Extension**: verify against current C-CDA 2.1 (typically a date in YYYY-MM-DD form like `2015-08-01` or later in revisions)
- **Use**: every C-CDA US Realm document chains this template. Sets US-specific required header elements (recordTarget, author, custodian, etc.).

---

## CCD (Continuity of Care Document)

- **Template root**: `2.16.840.1.113883.10.20.22.1.2`
- **LOINC document code**: `34133-9` Summarization of Episode Note
- **Use**: patient summary for transitions of care, HIE exchange, TEFCA, ONC certification.
- **Required sections (C-CDA 2.1)**: Allergies and Intolerances, Medications, Problem List, Procedures, Results, Vital Signs
- **Commonly recommended sections**: Encounters, Plan of Treatment, Immunizations, Social History, Functional Status, Mental Status, Reason for Referral, Family History, Health Concerns, Goals
- **Notes**: CCD is the workhorse — when in doubt, this is the document type.

---

## Discharge Summary

- **Template root**: `2.16.840.1.113883.10.20.22.1.8`
- **LOINC document code**: `18842-5` Discharge Summary
- **Use**: end of inpatient stay handoff.
- **Required sections**: Allergies, Medications (often Discharge Medications), Hospital Course, Hospital Discharge Diagnosis, Plan of Treatment, Reason for Visit, Problem List, Procedures, Results, Vital Signs (verify exact required list per IG)
- **Notes**: Hospital Course (LOINC `8648-8`) is the section most distinctive to this document type.

---

## Operative Note

- **Template root**: `2.16.840.1.113883.10.20.22.1.7`
- **LOINC document code**: `11504-8` Surgical Operation Note
- **Use**: narrative + structured record of a surgical procedure.
- **Required sections**: Anesthesia, Complications, Estimated Blood Loss, Operative Note Surgical Procedure, Postoperative Diagnosis, Preoperative Diagnosis, Procedure Description, Procedure Findings (verify per IG)
- **Notes**: typically authored by the operating physician.

---

## History and Physical

- **Template root**: `2.16.840.1.113883.10.20.22.1.3`
- **LOINC document code**: `34117-2` History and Physical Note
- **Use**: admission / pre-procedure H&P.
- **Required sections**: Allergies, Chief Complaint and/or Reason for Visit, Family History, History of Present Illness, Medications, Past Medical History, Physical Exam, Review of Systems, Social History, Vital Signs (verify per IG)

---

## Consultation Note

- **Template root**: `2.16.840.1.113883.10.20.22.1.4`
- **LOINC document code**: `11488-4` Consult Note
- **Use**: specialist consultation findings and recommendations.
- **Required sections**: Allergies, Assessment / Assessment and Plan, History of Present Illness, Medications, Physical Exam, Reason for Referral (verify per IG)

---

## Progress Note

- **Template root**: `2.16.840.1.113883.10.20.22.1.9`
- **LOINC document code**: `11506-3` Progress Note
- **Use**: inpatient daily note.
- **Required sections**: Assessment / Assessment and Plan (verify per IG)
- **Notes**: lightweight document; many sections are optional.

---

## Procedure Note

- **Template root**: `2.16.840.1.113883.10.20.22.1.6`
- **LOINC document code**: `28570-0` Procedure Note
- **Use**: non-surgical procedure narrative (e.g., endoscopy, biopsy).
- **Required sections**: Procedure Description, Procedure Indications, Procedure Findings (verify per IG)

---

## Referral Note

- **Template root**: `2.16.840.1.113883.10.20.22.1.14`
- **LOINC document code**: `57133-1` Referral Note
- **Use**: outbound referral to a specialist or facility.
- **Required sections**: Reason for Referral (LOINC `42349-1`), plus typically Allergies, Medications, Problem List, etc. (verify per IG)

---

## Transfer Summary

- **Template root**: `2.16.840.1.113883.10.20.22.1.13`
- **LOINC document code**: `18761-7` Transfer Summary Note
- **Use**: transfer to another care setting / facility.
- **Required sections**: Allergies, Medications, Problem List, Reason for Referral, Plan of Treatment (verify per IG)

---

## Care Plan

- **Template root**: `2.16.840.1.113883.10.20.22.1.15`
- **LOINC document code**: `52521-2` Overall Plan of Care/Advance Care Directives
- **Use**: longitudinal patient care plan including goals, interventions, health concerns.
- **Required sections**: Health Concerns, Goals, Interventions, Outcomes (verify per IG)

---

## Diagnostic Imaging Report

- **Template root**: `2.16.840.1.113883.10.20.6.1.1`
- **LOINC document code**: `18748-4` Diagnostic Imaging Report (or a more specific LOINC per modality)
- **Use**: radiology report.
- **Required sections**: Findings, Impression (verify per IG)
- **Notes**: DIR has its own IG and template family (DIR IG); the basic structure is CDA but the imaging-specific templates extend it.

---

## Unstructured Document

- **Template root**: `2.16.840.1.113883.10.20.22.1.10`
- **LOINC document code**: varies — use a LOINC that describes the wrapped content
- **Use**: wrap a non-CDA artifact (PDF, scanned image) as a CDA so it can flow through CDA-aware pipelines.
- **Notes**: legal under ONC but defeats the structured-exchange purpose. Prefer a proper C-CDA document whenever feasible.

---

## Document-Level Required Elements

Regardless of document type, every C-CDA 2.1 document MUST contain:

| Element | Notes |
|---------|-------|
| `<typeId root="2.16.840.1.113883.1.3" extension="POCD_HD000040"/>` | Identifies as CDA Release 2 |
| `<templateId>` for US Realm Header | Plus document-type templateId |
| `<id root="..." extension="..."/>` | Document identifier (your OID + extension) |
| `<code>` | LOINC document type code |
| `<title>` | Human-readable title |
| `<effectiveTime value="YYYYMMDDhhmmss[+/-ZZZZ]"/>` | Document creation time, with timezone |
| `<confidentialityCode code="N|R|V" codeSystem="2.16.840.1.113883.5.25"/>` | Confidentiality level |
| `<languageCode code="en-US"/>` | Document language |
| `<recordTarget>` | The patient (must have id, name, gender, birthTime) |
| `<author>` | Person and/or device |
| `<custodian>` | Organization responsible for the document |
| `<component><structuredBody>` | The document body |

Often required depending on context:

- `<dataEnterer>`, `<informant>`
- `<legalAuthenticator>`, `<authenticator>`
- `<participant>` (caregivers, emergency contacts)
- `<documentationOf>` (service event)
- `<componentOf>` (encompassing encounter — required when document is tied to an encounter)

---

## Choosing the Right Document Type

| Use case | Recommended document |
|----------|---------------------|
| Cross-organization patient summary | CCD |
| End of hospital stay | Discharge Summary |
| Outpatient handoff to specialist | Referral Note |
| Specialist's report back | Consultation Note |
| Surgical record | Operative Note |
| Diagnostic procedure record | Procedure Note |
| Admission record | History and Physical |
| Daily inpatient note | Progress Note |
| Transfer between facilities | Transfer Summary |
| Longitudinal care plan | Care Plan |
| Radiology report | Diagnostic Imaging Report (DIR IG) |
| Wrapping a PDF | Unstructured Document (last resort) |

---

## Verifying the IG Version

C-CDA evolves through:

- **C-CDA Release 2.1** — ONC's certification baseline. Published 2015 with Companion Guide revisions.
- **C-CDA Companion Guide** — clarifies and adds guidance; revised periodically. Pin to the version your trading partner expects.
- **C-CDA Release 3.0** — published. Adoption is gradual; verify per program (ONC certification primarily on R2.1 + Companion Guide; some HIEs / programs accept R3.0).

When you write or consume C-CDAs:

1. Confirm the trading partner's IG version + Companion Guide revision.
2. Pin templateId extensions to that version.
3. Validate against the Schematron published for that exact version.
4. Run the C-CDA Scorecard for quality scoring.

Names and OIDs in this reference are conceptual; the IG is authoritative.
