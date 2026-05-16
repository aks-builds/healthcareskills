# Apple HealthKit to FHIR Mapping

This reference summarizes the mapping from Apple HealthKit data types to FHIR resources. Use it as a starting structure; **always look up the current LOINC code per parameter at design time — do not guess from memory.**

## Mapping principles

- Each HealthKit sample maps to a FHIR `Observation` (occasionally `Condition` for diagnoses, `MedicationStatement` for medications, `AllergyIntolerance` for allergies — for `HKClinicalType` records).
- The `Observation.code` uses **LOINC**; the `valueQuantity.unit` uses **UCUM**.
- The `Observation.device` references a `Device` resource derived from `HKSourceRevision` (productType, version).
- The `Observation.method` distinguishes sensor vs. manual entry.
- Preserve the original time zone — HealthKit gives start/end in absolute time; capture the user's time zone separately if you need it for clinical interpretation.
- For paired values (blood pressure), use `Observation.component` — do not emit separate Observations for systolic and diastolic.
- Always capture `Provenance` linking the FHIR resource back to its HealthKit source bundle ID, so a clinician can see "this BP came from Withings via HealthKit" rather than just "this is a BP."

## HKQuantityType → FHIR Observation

| HealthKit identifier | FHIR Observation.code (LOINC — verify current) | UCUM unit | Notes |
|----------------------|------------------------------------------------|-----------|-------|
| `HKQuantityTypeIdentifierStepCount` | LOINC steps code (verify) | `{steps}` | Aggregated; consider day-bucket vs. minute-by-minute |
| `HKQuantityTypeIdentifierHeartRate` | LOINC HR code (verify) | `/min` | Common: 8867-4 — verify |
| `HKQuantityTypeIdentifierRestingHeartRate` | LOINC resting HR code (verify) | `/min` | |
| `HKQuantityTypeIdentifierHeartRateVariabilitySDNN` | LOINC HRV SDNN code (verify) | `ms` | |
| `HKQuantityTypeIdentifierOxygenSaturation` | LOINC SpO2 code (verify) | `%` | Common: 59408-5 — verify |
| `HKQuantityTypeIdentifierBodyMass` | LOINC body weight code (verify) | `kg` (convert from lb) | Common: 29463-7 — verify |
| `HKQuantityTypeIdentifierHeight` | LOINC body height code (verify) | `cm` | |
| `HKQuantityTypeIdentifierBodyMassIndex` | LOINC BMI code (verify) | `kg/m2` | |
| `HKQuantityTypeIdentifierBodyFatPercentage` | LOINC body fat % code (verify) | `%` | |
| `HKQuantityTypeIdentifierBloodPressureSystolic` | Use BP panel with component (verify panel code) | `mm[Hg]` | Pair with diastolic; use component |
| `HKQuantityTypeIdentifierBloodPressureDiastolic` | Same panel, component | `mm[Hg]` | Pair with systolic |
| `HKQuantityTypeIdentifierBloodGlucose` | LOINC glucose code (verify; specify fasting/random) | `mg/dL` or `mmol/L` | Always read unit |
| `HKQuantityTypeIdentifierBodyTemperature` | LOINC temp code (verify) | `Cel` or `[degF]` | |
| `HKQuantityTypeIdentifierRespiratoryRate` | LOINC RR code (verify) | `/min` | |
| `HKQuantityTypeIdentifierActiveEnergyBurned` | LOINC active energy code (verify) | `kcal` | |
| `HKQuantityTypeIdentifierBasalEnergyBurned` | LOINC basal energy code (verify) | `kcal` | |
| `HKQuantityTypeIdentifierDistanceWalkingRunning` | LOINC distance code (verify) | `m` | |
| `HKQuantityTypeIdentifierFlightsClimbed` | LOINC flights code (verify) | `{count}` | |
| `HKQuantityTypeIdentifierPeripheralPerfusionIndex` | LOINC PPI code (verify) | `%` | |
| `HKQuantityTypeIdentifierVO2Max` | LOINC VO2 code (verify) | `mL/(kg.min)` | |
| `HKQuantityTypeIdentifierForcedExpiratoryVolume1` | LOINC FEV1 code (verify) | `L` | Spirometer |
| `HKQuantityTypeIdentifierForcedVitalCapacity` | LOINC FVC code (verify) | `L` | |
| `HKQuantityTypeIdentifierPeakExpiratoryFlowRate` | LOINC PEFR code (verify) | `L/min` | |
| `HKQuantityTypeIdentifierInhalerUsage` | Adherence / event code (verify) | `{count}` | Smart inhaler |
| `HKQuantityTypeIdentifierInsulinDelivery` | LOINC insulin dose code (verify) | `[iU]` | |
| `HKQuantityTypeIdentifierWalkingHeartRateAverage` | LOINC walking HR code (verify) | `/min` | |
| `HKQuantityTypeIdentifierEnvironmentalAudioExposure` | LOINC sound level code (verify) | `dB` | |

