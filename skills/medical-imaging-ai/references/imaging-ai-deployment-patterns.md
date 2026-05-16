# Imaging AI Deployment Patterns

Reference for choosing how AI results enter the radiologist's reading workflow. The wrong pattern is the most common reason a technically good model never delivers clinical value.

## Contents

- The Five Common Patterns (decision matrix)
- Inline CAD (Computer-Aided Detection / Diagnosis)
- Worklist Re-Prioritization (Triage)
- Asynchronous Secondary Capture
- DICOM Structured Report (SR) Return
- FHIR DiagnosticReport Return
- Hanging Protocols and Series Visibility
- Failure Modes by Pattern

---

## The Five Common Patterns

| Pattern | Latency budget | Radiologist effort | Best for |
|---------|----------------|--------------------|-----------|
| Inline CAD | Seconds at read time | Reviews marks on every case | Detection tasks (lung nodules, MG calcifications) where missed findings drive the use case |
| Worklist re-prioritization (triage) | Seconds-to-minutes after acquisition | Reads high-priority studies first | Time-critical conditions: stroke (LVO/ICH), PE, tension pneumothorax, aortic dissection |
| Asynchronous secondary capture | Minutes (well before read) | Reviews an image or PDF in the viewer | Quantitative measurements (cardiac function, BMD, FFR-CT) and image-based summaries |
| DICOM SR | Minutes | Reviews / corrects structured findings | Tasks where downstream systems must consume the result discretely |
| FHIR DiagnosticReport return | Asynchronous to EHR | EHR-side surfacing to ordering clinician | Image-derived risk scores, opportunistic findings (CAC, vertebral fractures) consumed by primary care |

The patterns compose. A stroke triage model may simultaneously re-prioritize the worklist, emit a DICOM SR for the radiologist, and post a FHIR DiagnosticReport for downstream care management.

### Choosing

Ask:

1. **Who acts on the result?** Radiologist at read time → inline or triage. Specialist later → SR or FHIR. Non-radiologist clinician → FHIR.
2. **What is the cost of waiting?** Time-critical clinical condition → triage. Non-urgent → async patterns.
3. **Is the model adding marks or numbers?** Marks → inline / SEG. Numbers / structured findings → SR. Probability score → SR or FHIR Observation.
4. **Does the model need to fit existing report flow?** Yes → SR with measurements that pre-populate the report. No → secondary capture is enough.
5. **What does the orchestrator support?** Many orchestrators (Nuance PIN, Blackford, Calantic, Sirona, GE Edison, Philips ISP — verify current capabilities) support a subset of these patterns and dictate what is practical to deploy.

---

## Inline CAD

### What

Annotations or markers appear in the viewer while the radiologist is reading. The model has already run by the time the study is opened.

### When It Fits

- High-prevalence detection tasks where missed findings drive medico-legal and outcome risk: mammography CAD, lung-nodule detection on chest CT, intracranial aneurysm on CTA.
- Tasks where the radiologist's eye is the gold standard and the model is an assistant, not a replacement.

### Technical Requirements

- **Latency at read open**: marks must appear within seconds of the user opening the study. Pre-compute results during the acquisition-to-PACS window.
- **Viewer rendering**: marks must render in the radiologist's actual viewer (Visage, IntelliSpace, EI, vendor-native PACS viewer, Carestream, etc.). Two main paths:
  - **DICOM Segmentation (SEG)** for region overlays.
  - **DICOM Presentation State (GSPS) or Structured Display** for boxes / arrows.
  - **DICOM SR with TID 1500 Measurement Report** linking observations to image coordinates.
- **Hanging protocol**: the new series must be visible in the viewer's default layout for the protocol. Otherwise the radiologist never sees the marks.

### Watch For

- Over-marking. CAD that puts a mark on every nodule-shaped shadow trains the radiologist to ignore the marks. Tune sensitivity / specificity for the deployment context, not for the leaderboard.
- Bias. CAD lowering radiologist sensitivity on the unmarked findings has been observed in screening contexts. Subgroup-level monitoring is essential.

---

## Worklist Re-Prioritization (Triage)

### What

The model emits a binary or score-based flag immediately after acquisition. The flag is surfaced to RIS / PACS, which promotes the study to the top of the radiologist's worklist or routes it to an "urgent" pool.

### When It Fits

- Stroke imaging (LVO detection on CTA, ICH on non-contrast head CT).
- Pulmonary embolism on CT pulmonary angiography.
- Tension pneumothorax on portable CXR.
- Aortic dissection on chest CTA.
- C-spine fracture on cervical CT in trauma.

Any condition where minutes of read delay change patient outcomes.

### Technical Requirements

- **Latency from acquisition complete to flag**: seconds to a few minutes. Cloud inference is usually too slow unless co-located.
- **Integration channel**: the orchestrator pushes a flag to RIS (HL7 v2 SIU/ORM/ORU update or vendor-specific API), which displays a banner or moves the row in the worklist. Some PACS support direct DICOM tag-based flagging.
- **Result return alongside the flag**: a DICOM SR or a DICOM SEG so the radiologist has more than a binary at read time.
- **False-flag handling**: every triage flag has a false-positive rate. Document the expected rate at the chosen threshold so the radiology team understands what they are signing up for.

### Watch For

- Alert fatigue. A flag on every other chest CT is no flag at all.
- Compound triage. Three independent models flagging the same study with different priorities is a UI problem the radiology lead must own.
- Equity. Triage that systematically routes one subgroup's studies later in the read order is a regulatory and ethical problem.

