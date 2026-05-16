# Google Health Connect to FHIR Mapping

Google **Health Connect** is the current Android health data platform that replaces the deprecated Google Fit API for new app integrations. Health Connect is an on-device data store that aggregates health and fitness records across apps and wearables; each app requests **per-data-type permission** to read or write specific record types.

This reference covers the mapping from Health Connect record types to FHIR resources. **Verify the current Health Connect SDK and record-type list; Google adds and renames record types over time. Look up LOINC codes per parameter at design time.**

## Mapping principles

Same principles as HealthKit:

- Each Health Connect record maps to a FHIR `Observation` (or `Condition`, `MedicationStatement`, etc., for clinical types where applicable).
- `Observation.code` uses **LOINC**.
- `valueQuantity.unit` uses **UCUM**.
- `Observation.device` references a `Device` derived from the originating app metadata.
- Preserve original timestamps with full offset.
- For paired values (BP), use `Observation.component`.
- Always capture `Provenance` linking the FHIR resource to the originating Android app package name (e.g., `com.fitbit.FitbitMobile`) and the Health Connect provenance.

## Health Connect record types → FHIR

Health Connect organizes records by category. The mapping is parallel to HealthKit.

### Vital signs / cardiovascular

| Health Connect record type | FHIR Observation.code (LOINC — verify) | UCUM unit | Notes |
|----------------------------|-----------------------------------------|-----------|-------|
| `HeartRateRecord` | LOINC HR code (verify) | `/min` | Sample-level |
| `RestingHeartRateRecord` | LOINC resting HR code (verify) | `/min` | |
| `HeartRateVariabilityRmssdRecord` | LOINC HRV RMSSD code (verify) | `ms` | |
| `BloodPressureRecord` | LOINC BP panel code (verify) | `mm[Hg]` | Use components for systolic + diastolic |
| `OxygenSaturationRecord` | LOINC SpO2 code (verify) | `%` | |
| `RespiratoryRateRecord` | LOINC RR code (verify) | `/min` | |
| `BodyTemperatureRecord` | LOINC temp code (verify) | `Cel` | |
| `BasalBodyTemperatureRecord` | LOINC basal body temp (verify) | `Cel` | Often used for fertility tracking |

### Body composition

| Health Connect record type | FHIR Observation.code (LOINC — verify) | UCUM unit |
|----------------------------|-----------------------------------------|-----------|
| `WeightRecord` | LOINC body weight code (verify) | `kg` |
| `HeightRecord` | LOINC body height code (verify) | `m` |
| `BodyFatRecord` | LOINC body fat % code (verify) | `%` |
| `BoneMassRecord` | LOINC bone mass code (verify) | `kg` |
| `LeanBodyMassRecord` | LOINC lean body mass code (verify) | `kg` |
| `BodyWaterMassRecord` | LOINC body water mass code (verify) | `kg` |

### Activity

| Health Connect record type | FHIR Observation.code (LOINC — verify) | UCUM unit |
|----------------------------|-----------------------------------------|-----------|
| `StepsRecord` | LOINC steps code (verify) | `{steps}` |
| `DistanceRecord` | LOINC distance code (verify) | `m` |
| `ActiveCaloriesBurnedRecord` | LOINC active energy (verify) | `kcal` |
| `TotalCaloriesBurnedRecord` | LOINC total energy (verify) | `kcal` |
| `ElevationGainedRecord` | LOINC elevation code (verify) | `m` |
| `FloorsClimbedRecord` | LOINC flights code (verify) | `{count}` |
| `Vo2MaxRecord` | LOINC VO2 code (verify) | `mL/(kg.min)` |
| `PowerRecord` | LOINC power code (verify) | `W` |
| `SpeedRecord` | LOINC speed code (verify) | `m/s` |
| `ExerciseSessionRecord` | Workout / Exercise observation (verify code) | Duration in `min` |
| `ExerciseLapRecord` | Workout segment (verify) | |
| `ExerciseSegmentRecord` | Workout segment (verify) | |

### Sleep

| Health Connect record type | Mapping | Notes |
|----------------------------|---------|-------|
| `SleepSessionRecord` | LOINC sleep duration / sessions (verify) | Top-level session with start/end |
| Sleep stages within session | LOINC sleep stage codes (verify) | Awake / Light / Deep / REM as components or sub-records |

### Glucose / nutrition / fertility

| Health Connect record type | FHIR Observation.code (LOINC — verify) | UCUM unit | Notes |
|----------------------------|-----------------------------------------|-----------|-------|
| `BloodGlucoseRecord` | LOINC glucose (verify; fasting/random) | `mg/dL` or `mmol/L` | Always read unit |
| `NutritionRecord` | Nutrient codes (verify) | varies | Maps to multiple nutrient Observations or a structured intake |
| `HydrationRecord` | LOINC fluid intake (verify) | `L` | |
| `MenstruationFlowRecord` | LOINC menstrual codes (verify) | Categorical | |
| `MenstruationPeriodRecord` | LOINC code (verify) | Period start/end | |
| `OvulationTestRecord` | LOINC code (verify) | Categorical result | |
| `IntermenstrualBleedingRecord` | LOINC code (verify) | | |
| `CervicalMucusRecord` | LOINC code (verify) | Categorical | |
| `SexualActivityRecord` | LOINC code (verify) | Protected health information; treat sensitively |

### Other

| Health Connect record type | FHIR Observation.code (LOINC — verify) |
|----------------------------|-----------------------------------------|
| `WheelchairPushesRecord` | LOINC wheelchair pushes (verify) |
| `CyclingPedalingCadenceRecord` | LOINC cadence (verify) |

## Authorization

- Permission model is **per record type, per direction (read / write)** — analogous to HealthKit.
- Request only what you need.
- `PermissionController.getRequestPermissionResultContract` for the prompt.
- Users can grant read access without your app knowing whether data exists — the same opacity caveat as HealthKit applies. Do not infer signals from absence.

## Background reads

- Use `HealthConnectClient` with WorkManager / JobScheduler for periodic pulls.
- Health Connect itself does not push to apps; you pull on a schedule or when the user opens the app.
- Define a reasonable poll cadence — too frequent drains battery; too infrequent delays clinical action.

## Provenance

- Each Health Connect record carries metadata about the originating app (package name).
- Capture `Provenance.entity` referencing the originating package and `Provenance.agent` referencing Health Connect.
- A FHIR `Observation` derived from Health Connect should be traceable to: device (e.g., Fitbit Inspire 3), source app (Fitbit Android app), Health Connect, your ingest pipeline.

## Migration from Google Fit

- Google Fit APIs are deprecated for new apps; the Fit REST API and Android Fit APIs are being sunset. Verify current deprecation timeline.
- For older code reading from Fit:
  - Map existing Fit `DataType` constants to Health Connect record types.
  - Replicate the per-data-type permission UX.
  - Re-prompt users for Health Connect permission — Fit permissions do not transfer.
  - Plan a cutover window and dual-read period to verify data continuity.

## Device modeling

- Health Connect records carry minimal device metadata compared to HealthKit's `HKSourceRevision.productType`. Capture what is available and supplement from the originating app where possible.
- For a FHIR `Device`, use:
  - `Device.manufacturer` — derived from the source app's known manufacturer
  - `Device.deviceName` — e.g., the wearable model from the app's metadata
  - `Device.identifier` — Health Connect record ID + source package

## Verify-current sources

- Android Developer documentation — Health Connect SDK reference (current version)
- Google Health Connect release notes
- LOINC release notes (June, December)
- US Core IG `Observation` and `Vital Signs` profiles
- Mobile Health Working Group in HL7
