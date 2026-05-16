# DICOM Tags Reference

Quick reference to the DICOM data elements you'll meet most often when building integrations, query/retrieve, anonymization, and reporting pipelines. Tags are written `(GGGG,EEEE)` — group, element. **Always verify against the current published DICOM standard (PS3.6 Data Dictionary) when in doubt** — VRs and definitions change between releases.

## Contents

- Identity (Patient, Issuer, Study, Series, Instance)
- Study-Level
- Series-Level
- Instance / SOP-Level
- Equipment and Acquisition
- Image / Pixel Description
- Patient Position and Orientation
- Modality-Specific Highlights (CT, MR, US, MG, NM/PT, RT, SR)
- Frame of Reference and Spatial Registration
- Procedure and Performing Workflow (MWL / MPPS)
- Structured Report Specific Tags
- Privacy / De-identification Targets
- VR Cheat Sheet

---

## Identity

### Patient

| Tag | VR | Name | Notes |
|-----|----|------|-------|
| (0010,0010) | PN | PatientName | Format `Last^First^Middle^Prefix^Suffix` |
| (0010,0020) | LO | PatientID | Stable per assigning authority |
| (0010,0021) | LO | IssuerOfPatientID | Disambiguates IDs across sites |
| (0010,0024) | SQ | IssuerOfPatientIDQualifiersSequence | Universal Entity ID, etc. |
| (0010,0030) | DA | PatientBirthDate | YYYYMMDD |
| (0010,0032) | TM | PatientBirthTime | |
| (0010,0040) | CS | PatientSex | M / F / O |
| (0010,1000) | LO | OtherPatientIDs | Deprecated in newer editions; use OtherPatientIDsSequence |
| (0010,1002) | SQ | OtherPatientIDsSequence | Replacement for OtherPatientIDs |
| (0010,1010) | AS | PatientAge | nnn{D,W,M,Y} |
| (0010,1020) | DS | PatientSize | Height in m |
| (0010,1030) | DS | PatientWeight | kg |
| (0010,2160) | SH | EthnicGroup | |
| (0010,4000) | LT | PatientComments | Often holds free-text PHI |

---

## Study-Level

| Tag | VR | Name | Notes |
|-----|----|------|-------|
| (0020,000D) | UI | StudyInstanceUID | Required, globally unique |
| (0020,0010) | SH | StudyID | RIS-assigned alphanumeric |
| (0008,0020) | DA | StudyDate | |
| (0008,0030) | TM | StudyTime | |
| (0008,0050) | SH | AccessionNumber | RIS order identifier |
| (0008,0090) | PN | ReferringPhysicianName | |
| (0008,1030) | LO | StudyDescription | |
| (0008,1032) | SQ | ProcedureCodeSequence | |
| (0008,1048) | PN | PhysiciansOfRecord | |
| (0008,1060) | PN | NameOfPhysiciansReadingStudy | |

---

## Series-Level

| Tag | VR | Name | Notes |
|-----|----|------|-------|
| (0020,000E) | UI | SeriesInstanceUID | Required, globally unique |
| (0020,0011) | IS | SeriesNumber | |
| (0008,0021) | DA | SeriesDate | |
| (0008,0031) | TM | SeriesTime | |
| (0008,0060) | CS | Modality | CT, MR, CR, DX, US, NM, PT, MG, RF, XA, RT*, etc. |
| (0008,103E) | LO | SeriesDescription | |
| (0008,1050) | PN | PerformingPhysicianName | |
| (0008,1070) | PN | OperatorsName | |
| (0018,5100) | CS | PatientPosition | HFS, HFP, FFS, FFP, HFDR, HFDL, etc. |

---

## Instance / SOP-Level

| Tag | VR | Name | Notes |
|-----|----|------|-------|
| (0008,0016) | UI | SOPClassUID | What kind of object |
| (0008,0018) | UI | SOPInstanceUID | Unique per instance |
| (0008,0023) | DA | ContentDate | |
| (0008,0033) | TM | ContentTime | |
| (0020,0013) | IS | InstanceNumber | |
| (0008,0008) | CS | ImageType | e.g., `ORIGINAL\PRIMARY\AXIAL` |

---

## Equipment and Acquisition

