# Ambient AI Scribe Integration Checklist

A workflow integration checklist for ambient scribe tools that record the patient-clinician encounter, transcribe, generate a structured note draft, and write back into the EHR. Vendor offerings change quickly; confirm current capabilities and EHR integrations directly with each vendor.

## Representative Vendor Landscape

| Vendor | Notes |
|--------|-------|
| Nuance DAX (Microsoft) | Deep Epic and Cerner integration; enterprise contracts |
| Suki | Voice-first assistant; integrates with major EHRs |
| Abridge | Real-time generation; large health-system customers |
| Augmedix | Combines AI with human scribe review |
| DeepScribe | Ambient ambulatory focus |

> Verify each vendor's current capabilities, EHR integrations, retention policies, and contracting model directly with the vendor and your customer's privacy office. This list is not exhaustive and is for orientation only.

## Pre-Deployment Checklist

### Consent

- [ ] Written or recorded patient consent obtained for ambient audio capture.
- [ ] Consent flow accommodates patient declination without delaying care.
- [ ] State two-party consent law check completed. Many US states require all parties to consent to audio recording. Default to explicit consent in every state regardless of law.
- [ ] Consent language reviewed by privacy counsel.
- [ ] Consent is captured and persisted per encounter (FHIR `Consent` resource or vendor-equivalent record).
- [ ] Patient-facing description in plain language: what is being recorded, who hears the audio, how long it is retained, how to revoke.
- [ ] Process for patients who decline: scribe is disabled for that encounter and clinician documents normally.

> Verify current state recording laws against the actual statutes; the two-party-consent list and related rules change.

### Privacy and Security

- [ ] Business Associate Agreement (BAA) with the scribe vendor signed before any PHI is processed.
- [ ] PHI flow mapped end-to-end: microphone -> vendor cloud -> draft -> EHR -> retention.
- [ ] Vendor's cloud infrastructure verified HIPAA-eligible.
- [ ] Audio and transcript at-rest encryption confirmed.
- [ ] In-transit TLS 1.2+ confirmed at every hop.
- [ ] Vendor's SOC 2 Type 2 or equivalent report reviewed.
- [ ] Vendor's incident-response and breach-notification commitments documented.
- [ ] Audit logging captures every access to audio and transcript.
- [ ] Access controls limit who at the vendor can review audio (typically only authorized QA personnel, with logging).

### Retention

- [ ] Audio retention duration explicit (often 7-30 days for QA, then deleted).
- [ ] Transcript retention duration explicit; typically tied to medical-record retention.
- [ ] Patient-facing description includes retention period.
- [ ] Deletion mechanism verified end-to-end (audio actually leaves all storage and backups).
- [ ] Process for legal-hold scenarios documented.

### EHR Integration

- [ ] Launch model confirmed: SMART on FHIR EHR launch, vendor-specific embed, or hybrid.
- [ ] Note write-back path confirmed: FHIR `DocumentReference` or vendor-specific note API.
- [ ] LOINC document type code mapped to the customer's site (verified against current LOINC release).
- [ ] `docStatus` flow defined: `preliminary` while clinician edits, `final` on sign-off.
- [ ] Structured data extraction defined where applicable: Conditions, Observations, MedicationRequests created or proposed from the draft.
- [ ] Co-signature workflow handled for residents and APPs.
- [ ] Error handling for failed write-back: how is the draft preserved, who is notified.

### Clinical Workflow

- [ ] Clinician training on starting / pausing / stopping the scribe.
- [ ] Clinician training on what to review and how to edit the draft.
- [ ] Time expectation set: 80% accurate but 100% review required is still a win only if review is faster than writing from scratch. Measure.
- [ ] Default behavior when clinician forgets to stop the scribe (e.g., after the encounter ends, mic-up rules).
- [ ] Behavior in multi-clinician encounters (e.g., resident + attending).
- [ ] Behavior when family members or interpreters are in the room.

### Accuracy and Safety

- [ ] Word-error-rate (WER) baseline measured on the customer's actual speech distribution (accents, code-mixing, specialty terminology).
- [ ] Clinician-edit rate measured per section (HPI, ROS, exam, A&P).
- [ ] A&P treated as the highest-risk section — extra clinician scrutiny expected.
- [ ] Sample audits scheduled (e.g., monthly) to compare audio against final note.
- [ ] Process for clinician-reported errors: how feedback flows back to the vendor for model improvement.
- [ ] Hallucination monitoring: scribe must not introduce content not present in the audio.
- [ ] Negation and assertion handling tested for the customer's specialty.

### Regulatory Posture

- [ ] If the scribe drafts an A&P or recommends diagnoses, evaluate whether it crosses into SaMD territory. A pure transcription / scribe with clinician sign-off as the legal note generally does not, but verify against current FDA CDS guidance and ONC HTI-1 Predictive DSI transparency requirements where applicable.
- [ ] If used in ambulatory practice receiving Medicare payment, confirm compliance with current CMS documentation rules (signature, timing, attribution).
- [ ] Coding implications reviewed if scribe output feeds billing — copy-forward / over-documentation risk.

## Daily Operations Checklist

- [ ] Clinician sees the same scribe UI every encounter and knows how to invoke it.
- [ ] Consent reminder visible to the clinician before recording starts.
- [ ] Patient declination path is one click / one phrase.
- [ ] Draft available within the clinician's expected window (real-time or end-of-encounter, depending on vendor).
- [ ] Clinician edits, signs, and final note appears in EHR.
- [ ] Audio retention timer started; deletion happens on schedule.
- [ ] Daily exception report: failed write-backs, declined consents, system errors.

## Incident Triggers

Stop using the scribe and review with privacy + clinical leadership if any of the following occur:

- Audio retained beyond the documented window.
- A note is signed with content the audio does not support.
- A patient who declined consent is recorded.
- Audio or transcript appears in a system outside the BAA boundary.
- Vendor breach notification.
- Recurring hallucinated content (med doses, diagnoses, recommendations not in the encounter).

## Measurement

A scribe deployment is healthy when the following are tracked and trending:

| Metric | Direction |
|--------|-----------|
| Clinician-reported time savings per encounter | Up |
| Edit rate per section | Stable or down |
| Sign-off latency (encounter end -> note final) | Stable or down |
| Patient consent rate | Stable; very low decline if patient-facing experience is good |
| Hallucination incidents | Zero / near-zero |
| Audit-found discrepancies between audio and note | Low and falling |

If clinicians are spending more time reviewing the draft than they would have spent writing from scratch, the scribe is not yet a win for that user / specialty. Re-tune or re-train.
