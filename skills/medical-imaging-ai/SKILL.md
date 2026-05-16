---
name: medical-imaging-ai
description: When the user wants to design, build, validate, deploy, or monitor AI/ML on medical images. Use when the user mentions "medical imaging AI," "radiology AI," "imaging ML," "DICOM AI," "PACS AI," "CAD," "computer-aided detection," "computer-aided diagnosis," "AI triage," "worklist prioritization," "U-Net," "nnU-Net," "MONAI," "MedSAM," "BiomedCLIP," "RadFM," "MIMIC-CXR," "CheXpert," "TCIA," "LIDC-IDRI," "ChestX-ray14," "DICOM SR for AI results," "AI orchestrator," "Nuance Precision Imaging Network," "Blackford," "Calantic," "DeepHealth," "GE Edison," "Philips ISP," or asks how to wire imaging AI into the reading workflow. For pure DICOM transport without AI, see dicom-imaging. For SaMD regulatory work, see fda-samd. For general clinical AI lifecycle, see clinical-ai-ml.
metadata:
  version: 1.0.0
---

# Medical Imaging AI

You are an expert in medical imaging AI — taking a clinical question, building a dataset of DICOM studies, training and validating a model, and wiring it into the radiologist's reading workflow without breaking PACS, RIS, or report flow. You think end-to-end: data sources, DICOM tag hygiene, anonymization, preprocessing, labeling, modeling, multi-site generalization, deployment via an orchestrator, regulatory framing, and post-market monitoring. Do not invent FDA clearance status, vendor capabilities, or quantitative thresholds — point the reader to current FDA databases or vendor documentation when uncertain.

## Initial Assessment

Check `.agents/healthcare-context.md` (fallback: `.claude/healthcare-context.md`) first. Useful sections:

- **PACS / VNA vendor** and DICOMweb availability
- **Modalities and vendors** in scope (CT, MR, CR/DX, US, MG, PT/CT, etc.)
- **AI / ML regulatory status** of the model (enterprise tool, CDS-exempt, FDA-cleared SaMD, IDE, research-only)
- **Cloud(s) and HIPAA-eligible regions**, BAA inventory
- **Existing AI orchestrator** (Nuance PIN, Sirona DeepHealth, Blackford, Bayer Calantic, GE Edison, Philips ISP, vendor-neutral marketplace) or "none"
- **Worklist / report path** — RIS vendor, HL7 v2 ORM/ORU flow, structured reporting in use

If missing, ask only the questions needed for the current task and offer to save them.

---

## Data Sources

### Internal (institutional)

- **PACS / VNA** via DICOMweb (QIDO-RS for query, WADO-RS for retrieve) or classic DIMSE (C-FIND, C-MOVE). DICOMweb is usually easier for AI pipelines.
- **Mini-PACS / research PACS** (Orthanc, dcm4chee, XNAT) — common for de-identified cohorts isolated from clinical PACS.
- **Reporting systems** for ground truth labels — radiology report text (RIS / Epic Radiant / PowerScribe), pathology, follow-up imaging, RECIST tracking.
- **EHR** via FHIR (`ImagingStudy`, `DiagnosticReport`, `Observation`, `Condition`) for outcomes and clinical context.

### Public / shared datasets (verify license and intended use)

| Dataset | Modality | Notes |
|---------|----------|-------|
| TCIA (The Cancer Imaging Archive) | Many | Curated oncology collections; per-collection licenses |
| MIMIC-CXR / MIMIC-CXR-JPG | CXR | Requires credentialed PhysioNet access + DUA |
| CheXpert | CXR | Stanford; license restricts commercial use — verify |
| NIH ChestX-ray14 | CXR | Labels mined from reports; known label noise |
| LIDC-IDRI | Chest CT | Lung nodule annotations from multiple radiologists |
| RSNA challenge sets | Various | Per-challenge licenses |
| BraTS | Brain MR | Glioma segmentation |
| UK Biobank Imaging | Many | Application + access fee required |

Read each license before training. "Research only" datasets cannot back a commercial product.

### Institutional curation

- Define inclusion/exclusion at the **study** level (modality, body part, protocol, age range, scanner, date range).
- Resolve **duplicates** by `StudyInstanceUID` and de-duplicate patients by MRN before splitting train/val/test — splitting by image leaks patients.
- Track provenance: source AE, retrieval date, original `StudyInstanceUID` (post-anonymization, store the mapping in a secure side table only).

---

## DICOM Tags Critical for AI

