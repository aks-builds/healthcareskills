# Clinical Note Templates

Reusable structural templates for the most common clinical note types. These are skeletons — not clinical content. Always tailor to the patient, the encounter, and the institution's documentation policy. LOINC document type codes must be verified against the current LOINC release and the customer's site mapping before use.

## Contents

- SOAP Progress Note
- APSO Progress Note
- History and Physical (H&P)
- Discharge Summary
- Operative Note
- Consult Note
- Procedure Note
- Telephone / Patient Message

---

## SOAP Progress Note

Subjective -> Objective -> Assessment -> Plan. Classic Weed problem-oriented structure.

```
# Progress Note

Patient: [Name / MRN]
Date / Time: [ISO 8601]
Author: [Clinician name, role]
Encounter: [Encounter ID]

## Subjective
- Chief complaint / interval history
- Symptoms (onset, duration, severity, associated)
- Patient-reported response to current therapy
- Pertinent positives and negatives

## Objective
- Vitals (most recent + trend if relevant)
- Pertinent exam findings (focused, not exhaustive)
- New labs / imaging / studies since last note
- Current medications (active)

## Assessment
- Problem #1: [name] — interpretation and current status
- Problem #2: [name] — interpretation and current status
- ...

## Plan
- Problem #1:
  - Diagnostics: [...]
  - Therapeutics: [...]
  - Patient education: [...]
  - Follow-up: [...]
- Problem #2:
  - ...

## Disposition / Follow-up
- [Next step in care]

[Electronic signature]
```

---

## APSO Progress Note

Assessment and Plan first, so the next reader sees the conclusion before the supporting data. Increasingly preferred over SOAP in many hospitals.

```
# Progress Note (APSO)

Patient: [Name / MRN]
Date / Time: [ISO 8601]
Author: [Clinician name, role]
Encounter: [Encounter ID]

## Assessment
- Problem #1: [name] — interpretation and current status
- Problem #2: [name] — interpretation and current status
- ...

## Plan
- Problem #1:
  - Diagnostics, therapeutics, education, follow-up
- Problem #2:
  - ...

## Subjective
- Interval history, symptoms, patient-reported response

## Objective
- Vitals, exam, new studies, current meds

[Electronic signature]
```

---

## History and Physical (H&P)

Used at admission, transfer to a new service, or new ambulatory patient. The most comprehensive note type.

```
# History and Physical

Patient: [Name / MRN]
Date / Time: [ISO 8601]
Author: [Clinician name, role]
Encounter: [Encounter ID]

## Chief Complaint (CC)
[One sentence in the patient's own words where possible]

## History of Present Illness (HPI)
- Onset, location, duration, character, aggravating / alleviating factors, radiation, timing, severity (OLD CARTS or PQRST)
- Pertinent positives and negatives
- Relevant prior workup

## Past Medical History (PMH)
- Active conditions (Problem List)
- Resolved conditions of relevance

## Past Surgical History (PSH)
- Procedures with dates

## Medications
- Current medications (name, dose, frequency, route, indication)
- Recently stopped / changed and why

## Allergies
- Allergen, reaction, severity
- "NKDA" if applicable, documented

## Family History (FH)
- First-degree relatives, conditions, ages at diagnosis where known

## Social History (SH)
- Living situation, occupation
- Tobacco, alcohol, recreational drugs (with quantitation)
- Diet, exercise, sleep
- Sexual history when relevant
- Travel, exposures, occupational hazards when relevant
- Social determinants: housing, food, transportation when relevant

## Review of Systems (ROS)
- Constitutional, HEENT, cardiac, respiratory, GI, GU, MSK, neuro, psych, skin, heme/lymph, endocrine, allergy/immuno
- Pertinent positives in detail; "all other systems reviewed and negative" only if true

## Physical Exam
- Vitals
- General appearance
- HEENT, neck
- Cardiovascular
- Respiratory
- Abdomen
- Extremities
- Skin
- Neurological (focused or comprehensive)
- Psychiatric (mental status as appropriate)

## Labs and Studies
- Pertinent prior and current results

## Assessment
- Differential diagnosis with reasoning
- Working diagnosis

## Plan
- Workup
- Treatment
- Disposition
- Follow-up

[Electronic signature]
```

---

## Discharge Summary

Required for inpatient stays; closes out the encounter and communicates with the receiving provider.