| Tag | VR | Name | Notes |
|-----|----|------|-------|
| (0008,0070) | LO | Manufacturer | |
| (0008,0080) | LO | InstitutionName | |
| (0008,0081) | ST | InstitutionAddress | Often holds PHI-adjacent info |
| (0008,1010) | SH | StationName | |
| (0008,1040) | LO | InstitutionalDepartmentName | |
| (0008,1090) | LO | ManufacturerModelName | |
| (0018,1000) | LO | DeviceSerialNumber | |
| (0018,1020) | LO | SoftwareVersions | |
| (0018,1030) | LO | ProtocolName | |

---

## Image / Pixel Description

| Tag | VR | Name | Notes |
|-----|----|------|-------|
| (0028,0002) | US | SamplesPerPixel | 1 (mono) or 3 (RGB) |
| (0028,0004) | CS | PhotometricInterpretation | MONOCHROME2, RGB, YBR_FULL_422, etc. |
| (0028,0008) | IS | NumberOfFrames | >1 for multi-frame |
| (0028,0010) | US | Rows | |
| (0028,0011) | US | Columns | |
| (0028,0030) | DS | PixelSpacing | mm row\mm col |
| (0028,0100) | US | BitsAllocated | |
| (0028,0101) | US | BitsStored | |
| (0028,0102) | US | HighBit | |
| (0028,0103) | US | PixelRepresentation | 0 unsigned, 1 signed |
| (0028,1050) | DS | WindowCenter | |
| (0028,1051) | DS | WindowWidth | |
| (0028,1052) | DS | RescaleIntercept | CT HU calculation |
| (0028,1053) | DS | RescaleSlope | CT HU calculation |
| (7FE0,0010) | OB/OW | PixelData | Encapsulated when compressed |

---

## Patient Position and Orientation

| Tag | VR | Name | Notes |
|-----|----|------|-------|
| (0020,0032) | DS | ImagePositionPatient | x\y\z mm of voxel (0,0) |
| (0020,0037) | DS | ImageOrientationPatient | row-cosine x\y\z, col-cosine x\y\z |
| (0018,0050) | DS | SliceThickness | mm |
| (0018,0088) | DS | SpacingBetweenSlices | mm |
| (0020,1041) | DS | SliceLocation | |
| (0020,0052) | UI | FrameOfReferenceUID | Same UID = same patient spatial frame |

---

## Modality-Specific Highlights

### CT

| Tag | Name |
|-----|------|
| (0018,0060) | KVP |
| (0018,1150) | ExposureTime |
| (0018,1151) | XRayTubeCurrent |
| (0018,1152) | Exposure |
| (0018,9345) | CTDIvol |
| (0018,9346) | CTDIPhantomTypeCodeSequence |
| (0018,0090) | DataCollectionDiameter |
| (0018,1100) | ReconstructionDiameter |
| (0018,9302) | AcquisitionType (HELICAL, AXIAL, etc.) |

CT dose info should also live in the RDSR (Radiation Dose Structured Report) per IHE REM.

### MR

| Tag | Name |
|-----|------|
| (0018,0080) | RepetitionTime (TR) |
| (0018,0081) | EchoTime (TE) |
| (0018,0082) | InversionTime (TI) |
| (0018,0087) | MagneticFieldStrength |
| (0018,1314) | FlipAngle |
| (0018,0023) | MRAcquisitionType (2D, 3D) |
| (0018,0024) | SequenceName |
| (0018,0025) | AngioFlag |
| (0018,0091) | EchoTrainLength |

### US (Ultrasound)

| Tag | Name |
|-----|------|
| (0028,0014) | UltrasoundColorDataPresent |
| (0008,2129) | NumberOfStages |
| (0018,5022) | TransducerType |
| (0018,6011) | SequenceOfUltrasoundRegions |

### MG (Mammography)

| Tag | Name |
|-----|------|
| (0008,0008) | ImageType (e.g., `DERIVED\PRIMARY\TOMO\NONE` for tomosynthesis) |
| (0018,5101) | ViewPosition (CC, MLO, ML, LM, etc.) |
| (0020,0062) | ImageLaterality (L, R) |
| (0028,1040) | PixelIntensityRelationship |

### NM / PT

