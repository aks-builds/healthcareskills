# DICOM Preparation for AI

Reference for getting DICOM studies from PACS into a state where you can train and evaluate a model without learning the scanner instead of the disease.

## Contents

- Anonymization (PS3.15 profile selection)
- Pixel-Data Burn-In Detection
- Vendor Quirks in Pixel Encoding
- Windowing and Intensity Normalization
- Resampling and Spatial Harmonization
- Quality Filters Before Training
- Provenance to Keep

---

## Anonymization

### Apply a Documented Profile

DICOM PS3.15 Annex E defines the Basic Application Confidentiality Profile and option sets. For an AI training set, a typical combination is:

- **Basic Application Confidentiality Profile** (required baseline)
- **Clean Pixel Data Option** (handle burned-in identifiers in the pixel array)
- **Clean Recognizable Visual Features Option** (defacing for head/neck CT and MR where face reconstruction is feasible)
- **Retain Longitudinal With Modified Dates Option** (preserve relative timing for follow-up studies)
- **Retain Patient Characteristics Option** (sex, age) only if scientifically required and consistent with your release framework

If you also need to release the dataset externally, layer Expert Determination on top: document methodology, residual re-identification risk, and the determiner's qualifications. Safe Harbor on structured metadata alone is rarely sufficient for imaging because pixels carry residual identifying information.

### Tag Handling

- **Direct identifiers** (PatientName, PatientID, IssuerOfPatientID, AccessionNumber): remove or replace with a pseudonymous mapping held in a separate, access-controlled side table.
- **UIDs** (StudyInstanceUID, SeriesInstanceUID, SOPInstanceUID): remap deterministically with an HMAC keyed by a project secret so longitudinal joins survive without revealing identity.
- **Dates**: apply a per-patient random offset within a fixed range so relative intervals are preserved.
- **Private tags**: vendor private groups (odd group numbers) often carry useful acquisition detail but may also contain operator or patient information. Whitelist per vendor; default to removing the group.
- **Institution / station / equipment**: keep Manufacturer, ManufacturerModelName, SoftwareVersions for subgroup analysis; consider removing or hashing InstitutionName, StationName, OperatorsName if the release framework requires it.

### Reports Separately

Accompanying reports (DICOM SR, encapsulated PDF, RIS text) frequently contain more PHI than pixels. De-identify reports with a separate pipeline (Philter, NeuroNER, AWS Comprehend Medical, Azure Health De-id, Google DLP) and validate residual leakage with random spot-checks.

---

## Pixel-Data Burn-In Detection

### When to Worry

Burned-in identifiers appear most often in:

- **Ultrasound** (US): patient name and date routinely burned into the corner.
- **Mammography** (MG): laterality, view, technologist annotations.
- **Fluoroscopy / angiography** (XA, RF): annotation overlays.
- **Screen captures and secondary captures** (SC, derived series): often a snapshot of a full viewer window including patient banner.
- **Some MR sequences**: vendor-specific annotation burn-in.
- **Portable CR/DX**: technologist annotations from the cart.

### Detection Approach

1. **Trust BurnedInAnnotation (0028,0301) only as a hint.** It is frequently `NO` even when annotations are present, and vice versa.
2. **Run OCR on a corner-region crop** (e.g., Tesseract over the top-left and top-right 256 pixels) for any modality on the suspect list.
3. **Or train a small burned-in-text detector** (binary classifier on small patches) — useful at scale.
4. **Disposition**: mask (paint over with a constant value) or exclude. Masking preserves the study at the cost of a region of the image; exclusion preserves image integrity at the cost of dataset size. Track which you did.

### Validation

Random-sample audited review of the post-anonymization corpus is non-negotiable. A 1% sample reviewed by a clinician or trained data engineer with explicit re-identification criteria catches both burn-in leakage and tag leakage.

---

## Vendor Quirks in Pixel Encoding

### PhotometricInterpretation

`PhotometricInterpretation (0028,0004)` can be:

- `MONOCHROME2` — minimum sample value is black (standard).
- `MONOCHROME1` — minimum sample value is white (inverted). Common on older CR and on some legacy archives.
- `RGB`, `YBR_FULL`, `YBR_FULL_422`, `PALETTE COLOR` — color modalities.

If you do not handle MONOCHROME1, your model trains on inverted images for a subset of studies and you discover the bug at external validation.

### Rescale Slope and Intercept (CT, PT)

For CT, the pixel array is **not** Hounsfield Units. You must apply:

```
HU = pixel_value * RescaleSlope + RescaleIntercept
```

Tags: `(0028,1052) RescaleIntercept`, `(0028,1053) RescaleSlope`. Always apply these before any windowing or normalization. Skipping this step is a common silent bug because the pixel values still look "image-like."

For PET, also account for `RescaleType` (e.g., `US` for SUV-units) and units conversion (SUV body weight vs. lean body mass).

### Bit Depth and Signed Values