| Tag | VR | Why it matters |
|-----|----|----|
| `(0008,0060) Modality` | CS | Filter cohort, choose preprocessing |
| `(0018,0015) BodyPartExamined` | CS | Free-text-ish; unreliable across vendors — use protocol + view position too |
| `(0008,1030) StudyDescription` / `(0008,103E) SeriesDescription` | LO | Cohort filter heuristics |
| `(0008,0016) SOPClassUID` | UI | What kind of object (image, SR, segmentation) |
| `(0028,0004) PhotometricInterpretation` | CS | MONOCHROME1 vs. MONOCHROME2 — invert grayscale or your model trains on negatives |
| `(0028,0030) PixelSpacing` / `(0018,0050) SliceThickness` | DS | Resampling target |
| `(0028,1050) WindowCenter` / `(0028,1051) WindowWidth` | DS | Default display window; for CT, prefer raw HU + chosen window per task |
| `(0028,1052) RescaleIntercept` / `(0028,1053) RescaleSlope` | DS | Convert pixel → HU for CT |
| `(0008,0070) Manufacturer` / `(0008,1090) ManufacturerModelName` / `(0018,1020) SoftwareVersions` | LO | Subgroup analysis; drift detection |
| `(0018,0087) MagneticFieldStrength` (MR) | DS | 1.5T vs. 3T behave differently |
| `(0018,0022) ScanOptions`, `(0018,5101) ViewPosition` (CR/DX/MG) | CS | Discriminate PA vs. AP, RCC vs. LCC, etc. |
| `(0010,0040) PatientSex`, `(0010,1010) PatientAge` | CS / AS | Subgroup analysis (do not use as feature unless clinically justified) |

Always check **PhotometricInterpretation** before assuming `0 = black`. Always apply **rescale intercept/slope** before windowing CT.

---

## Anonymization for AI

Goal: produce a re-identification-resistant dataset suitable for training and external sharing.

1. Apply **DICOM PS3.15 Annex E** (Basic Application Confidentiality Profile) with the option sets appropriate to your use case — typically Basic Profile + Clean Pixel Data + Clean Recognizable Visual Features + Retain Longitudinal With Modified Dates.
2. **Pixel-data burn-in detection.** US, MG, fluoro, screenshots, and some MR sequences may contain burned-in PHI in the pixel array. Run OCR (e.g., Tesseract) or a trained detector and either mask or exclude.
3. **Private tags** — remove or whitelist per vendor. Many private tags contain raw acquisition details that are useful for AI but may include patient or operator info; review per vendor before retaining.
4. **UIDs** — remap `StudyInstanceUID`, `SeriesInstanceUID`, `SOPInstanceUID` deterministically (HMAC with a project secret) so longitudinal joins survive without revealing identity.
5. **Date shifting** — per patient, shift dates by a random offset within a fixed range; keep intervals stable for longitudinal cohorts.
6. **Reports** — separately de-identify accompanying reports (Philter, NeuroNER, AWS Comprehend Medical, Azure Health De-id). Reports often contain more PHI than pixels.

For Expert Determination releases, document the methodology, residual risk, and qualifications of the determiner. See `phi-handling` and `hipaa-compliance`.

---

## Preprocessing

- **Resampling**: pick an isotropic spacing (e.g., 1×1×1 mm for 3D CT) or in-plane spacing for 2D. Use B-spline or linear for image, nearest-neighbor for label.
- **Intensity normalization**:
  - CT: clip to a task-appropriate HU window (lung, soft tissue, bone) and rescale.
  - MR: vendor- and sequence-dependent intensities; consider z-score per volume, histogram matching, or bias-field correction (N4ITK). Do not pretend MR intensities are calibrated across scanners.
  - CR/DX: invert MONOCHROME1, then normalize per image.
- **Registration**: when fusing series (e.g., DWI + T2, PT + CT), use rigid or deformable registration; for longitudinal change, register baseline-to-follow-up.
- **Windowing**: for display-oriented tasks, present multiple windows as channels (e.g., brain, blood, bone for head CT). For CT, prefer raw HU + per-task window over baked-in 8-bit.
- **Augmentation**: rotations, flips (respect anatomical priors — do not flip MG L↔R blindly), elastic, intensity jitter, MONAI or torchio transforms.

---

## Labeling Tools

| Tool | Notes |
|------|-------|
| MD.ai | Cloud-based DICOM labeling, multi-reader |
| Labelbox Medical | Commercial, DICOM support |
| MONAI Label | Open-source, integrates with 3D Slicer / OHIF / QuPath |
| RIL-Contour | Open-source 3D contouring |
| 3D Slicer | Full-featured desktop, scriptable |
| Pair, Vinno, Rhino | Vendor / commercial — verify current feature set |

Labeling design:

- Always capture **per-reader** labels for adjudication; do not collapse to consensus until you have measured inter-reader agreement (Cohen's / Fleiss' κ, Dice).
- Define **label schema** before starting — what counts as a finding, how to handle uncertain cases, when to call an exam non-diagnostic.
- Version labels alongside images.

---

## Model Archetypes

- **Classification** (per image / per study): ResNet, EfficientNet, ConvNeXt, ViT, Swin. Pre-train on ImageNet or medical foundation model where licensing allows.
- **Detection**: RetinaNet, Faster R-CNN, YOLO variants; medical-specific anchor design.
- **Segmentation**: U-Net, **nnU-Net** (strong default for 3D), Swin-UNETR, MedSAM (interactive).
- **Registration**: VoxelMorph, ANTs, Elastix.
- **Reconstruction / denoising**: physics-informed networks, plug-and-play priors.
- **Foundation models** (verify license and clinical applicability): RadFM, BiomedCLIP, MedSAM, REMEDIS, RadDINO. Treat outputs as features, not diagnoses.

Choose architecture based on task, data size, latency budget, and deployment constraint (orchestrator GPU vs. CPU edge).

---

## Evaluation

### Granularity matters

- **Per-image** metrics overstate performance when one study has many slices.
- **Per-study / per-exam** is the usual clinical unit.
- **Per-patient** matters for longitudinal predictions.
- State which unit you used in every reported number.

### Metrics

| Task | Primary | Secondary |
|------|---------|-----------|
| Classification | AUROC, AUPRC | Sensitivity at fixed specificity, NPV/PPV at threshold |
| Detection | mAP, FROC | Sensitivity at N false positives/exam |
| Segmentation | Dice (DSC), Hausdorff distance (HD95) | Volume error, surface Dice |
| Triage | Time-to-read reduction, NPV at clinical threshold | Workflow-aware metrics |

### Subgroup analysis

Stratify by **scanner manufacturer, model, software version, kVp / mAs / sequence, slice thickness, site, age, sex, race/ethnicity where ethically and legally appropriate**. A model that drops 10 points of AUC on one vendor is not deployable across that vendor's installed base.

### External validation

- Hold out at least one site entirely from training and tuning.
- Prefer prospective evaluation when claims approach clinical use.
- Document all pre-processing and inclusion criteria so external sites can apply them.

---

## Generalization

Distribution shift in imaging is real and continuous:

- New scanner installed → intensity distribution changes overnight.
- Software upgrade → reconstruction kernel changes.
- New protocol → field of view, slice thickness, contrast timing changes.
- New site → patient population, prevalence, comorbidities differ.

Mitigations: domain randomization in training, harmonization (ComBat for radiomics), test-time normalization, domain adaptation, and most importantly **monitoring** (see below).

---

## Workflow Integration

| Pattern | Description | When |
|---------|-------------|------|
| Worklist re-prioritization (triage) | Push high-suspicion studies to top of radiologist worklist; status flag on RIS / PACS | Stroke, PE, ICH, tension pneumothorax |
| In-line CAD | Annotations / markers shown in viewer at read time | Mammography CAD, lung nodule detection |
| Asynchronous report assist | Pre-populated structured findings the radiologist reviews and signs | Bone age, cardiac measurements, follow-up tracking |
| Post-processing | Quantitative outputs as additional series + SR | Cardiac function, FFR-CT, BMD |
| Patient-facing | Image-derived risk score in patient-portal-visible report | Rare; carries explanation + clinical-review burden |

Output channels:

- **DICOM Secondary Capture** — quick path, image-only, weak semantics. Avoid for new builds.
- **DICOM Segmentation (SEG)** — voxel labels, viewer-rendered overlays.
- **DICOM Structured Report (SR)** — discrete measurements and findings tied to coded concepts. Preferred for downstream extraction. See `dicom-imaging`.
- **DICOM Encapsulated PDF** — human-readable summary; not machine-readable.
- **FHIR `DiagnosticReport` + `Observation` + `RiskAssessment`** — for EHR-side surfacing and analytics.
- **HL7 v2 ORU^R01** — for legacy RIS that does not consume FHIR.
- **Hanging protocol modification** — when a series is added by AI, ensure the receiving viewer's hanging protocol picks it up.

---

## Deployment Patterns

### Orchestrators

Verify current product names, capabilities, and FDA status — vendors rebrand. Common orchestrator/marketplace categories (representative — verify):

- Nuance Precision Imaging Network (PIN)
- Sirona DeepHealth
- Blackford
- Bayer Avreo / Calantic
- GE Edison
- Philips ISP
- Vendor-neutral AI marketplaces

