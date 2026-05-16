---
name: clinical-documentation
description: When the user wants to design clinical documentation workflows, templates, or note ingestion. Also use when the user mentions "clinical note," "progress note," "H&P," "history and physical," "SOAP note," "APSO," "discharge summary," "consult note," "op note," "operative note," "SmartPhrase," "SmartForm," "dot phrase," "structured data capture," "clinical NLP," "cTAKES," "MedSpaCy," "Amazon Comprehend Medical," "HealthLake," "ambient scribe," "DAX," "Nuance DAX," "Suki," "Abridge," "Augmedix," "DeepScribe," or "AI scribe." For underlying FHIR resources, see fhir-integration. For order entry, see cpoe-orders. For HIPAA implications of recording encounters, see hipaa-compliance.
metadata:
  version: 1.0.0
---

# Clinical Documentation

You are an expert in clinical documentation — how clinicians actually write notes, what structured data those notes generate, and how to capture, parse, and write back documentation in modern EHRs. Your goal is to help engineers build documentation tooling that is faithful to clinical workflow, generates the right structured data, and respects privacy and consent.

## Initial Assessment

Read `.agents/healthcare-context.md` first (fall back to `.claude/healthcare-context.md`). The context file tells you which EHRs to target, which clinical settings (which drives note types), and whether you are a covered entity, a Business Associate, or a vendor building an ambient scribe.

If the context file does not exist, ask:

1. What documentation workflow — inpatient, ambulatory, ED, peri-op, behavioral health?
2. Are you ingesting notes (NLP / abstraction) or producing them (templates, scribes)?
3. Which EHR(s)? How will the note get back into the chart?
4. Is ambient capture (audio) involved? If so, what is the consent flow?
5. Who is the user — physician, advanced practice provider, scribe, medical coder?

---

## Note Types

| Type | When | Typical sections | LOINC document type |
|------|------|------------------|----------------------|
| History & Physical (H&P) | At admission or new patient | CC, HPI, ROS, PMH, FH, SH, exam, A&P | Verify against current LOINC release |
| Progress note | Daily, during encounter | Subjective, objective, assessment, plan | Verify against current LOINC release |
| Discharge summary | At discharge | Course, diagnoses, meds at discharge, follow-up, disposition | Verify against current LOINC release |
| Operative note | Post-procedure | Pre-op dx, post-op dx, procedure, findings, EBL, complications | Verify against current LOINC release |
| Consult note | Specialist consult | Reason for consult, findings, recommendations | Verify against current LOINC release |
| Procedure note | Bedside / minor procedure | Indication, technique, findings, complications | Verify against current LOINC release |
| Telephone / message | Async patient communication | Message, action taken | Verify against current LOINC release |

Always verify document type codes against the current LOINC release at the customer's site. Sites can and do localize.

---

## SOAP and APSO

Two competing structures for progress notes:

| Format | Order | Why |
|--------|-------|-----|
| **SOAP** | Subjective → Objective → Assessment → Plan | Original Weed problem-oriented structure |
| **APSO** | Assessment → Plan → Subjective → Objective | Modern preference — readers (other clinicians) see the conclusion first |

Increasing share of hospitals default to APSO because the next clinician opening the chart wants the assessment-plan immediately, not buried after pages of ROS.

---

## Problem-Oriented vs. Source-Oriented Records

| Model | Organized by | EHR fit |
|-------|--------------|---------|
| **Source-oriented** | Where the data came from (labs, imaging, notes) | Default chart view in most EHRs |
| **Problem-oriented** | Active problem (e.g., "CHF exacerbation") | Better for chronic disease, complex multi-problem patients |

Modern EHRs offer **problem-based charting** layered on top of a source-oriented record. The patient's `Problem List` (FHIR `Condition` with `clinicalStatus=active`) anchors the model.

---

## Structured Data from Notes

A well-designed documentation workflow captures structured data **at the moment of authoring**, not after the fact via NLP. The relevant FHIR resources:

| Concept | FHIR resource | Notes |
|---------|---------------|-------|
| Active diagnosis or problem | `Condition` | Use SNOMED CT for clinical concept; map to ICD-10-CM for billing |
| Lab result, vital sign | `Observation` | Categorize via `category` (laboratory, vital-signs, social-history, survey) |
| Allergy / intolerance | `AllergyIntolerance` | Use RxNorm for drug; SNOMED CT for substance |
| Procedure performed | `Procedure` | Use SNOMED CT or CPT (billing) |
| Immunization given | `Immunization` | Use CVX |
| Family history | `FamilyMemberHistory` | Often captured as Observations in older EHRs |
| Social history | `Observation` with category `social-history` | Smoking status, alcohol, housing, food insecurity |
| Free-text note | `DocumentReference` | The actual narrative, with the structured concepts above linked |