```
# Discharge Summary

Patient: [Name / MRN]
Admission Date: [ISO 8601]
Discharge Date: [ISO 8601]
Attending: [Name]
Discharging Service: [Service]

## Admission Diagnosis
[Primary]

## Discharge Diagnoses
- Primary: [...]
- Secondary: [...]

## Hospital Course
[Narrative summary of the stay, organized by problem or by chronology, including significant events, consultations, procedures, complications, and resolution]

## Procedures Performed
- [Procedure, date, performing provider]

## Discharge Medications
- [Medication, dose, frequency, route, duration, indication]
- Mark each as: continued, modified, new, or held
- Stopped during admission: [list with reason]

## Allergies
- [Allergen, reaction]

## Discharge Condition
[Stable / improving / fair / etc.]

## Discharge Disposition
[Home / SNF / rehab / hospice / etc.]

## Follow-up
- [Provider, specialty, recommended timing, contact info]
- Pending tests at discharge: [results, who will follow up]

## Patient Instructions
[Plain-language summary of what the patient should do at home: when to take which meds, activity restrictions, when to seek care, follow-up appointments]

[Electronic signature]
```

---

## Operative Note

Post-procedure documentation. Many institutions use a structured template that maps each field to discrete EHR data.

```
# Operative Note

Patient: [Name / MRN]
Date of Operation: [ISO 8601]
Surgeon: [Name]
Assistants: [Names]
Anesthesia: [Type and provider]

## Preoperative Diagnosis
[...]

## Postoperative Diagnosis
[...]

## Procedure Performed
[Specific procedure name(s)]

## Indications
[Why this procedure was performed]

## Findings
[Pertinent intraoperative findings]

## Description of Procedure
[Stepwise narrative of the procedure including positioning, prep, incision, key technical steps, hemostasis, closure]

## Estimated Blood Loss (EBL)
[Volume]

## Specimens
[List and disposition]

## Complications
[None or specific complications]

## Disposition
[Recovery, ICU, floor, etc.]

[Electronic signature]
```

---

## Consult Note

Used when a specialist sees a patient in consultation.

```
# Consult Note

Patient: [Name / MRN]
Date / Time: [ISO 8601]
Consulting Service: [Specialty]
Consulting Provider: [Name]
Requesting Provider: [Name]

## Reason for Consult
[Specific question the requesting provider wants answered]

## History
[Pertinent history from the patient and the chart]

## Examination
[Focused exam relevant to the consult question]

## Pertinent Data
[Labs, imaging, prior studies]

## Impression
[Specialist's interpretation of the clinical situation]

## Recommendations
- [Specific, actionable recommendations]
- [Diagnostic next steps]
- [Therapeutic recommendations]
- [Follow-up plan]

## Plan to follow / sign off
[Will continue to follow / signed off after this encounter]

[Electronic signature]
```

---

## Procedure Note

Bedside or office procedure (e.g., paracentesis, lumbar puncture, joint injection, biopsy).

```
# Procedure Note

Patient: [Name / MRN]
Date / Time: [ISO 8601]
Performing Provider: [Name]
Supervising Provider: [Name, if trainee]
Procedure: [Specific procedure]

## Indication
[Clinical justification]

## Consent
[Informed consent obtained, risks/benefits/alternatives discussed]

## Time Out
[Confirmed: correct patient, correct site, correct procedure]

## Technique
[Stepwise description]

## Findings
[Pertinent findings]

## Specimens
[Sent for what testing]

## Complications
[None or specific]

## Post-Procedure
[Status, disposition, monitoring plan]

[Electronic signature]
```

---

## Telephone / Patient Message Note

For asynchronous patient communication captured in the chart.

```
# Telephone Encounter / Patient Message

Patient: [Name / MRN]
Date / Time: [ISO 8601]
Author: [Clinician name, role]
Initiated by: [Patient / Provider / Other]

## Message Summary
[What the patient communicated; what the question is]

## Action Taken
[Advice given, prescription sent, referral made, escalation, etc.]

## Provider Sign-off
[If staff documented and provider co-signed]

[Electronic signature]
```

---

## Implementation Notes

### LOINC Document Type Codes

Each note type should be tagged with a LOINC document type code on the FHIR `DocumentReference.type`. Common categories:

- Progress note
- Discharge summary
- History and physical note
- Consultation note
- Operative note
- Procedure note
- Telephone encounter note

Specific codes must be verified against the current LOINC release and the customer's site mapping — sites localize.

### Structured Capture

Wherever possible, capture structured data at the moment of authoring rather than relying on downstream NLP:

| Concept | FHIR resource |
|---------|---------------|
| Diagnosis / problem | `Condition` |
| Lab / vital | `Observation` |
| Allergy | `AllergyIntolerance` |
| Procedure | `Procedure` |
| Immunization | `Immunization` |
| Family history | `FamilyMemberHistory` |
| Social history | `Observation` (category `social-history`) |
| Free-text note | `DocumentReference` |

### Copy-Forward Discipline

- Track the percentage of each note that is copy-forward vs. new content.
- Surface a warning if a daily progress note is >90% identical to the previous day's.
- Audit periodically for "notes" that have not changed in days for evolving conditions.

### Attribution

- Every note must have a clear author and timestamp.
- Co-signature workflow for residents, students, APPs.
- Amendments and addenda are timestamped and labelled distinctly from the original note.