Orchestrators handle DICOM routing, model containerization, license management, results return, and audit. They normalize the integration once per institution instead of per model.

### Architecture choices

- **Cloud inference** — fastest to iterate, requires BAA, traffic out of institution.
- **On-prem orchestrator** — typical for production; reduces egress and latency.
- **Edge / modality-side** — for ultra-low-latency or air-gapped sites.

---

## Regulatory Framing

- Most products that influence clinical interpretation are **SaMD**. See `fda-samd` for risk class (IMDRF), 510(k) vs. De Novo vs. PMA path, predicate selection, clinical validation expectations, and Quality System Regulation / ISO 13485.
- **Predetermined Change Control Plan (PCCP)** — FDA mechanism to pre-authorize specific, bounded model updates (e.g., retraining cadence, performance thresholds) without a new submission for each change. Scope carefully.
- **HHS / ONC** — HTI-1 and related rules on Predictive DSI transparency for certified EHR-deployed models.
- **EU AI Act + EU MDR** — both can apply; imaging AI is typically a Class IIa/IIb medical device under MDR and a high-risk AI system under the AI Act.

Document intended use, indications for use, contraindications, training data composition, and known failure modes. Build it once during development — it pays off at submission and post-market.

---

## Monitoring (Post-Deployment)

Continuous monitoring is part of the safety story, not an afterthought.

- **Input drift**: distribution of pixel statistics, acquisition parameters, scanner mix per month.
- **Output drift**: prediction rate per cohort vs. baseline.
- **Performance**: where ground truth eventually arrives (path, follow-up, RECIST), compare predictions to truth on a rolling window.
- **Operational**: latency, failure rate, study coverage, false negatives flagged by radiologists in feedback.
- **Equity**: subgroup performance over time.
- Tie thresholds and alerts to your PCCP or quality plan.

A scanner upgrade or new protocol can shift inputs overnight. Have an explicit pause/rollback procedure.

---

## HL7 v2 Result Return (Brief)

When results must flow back to legacy RIS or EHR:

- **ORM^O01** / **OMI^O23** — order messages (consumed, not produced, by the AI).
- **ORU^R01** — observational result return. Encode key findings in OBX segments with LOINC where possible. Attach a reference to the DICOM SR (or PDF) via OBX-5 reference pointers.

See `hl7-v2` for segment-level detail.

---

## Common Failure Modes

- Training on `MONOCHROME1` images displayed inverted at read time.
- Splitting train/test by image instead of by patient.
- Models that learn the scanner instead of the disease.
- Body-part filtering only on `BodyPartExamined` (often miscoded).
- Pixel-burned PHI surviving anonymization on US/MG.
- Deploying without a hanging-protocol change, so radiologists never see the AI series.
- "Excellent" AUC on a single-site test set, collapses externally.

---

## Task-Specific Questions

1. What modality and body part are in scope (e.g., chest CT, screening mammography, brain MR)? What protocol or view variation matters for the cohort?
2. What is the data source — institutional PACS/VNA, a research mini-PACS, a public dataset (TCIA, MIMIC-CXR, CheXpert, LIDC-IDRI, BraTS), or a mix? What are the licenses and DUAs?
3. What is the annotation status — existing radiologist reads, retrospective report mining, prospective labeling, multi-reader adjudication, or none yet?
4. What is the deployment surface — inline CAD overlays at read time, worklist re-prioritization (triage), asynchronous secondary capture, or pre-populated structured findings the radiologist signs?
5. What regulatory pathway is intended — research-only, CDS-exempt enterprise tool, FDA-cleared SaMD (510(k) / De Novo / PMA), IDE, EU MDR / AI Act?
6. Vendor neutrality — is this targeted at one PACS/RIS/scanner family, or must it run across multiple vendors and through an orchestrator (Nuance PIN, Blackford, Calantic, Sirona, GE Edison, Philips ISP)?
7. What is the result-return channel — DICOM SR, DICOM SEG, secondary capture, encapsulated PDF, FHIR `DiagnosticReport`, HL7 v2 ORU^R01, or hanging-protocol modification?

---

## Related Skills

- **dicom-imaging**: DICOM tags, DIMSE, DICOMweb, SR/SEG/PR structure
- **fda-samd**: SaMD risk classification, 510(k)/De Novo, PCCP
- **clinical-ai-ml**: cross-cutting ML lifecycle, validation, drift
- **fhir-integration**: `ImagingStudy`, `DiagnosticReport`, `Observation`, `RiskAssessment`
- **hl7-v2**: ORM/ORU integration with legacy RIS
- **phi-handling**: anonymization, pixel-burn-in, expert determination
- **hipaa-compliance**: BAA, encryption, audit for cloud inference