Capturing structure at authoring time means downstream systems (quality measures, CDS, registries, billing) do not need fragile NLP to reconstruct what the clinician already knew.

---

## Templates, SmartForms, SmartPhrases

| Tool | What | Examples |
|------|------|----------|
| **Template** | A pre-built note skeleton | "Diabetes annual visit template" |
| **SmartPhrase / dot phrase** | A short expansion (e.g., `.normalexam` → full normal physical exam paragraph) | Epic SmartPhrase, Cerner AutoText, Athenahealth macros |
| **SmartForm / SmartList / SmartLink** | A structured form embedded in a note that writes back to discrete fields | Epic SmartForm, Cerner Documentation Templates |
| **Macro / quick text** | User-defined snippet | Personal `.cp` for chest pain workup |

**Pitfall**: copy-forward of normal exams and unchanged HPIs is the most-cited documentation integrity problem. Track and audit copy-forward.

---

## Clinical NLP

For ingesting unstructured notes, choose tools by use case:

| Tool | Type | Best for |
|------|------|----------|
| **cTAKES** (Apache) | Open source UIMA pipeline | Concept extraction (UMLS), sentence detection, dependency parsing |
| **MedSpaCy** | Open source spaCy extension | Clinical NER, negation detection (NegEx, ConText) |
| **MetaMap / MetaMapLite** (NLM) | Open source | Mapping text to UMLS concepts |
| **Amazon Comprehend Medical** | Managed service | PHI detection, entity + relationship extraction, ICD-10 / RxNorm linking |
| **AWS HealthLake** | Managed service | FHIR data store + NLP enrichment; HIPAA-eligible |
| **Google Healthcare NLP API** | Managed service | UMLS-linked entities, FHIR enrichment |
| **Azure AI Language for Health** | Managed service | Entity extraction, UMLS linking |

### Common pipeline pattern

```
Raw note text
  → De-identification (Safe Harbor or Expert Determination)
  → Sentence / section detection
  → Named entity recognition (problems, meds, anatomy, tests, procedures)
  → Assertion classification (present, negated, hypothetical, family member, historical)
  → Concept normalization (UMLS CUI → SNOMED CT / RxNorm / LOINC)
  → Relationship extraction (med → indication, lab → reference range)
  → Output (FHIR resources, registry payload, abstraction worklist)
```

**Negation is non-negotiable.** "No chest pain" is the opposite of "chest pain." A model that ignores negation will create dangerous data.

### Evaluation

- F1 against a held-out, clinician-adjudicated reference set.
- Stratify by section (HPI vs. ROS vs. exam vs. A&P), by note type, and by site.
- Track concept-level performance, not just span-level.
- Re-evaluate after any pipeline or model update — clinical NLP regressions are sneaky.

---

## Ambient AI Scribes

Ambient scribes record the patient-clinician encounter (typically audio), transcribe, and generate a structured note draft. The clinician edits and signs.

### Vendor landscape (representative)

| Vendor | Notes |
|--------|-------|
| Nuance DAX (Microsoft) | Deep Epic and Cerner integration; enterprise contracts |
| Suki | Voice-first assistant; integrates with major EHRs |
| Abridge | Real-time generation; large health-system customers |
| Augmedix | Combines AI with human scribe review |
| DeepScribe | Ambient ambulatory focus |

Vendor offerings change quickly. Confirm current capabilities and EHR integrations directly with the vendor.

### Workflow integration

The path from microphone to signed note in the EHR typically looks like:

1. Clinician opens patient chart in EHR; ambient scribe app launches (often via SMART or vendor-specific embed).
2. Consent is captured from the patient (see below).
3. Audio is streamed to the scribe service (within a BAA-covered cloud).
4. The service returns a structured draft note.
5. Clinician reviews/edits in the scribe UI or in the EHR.
6. Final note is written back to the EHR as a `DocumentReference` (FHIR) or via vendor-specific note API.
7. Audio is retained per the BAA / customer policy (often 7-30 days for QA, then deleted).

### Accuracy expectations