---

## Asynchronous Secondary Capture

### What

The model writes one or more derived images (or a PDF) into the study as an additional series. The radiologist sees them when opening the study.

### When It Fits

- Quantitative results that benefit from a graphical summary: cardiac function (LVEF, GLS), bone mineral density on opportunistic CT, vertebral fracture detection, FFR-CT colormaps, lung-nodule detection summary panels.
- Pre-populated structured findings that the radiologist reviews and signs.

### Technical Requirements

- **DICOM Secondary Capture (SC)** SOP class for image-only output (modality-agnostic).
- **DICOM Encapsulated PDF** when a human-readable summary is the primary output.
- **DICOM SR with TID 1500** if the result is also structured.
- Hanging-protocol update so the new series appears in the read layout.

### Watch For

- Secondary capture alone is weakly machine-readable; downstream extraction is hard. Pair with a DICOM SR or FHIR Observation when downstream systems care about the value.
- PDF results do not survive into structured analytics. If you ever want to chart the metric across patients, do not stop at PDF.

---

## DICOM Structured Report (SR) Return

### What

The AI emits a DICOM SR object (SOP class such as Enhanced SR, Comprehensive SR, or TID 1500 Measurement Report) containing coded concepts and measurements tied to image coordinates.

### When It Fits

- Any output the radiology report or downstream system will consume discretely: measurements, scores, classifications, findings with image references.
- Reporting workflows where the SR pre-populates structured fields in the dictation system (PowerScribe, Fluency).
- Output that must survive PACS-to-VNA migration with semantics intact.

### Technical Requirements

- **Concept coding**: use a published terminology — DICOM, SNOMED CT, LOINC, RadLex — for both the question and the answer. Avoid free-text where a code exists.
- **Image coordinate linkage**: SCOORD / SCOORD3D content items so the receiving viewer can render the measurement on the original image.
- **Manufacturer / SoftwareVersions tags on the SR** so post-deployment monitoring can attribute results to a specific model version.
- **AlgorithmIdentification / AlgorithmVersion**: include the model identifier and version explicitly in the SR. This is the audit trail.

### Watch For

- Viewer support varies. Verify that the radiologist's actual viewer renders the SR correctly before deploying.
- Multiple SRs from multiple models in the same study should not stomp each other. Use SeriesDescription and content item structure that disambiguates.

---

## FHIR DiagnosticReport Return

### What

The model posts a FHIR `DiagnosticReport` (with `Observation`, `ImagingStudy` reference, and optional `RiskAssessment`) to the EHR. The result is visible in the patient chart outside the radiology viewer.

### When It Fits

- Opportunistic findings consumed by non-radiologists: opportunistic CAC scoring, vertebral fracture detection, sarcopenia metrics, hepatic steatosis scoring, opportunistic AAA screening.
- Image-derived risk scores that feed care pathways (CKD progression, cardiovascular risk).
- Population-health and analytics ingestion where the image-derived metric must reach the data lake.

### Technical Requirements

- **Profile alignment**: align with US Core, mCODE for oncology, or institution-specific FHIR profiles. Pick discrete `Observation` codes from LOINC where they exist.
- **References**: `DiagnosticReport.imagingStudy` → the FHIR `ImagingStudy`; `Observation.derivedFrom` → other observations; provenance via `Provenance` resource if your EHR supports it.
- **Performer identity**: the model itself (`Device` resource) is the performer. Include manufacturer, model name, version.
- **Update semantics**: if the model re-runs and the result changes, the new `DiagnosticReport` should reference and supersede the prior one.

### Watch For

- EHR-side rendering varies. A FHIR-posted result that is technically present but invisible to the clinician is no result.
- Patient-portal visibility. If the result is in `DiagnosticReport` and the patient portal pulls all such reports, the patient may see the result before the ordering clinician has reviewed it. Coordinate with the EHR governance team.

---

## Hanging Protocols and Series Visibility

A frequent silent failure: the model produces a new series, the orchestrator pushes it to PACS, and the radiologist never sees it because the hanging protocol does not include it.

Coordinate with the PACS administrator on:

- **Series matching rules**: SeriesDescription pattern that the new series will carry.
- **Default layout slot**: where the AI series appears in the on-screen layout for that protocol.
- **Visibility tier**: always visible vs. on-demand vs. minimized.
- **Protocol versioning**: hanging protocols are per-vendor and per-site; a change at one site does not propagate.

Test in the actual reading environment with the actual radiologists before declaring deployment complete.

---

## Failure Modes by Pattern

### Inline CAD

- Marks not rendered (hanging-protocol miss).
- Over-marking driving radiologist alert fatigue.
- CAD-induced bias on unmarked findings.

### Worklist Re-Prioritization

- Latency too high; flag arrives after the study has been read.
- Flag stomps a higher-priority flag from another model.
- Subgroup-equity drift in triage rates over time.

### Secondary Capture

- Result is visible but not machine-readable.
- New series invisible due to hanging-protocol mismatch.
- PDF results lost to analytics.

### DICOM SR

- Viewer does not render the SR.
- Concept codes that the institution does not use elsewhere; result is technically structured but practically isolated.
- Missing AlgorithmVersion → cannot reconstruct which model produced a given output six months later.

### FHIR DiagnosticReport

- EHR profile drift; the post succeeds but the result is not displayed.
- Patient-portal premature disclosure.
- No `Provenance`, no audit trail of which model version produced the result.