| Tag | Name |
|-----|------|
| (0018,1072) | RadiopharmaceuticalStartTime |
| (0018,1074) | RadionuclideTotalDose |
| (0018,1075) | RadionuclideHalfLife |
| (0018,1076) | RadionuclidePositronFraction |
| (0054,0410) | PatientOrientationCodeSequence |

### RT (Radiation Therapy)

| Tag | Name |
|-----|------|
| (3004,*) | DoseGrid, RTDoseSequence tags |
| (3006,*) | ROIContourSequence, StructureSetROISequence, etc. |
| (300A,*) | RTPlan items (Beam, ControlPoint, Brachy, etc.) |

### SR (Structured Report) Common Tags

| Tag | Name |
|-----|------|
| (0040,A040) | ValueType (CONTAINER, TEXT, NUM, CODE, IMAGE, COMPOSITE, etc.) |
| (0040,A043) | ConceptNameCodeSequence |
| (0040,A050) | ContinuityOfContent |
| (0040,A730) | ContentSequence |
| (0040,A040)/A168 | ConceptCodeSequence (for CODE items) |
| (0040,A300) | MeasuredValueSequence |
| (0040,A30A) | NumericValue |
| (0040,08EA) | MeasurementUnitsCodeSequence (UCUM) |

---

## Procedure and Performing Workflow (MWL / MPPS)

MWL response and MPPS share many tags with general study data, plus:

| Tag | Name |
|-----|------|
| (0040,0100) | ScheduledProcedureStepSequence |
| (0040,1001) | RequestedProcedureID |
| (0032,1060) | RequestedProcedureDescription |
| (0040,0001) | ScheduledStationAETitle |
| (0040,0002) | ScheduledProcedureStepStartDate |
| (0040,0003) | ScheduledProcedureStepStartTime |
| (0040,0009) | ScheduledProcedureStepID |
| (0040,0252) | PerformedProcedureStepStatus (IN PROGRESS, COMPLETED, DISCONTINUED) |
| (0040,0254) | PerformedProcedureStepDescription |

---

## Privacy / De-identification Targets

PS3.15 Annex E enumerates many tags requiring action under the Basic Application Confidentiality Profile. Frequent targets include:

- All PatientName / PatientID / PatientBirthDate / PatientSex variants
- ReferringPhysicianName, PerformingPhysicianName, OperatorsName, NameOfPhysiciansReadingStudy
- AccessionNumber, StudyID
- InstitutionName, InstitutionAddress, StationName
- DeviceSerialNumber
- PatientComments (and any free-text comment fields)
- Private tags (group element with odd group number) — most profiles strip unless explicitly retained
- UIDs — typically regenerated under a deterministic mapping
- Date/time fields — shifted or zeroed depending on option
- Burned-in pixel data — requires OCR or manual review

Always use a vetted library (DCMTK, pixelmed, dicom-anonymizer, GDCM) and document which profile + options were applied.

---

## VR Cheat Sheet

| VR | Meaning |
|----|---------|
| AE | Application Entity (max 16) |
| AS | Age String |
| AT | Attribute Tag |
| CS | Code String |
| DA | Date YYYYMMDD |
| DS | Decimal String |
| DT | DateTime |
| FL / FD | Float / Double |
| IS | Integer String |
| LO / LT | Long String / Long Text |
| OB / OW / OF / OD | Other Byte / Word / Float / Double |
| PN | Person Name |
| SH / ST | Short String / Short Text |
| SL / SS / UL / US | Signed/Unsigned Long/Short |
| SQ | Sequence of Items |
| TM | Time HHMMSS[.frac] |
| UI | Unique Identifier (UID) |
| UN | Unknown |
| UR | Universal Resource Identifier |
| UT | Unlimited Text |
| UV / SV | Unsigned/Signed Very Long (64-bit) |

---

## Tips

- Always verify SOP Class UIDs and Transfer Syntax UIDs against the current DICOM standard — they are long URN-style strings and easy to mistype.
- StudyInstanceUID / SeriesInstanceUID / SOPInstanceUID are **globally unique and immutable** — never reuse across studies.
- When a tag has a Sequence VR (SQ), every nested item must have its own Item delimiter; nested datasets follow the same VR/encoding rules.
- PatientID alone collides across sites — pair with IssuerOfPatientID (0010,0021) for cross-enterprise correlation.
- Burned-in PHI is the silent failure mode — tag-based de-identification will miss it.