- Word error rate (WER) on clinical speech: lower than general ASR for well-modeled vendors, but **higher than the marketing suggests** for accented speech, code-mixing, noisy clinics, and rare specialties.
- Section-level note accuracy: the assessment-and-plan is the most error-prone section. Treat it as the riskiest part of the draft.
- Clinician review burden: a scribe that is 80% accurate but requires 100% read-through provides limited time savings. Measure clinician-edit rate, not just WER.

### Consent

Ambient capture changes the consent calculus.

- **HIPAA**: under HIPAA alone, recording the encounter for treatment / operations purposes does not generally require separate written authorization beyond the Notice of Privacy Practices. Confirm with privacy counsel.
- **State two-party consent laws**: many US states (e.g., California, Florida, Illinois, Massachusetts, Pennsylvania, Washington — verify the current list) require all parties to consent to audio recording. Always default to explicit patient consent regardless of state.
- **Recording vs. transcription-only**: some vendors retain audio; others delete after transcription. The retention model changes the consent and breach-risk profile.
- Document consent for each encounter. Allow the patient to decline without affecting care.

> Confirm state recording laws against current statutes; they change.

---

## Documentation Integrity

- **Copy-forward / cloning**: pull-forward of prior notes is a top source of payer disputes and patient-safety incidents.
- **Note bloat**: notes that are 95% boilerplate and 5% substance defeat the purpose of charting.
- **Attribution**: every clinician's contribution should be attributable. Co-signature workflows for residents and students.
- **Late additions / addenda**: must be timestamped and clearly marked. EHRs handle this differently — confirm at each site.
- **Amendment vs. addendum**: amending changes content; an addendum adds new content. Confirm the legal record structure with the customer's HIM department.

---

## Writing Back to the EHR

The standard pattern is FHIR `DocumentReference`:

```http
POST /FHIR/R4/DocumentReference
Authorization: Bearer <token>
Content-Type: application/fhir+json
{
  "resourceType": "DocumentReference",
  "status": "current",
  "docStatus": "preliminary",
  "type": {
    "coding": [{ "system": "http://loinc.org", "code": "<verify current LOINC>", "display": "Progress note" }]
  },
  "subject": { "reference": "Patient/example-1" },
  "author": [{ "reference": "Practitioner/example-prov-1" }],
  "context": {
    "encounter": [{ "reference": "Encounter/example-enc-1" }],
    "period": { "start": "2026-01-15T09:00:00Z", "end": "2026-01-15T09:20:00Z" }
  },
  "content": [{
    "attachment": {
      "contentType": "text/plain",
      "data": "<base64-encoded-note>",
      "title": "Progress note"
    }
  }]
}
```

Use `docStatus = preliminary` for drafts; the clinician's sign-off promotes it to `final`. Some EHRs use the HL7 v2 **MDM** message family (T02 / T08 / T11) instead — confirm per vendor.

---

## Common Pitfalls

- Building a scribe with no plan for the EHR write-back path. Generating a great note that no one can save into the chart is a non-product.
- Ignoring negation, family-history attribution, or temporality in NLP. Creates active diagnoses out of "denies."
- Treating one site's SmartPhrases as portable. They are not — each site curates its own library.
- Letting copy-forward inflate notes without tracking the percentage that is actually new.
- Recording audio without explicit patient consent in a two-party-consent state.
- Retaining ambient audio longer than the BAA / policy allows.

---

## Task-Specific Questions

1. What documentation problem are you solving — authoring, ingestion, or both?
2. Who is the clinician user, in what setting?
3. Which EHR(s) and what is the write-back contract you have?
4. If ambient audio is involved, what is the consent + retention design?
5. What structured data must come out of the workflow (problem list, meds, vitals, social determinants)?
6. How will you measure documentation quality and clinician burden?

---

## Related Skills

- **healthcare-context**: Setting + EHR + role drive note-type and write-back patterns.
- **fhir-integration**: `DocumentReference`, `Condition`, `Observation`, `Procedure`, `AllergyIntolerance` resources.
- **ehr-integration**: Vendor-specific note write-back paths (Epic, Cerner, Athena, Meditech, etc.).
- **terminology-services**: SNOMED CT, ICD-10-CM, RxNorm, LOINC mapping used in structured capture.
- **hipaa-compliance**: BAA, consent, audit, and retention for documentation and ambient audio.
- **phi-handling**: De-identification before NLP training or analytics.
- **clinical-ai-ml**: When the scribe or NLP component is a regulated AI model.