## HKCategoryType → FHIR Observation (categorical)

| HealthKit identifier | Mapping | Notes |
|----------------------|---------|-------|
| `HKCategoryTypeIdentifierSleepAnalysis` | LOINC sleep duration / stage codes (verify) | iOS 16+ exposes stages (REM, core, deep, awake). Use multiple Observations or a single Observation with components. |
| `HKCategoryTypeIdentifierMindfulSession` | Activity / event code (verify) | Duration as `valueQuantity` in `min` |
| `HKCategoryTypeIdentifierMenstrualFlow` | LOINC menstrual codes (verify) | Categorical value |
| `HKCategoryTypeIdentifierOvulationTestResult` | LOINC code (verify) | Categorical |
| `HKCategoryTypeIdentifierIrregularHeartRhythmEvent` | Use AFib observation / condition path | The FDA-cleared AFib notification — a clearance-scoped signal, document accordingly |
| `HKCategoryTypeIdentifierHighHeartRateEvent` | Event observation (verify code) | |
| `HKCategoryTypeIdentifierLowHeartRateEvent` | Event observation (verify code) | |
| `HKCategoryTypeIdentifierAudioExposureEvent` | Environmental event (verify) | |

## HKCorrelationType

| HealthKit identifier | Mapping |
|----------------------|---------|
| `HKCorrelationTypeIdentifierBloodPressure` | Single FHIR `Observation` with `component[]` for systolic + diastolic. Do not split. |
| `HKCorrelationTypeIdentifierFood` | Multiple nutrient Observations under a meal context, or `NutritionIntake` (if profile available). |

## HKWorkout

Maps to a FHIR `Observation` with a workout code, or to a fitness-specific profile if available. Include:

- Workout activity type (running, cycling, HIIT, etc.) → coded value
- Start and end (`effectivePeriod`)
- Duration (`valueQuantity` in `min`)
- Energy burned, distance, average HR — as components or linked Observations

## HKClinicalType (Apple Health Records)

`HKClinicalType` records come from connected providers via SMART on FHIR patient launch. They are already FHIR resources. Map directly:

| HealthKit clinical type | FHIR resource |
|-------------------------|---------------|
| `HKClinicalTypeIdentifierAllergyRecord` | `AllergyIntolerance` |
| `HKClinicalTypeIdentifierConditionRecord` | `Condition` |
| `HKClinicalTypeIdentifierImmunizationRecord` | `Immunization` |
| `HKClinicalTypeIdentifierLabResultRecord` | `Observation` (lab) |
| `HKClinicalTypeIdentifierMedicationRecord` | `MedicationStatement` |
| `HKClinicalTypeIdentifierProcedureRecord` | `Procedure` |
| `HKClinicalTypeIdentifierVitalSignRecord` | `Observation` (vitals) |

Use these as-is; do not re-derive — they are the provider's authoritative records.

## Device and Provenance

For each FHIR `Observation`, capture:

- `Observation.device` → `Device` derived from `HKSourceRevision.productType` (e.g., "iPhone14,5", "Watch6,3") and `HKSourceRevision.version`.
- `Provenance.entity` → references to the source app bundle ID (e.g., `com.withings.wiscale2`) and HealthKit itself.
- `Provenance.agent.who` → the app that wrote into HealthKit.

This is what lets a clinician later see: "BP from Withings BPM Connect via HealthKit, written 2025-09-12 14:32 local, received by the FHIR store 2025-09-12 14:37."

## Read-permission opacity (important pitfall)

HealthKit lets the user grant **read access without telling the app** that any data exists. The app cannot tell the difference between "permission denied" and "no data" for read scopes. Design implications:

- Do not infer a clinical signal from "no data." Treat absence as ambiguous.
- Surface to the patient: "If you don't see your data here, check that you've granted access in Health → Sharing."
- Do not gate clinical decisions on the absence of a wearable signal.

## Authorization request pattern

Bundle the HealthKit authorization request at a clear in-app moment (e.g., post-onboarding "Connect your Apple Health data" screen). Request only the types you need. Re-request when adding new data types.

## Background delivery

- `HKObserverQuery` + `enableBackgroundDelivery` allow your app to be woken when new data arrives.
- Requires HealthKit background-delivery entitlement.
- Use sparingly — background wakes still consume battery.

## Verify-current sources

- Apple Developer documentation — HealthKit framework and HKQuantityType identifiers (current iOS version)
- LOINC release notes (typically June and December)
- HL7 US Core profiles for `Vital Signs` and `Observation`
- HL7 implementation guide for wearables / patient-generated data (if applicable)
- Mobile Health (mHealth) Working Group resources in HL7