- `(0028,0100) BitsAllocated`, `(0028,0101) BitsStored`, `(0028,0102) HighBit`, `(0028,0103) PixelRepresentation` (0 unsigned, 1 signed).
- A CT scanner storing 12 bits in 16-bit pixels with a sign bit is common. Use a tested DICOM library (pydicom, GDCM, DCMTK) to materialize the pixel array rather than hand-rolling the bit math.

### Modality-Specific

- **MR**: intensities are not calibrated across scanners or sequences. Do not assume that intensity X on one scanner equals intensity X on another. Consider z-score per volume, histogram matching, or N4ITK bias-field correction; the right answer depends on the task.
- **MG**: presentation intent (`PresentationIntentType`) distinguishes "For Presentation" (processed for display) and "For Processing" (raw). Pick one and stay consistent — they are different images.
- **CR/DX**: `(0018,5101) ViewPosition` (PA/AP/LAT) and `(0020,0062) ImageLaterality` are often more reliable than `BodyPartExamined`.

---

## Windowing and Intensity Normalization

### CT

Windowing converts HU to display intensity:

```
display = clip((HU - WindowCenter + 0.5) / (WindowWidth - 1) + 0.5, 0, 1)
```

Common windows: lung (C=-600, W=1500), mediastinum (C=40, W=400), abdomen (C=50, W=400), bone (C=400, W=1500), brain (C=40, W=80), stroke (C=35, W=10–30), bone-window head (C=600, W=2800).

For training, two patterns are common:

1. **Single task-appropriate window**, normalized to [0,1] or z-scored.
2. **Multiple windows as channels** (e.g., lung + mediastinum + bone as three channels), letting the network choose what to attend to.

Always apply rescale slope/intercept first.

### MR

There is no universal MR window. Options:

- **Per-volume z-score** (mean 0, std 1 within the brain mask or foreground).
- **Histogram matching** to a reference template (Nyul, ITK).
- **N4ITK bias-field correction** before intensity normalization for spatial inhomogeneity.

For multi-sequence inputs (T1, T2, FLAIR, DWI), normalize each sequence independently then stack as channels.

### CR / DX / MG

- Invert if `PhotometricInterpretation == MONOCHROME1`.
- Then either per-image min-max to [0,1], CLAHE, or a vendor-provided LUT (`(0028,3010) VOI LUT Sequence`).
- For mammography, the choice of "For Presentation" vs "For Processing" dominates; pick one and stick with it.

---

## Resampling and Spatial Harmonization

### Why Resample

Scanners produce voxels at varying spacing. Without harmonization:

- A 3D model sees the same anatomic structure as different sizes across studies.
- Aggregations across slices (e.g., 3D segmentation losses) are biased toward studies with thinner slices.

### Target Choices

- **2D classification (CXR, MG)**: resample to a fixed pixel grid (e.g., 1024×1024 for CXR, 2048×2048 for MG). Be careful with aspect-ratio handling — pad rather than distort.
- **3D CT segmentation**: 1×1×1 mm isotropic is a common default; lung-nodule work sometimes uses 0.7×0.7×1 mm.
- **MR**: depends on sequence and target structure; respect the in-plane vs. through-plane anisotropy unless you have a reason to make it isotropic.

### Interpolation

- **Image**: linear or B-spline (cubic). Linear is faster, B-spline preserves more detail.
- **Labels (segmentation masks)**: nearest-neighbor only. Any other interpolation will create non-integer labels.

### Tools

- **SimpleITK** / **ITK** — image-side resampling, registration.
- **nibabel** — NIfTI I/O (when you convert to NIfTI for training).
- **MONAI transforms** — DICOM-aware augmentation and resampling.
- **dcm2niix** — convert DICOM to NIfTI when the training pipeline prefers NIfTI.

---

## Quality Filters Before Training

Before any model run, filter the cohort on:

- **Modality** matches the intent (`(0008,0060) Modality`).
- **BodyPartExamined** plus protocol heuristics on `StudyDescription` / `SeriesDescription` — `BodyPartExamined` is often miscoded or missing.
- **PhotometricInterpretation** handled.
- **Slice thickness within an acceptable range** for 3D tasks.
- **Reconstruction kernel** (for CT) within an acceptable family.
- **Field strength** (for MR) — 1.5T and 3T behave differently; consider stratifying or harmonizing.
- **Image dimensions** above a minimum (reject thumbnails, key-image series).
- **Localizers and dose-report series** removed.

Document the inclusion / exclusion as part of the dataset card — both for reproducibility and so an external site can apply the same filter to their data at validation time.

---

## Provenance to Keep

Even with anonymization, keep (in a secure, access-controlled side table):

- Pre-anonymization `StudyInstanceUID` → post-anonymization UID mapping.
- Source PACS / AE / VNA identifier.
- Retrieval timestamp and the user / service account that pulled the study.
- The PS3.15 option set actually applied (profile + options selected).
- The git commit hash of the anonymization pipeline.
- The mapping from clinical patient identifier to study-set patient identifier (for re-identification only by an authorized custodian under documented procedure).

For the public artifact (the anonymized dataset itself), keep the post-anonymization UIDs, scanner metadata retained, date offsets at the per-patient granularity (not the actual offset value), and the version of the anonymization profile applied.
