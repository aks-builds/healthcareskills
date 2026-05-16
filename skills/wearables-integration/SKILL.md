---
name: wearables-integration
description: When the user wants to integrate consumer wearables or fitness devices into a clinical or health-app workflow. Also use when the user mentions "Apple HealthKit," "HKQuantityType," "HKSampleType," "Apple Health Records," "Health Connect," "Google Health Connect," "Google Fit," "Fitbit Web API," "Garmin Health API," "Oura," "Whoop," "Dexcom," "Abbott LibreView," "Medtronic CareLink," "Withings," "CGM integration," "BLE health profile," "GATT," "Heart Rate Profile," "Blood Pressure Profile," "Glucose Profile," "Body Composition Service," "Health Device Profile," "HDP," or "consumer wearable BAA." For clinical-grade RPM with billable CPT codes see remote-patient-monitoring. For FHIR Observation modeling see fhir-integration.
metadata:
  version: 1.0.0
---

# Wearables Integration

You are an expert in integrating consumer wearables and BLE health devices into clinical workflows. Your goal is to help engineers pull data from platforms like Apple HealthKit, Google Health Connect, Fitbit, Garmin, Oura, Whoop, Dexcom/Libre, Withings, and direct BLE devices — and decide what to do with it once it lands.

## Initial Assessment

Read `.agents/healthcare-context.md` first (fall back to `.claude/healthcare-context.md`). Pay attention to:

- Whether you are building a patient-facing app, a clinician-facing dashboard, or both.
- Wellness vs. medical-grade intent for each data type.
- BAA status with each platform — most consumer platforms do **not** sign BAAs.
- Mobile platforms supported (iOS, Android, both, web).

If absent, ask which wearables, what conditions, who is the audience, and whether the platform must be HIPAA-eligible end-to-end.

---

## Consumer vs. Medical-Grade

A core engineering decision before any code is written.

- **Consumer wearable data** (Apple Watch HR, Fitbit steps, Oura sleep stages) is typically not FDA-cleared for diagnostic use. Treat it as supportive context, not as a clinical signal you act on.
- **Medical-grade signals** can come from the same vendors in specific clearance scopes — e.g., Apple Watch ECG and AFib notification (FDA-cleared), Dexcom CGM, Abbott Libre. Use only the cleared signal for clinical decisions and confirm labeled use.
- For RPM billing under CMS 99453/99454, the device generally must be a **medical device** under FFDCA 201(h). Consumer wellness wearables typically do not qualify. See `remote-patient-monitoring`.

---

## Apple HealthKit (iOS / iPadOS / watchOS)

The HealthKit framework on iOS gives the patient's app access to read and write a wide range of health data **with the user's permission, per data type**.

### Core types

- `HKQuantityType` — scalar values (steps, heart rate, body mass, blood glucose, blood pressure systolic/diastolic, oxygen saturation, etc.)
- `HKCategoryType` — categorical (sleep analysis, mindful sessions, menstrual flow)
- `HKCorrelationType` — paired values (blood pressure correlation)
- `HKWorkout` — workouts with sessions, distance, energy
- `HKClinicalType` (Health Records) — clinical records ingested via SMART on FHIR from connected providers
- `HKDocumentType` — CDA documents

### Authorization

- Permissions are **per data type** and **direction-specific** (read vs. write).
- The user can grant **read access without telling the app** — the app cannot tell the difference between "permission denied" and "no data" for read scopes. Design accordingly.
- Request only the types you need. Bundle the authorization request at a clear in-app moment.
- Background delivery requires `HKObserverQuery` + `enableBackgroundDelivery` and entitlements.

### HealthKit-to-FHIR

Map HealthKit samples to FHIR `Observation`:

- HK quantity sample → `Observation.valueQuantity` (use UCUM units)
- HK source/device → `Observation.device` (preserve the source — Apple Watch, third-party app, manual entry)
- HK `sourceRevision.productType` and `sourceRevision.version` → `Device.manufacturer`, `Device.deviceName`
- LOINC code → look up the current LOINC code per parameter; do not guess
- Time zone → preserve the original time zone; HealthKit gives you start/end dates in UTC

Persist provenance (the HK source bundle ID) so the clinician can see "this BP came from Withings via HealthKit," not just "this is a BP."

### Apple Health Records & SMART on FHIR for Patients

- Apple Health Records lets patients download their clinical records from connected providers into the iPhone Health app via SMART on FHIR (patient launch).
- Your app can read these as `HKClinicalType` records (FHIR resources serialized).
- Pattern: app requests `HKClinicalType` permission, reads Conditions/Medications/Observations/AllergyIntolerance/Procedures/Immunizations, displays per-source.

---

## Google Health Connect (Android)

Health Connect is the current Android health data platform. It **replaces the deprecated Google Fit API** for new apps.

### Core concepts

- A single on-device store that aggregates data across apps and wearables.
- Permission model is **per data type**, similar to HealthKit.
- Data types include heart rate, steps, BP, blood glucose, oxygen saturation, sleep, exercise sessions, body composition, etc.
- Background reads use `HealthConnectClient` and JobScheduler patterns.

### Migration from Google Fit

- Google Fit APIs are being deprecated / sunset (Fit REST API and Android Fit APIs). New apps should use Health Connect.
- For older code reading from Fit, plan a migration; do not start new integrations on Fit.

### Mapping to FHIR

Same principles as HealthKit — Health Connect record → FHIR `Observation` with preserved provenance to the originating app or device.

---

## Fitbit Web API

- OAuth 2.0 with PKCE for user authorization.
- Endpoints per data scope (activity, heart, sleep, weight, etc.).
- Rate limits per token; design for batched daily pulls + on-demand syncs.
- Subscriptions: Fitbit pushes a notification when new data is available; you then call back.
- Note: Google acquired Fitbit; verify the current state of the API, EU data restrictions, and any migration to Health Connect / other endpoints.

---

## Garmin Health, Oura, Whoop, Withings

| Vendor | API style | Common data |
|--------|-----------|-------------|
| Garmin Health | OAuth 1.0a (older) or 2.0; partner program | Daily summaries, sleep, HRV, stress, training load |
| Oura | OAuth 2.0; v2 API | Sleep, readiness, activity, HRV, temperature |
| Whoop | OAuth 2.0 | Strain, recovery, sleep, HR, HRV |
| Withings | OAuth 2.0 | Weight, BP, sleep, temperature, ECG |

For each, confirm:

- Partner / business agreement required?
- BAA available (usually **no** for consumer wearable vendors — see "BAA gaps" below).
- Rate limits, retention windows, and historical-data access.
- Notification / webhook model.

---

## Continuous Glucose Monitors

CGMs are typically medical-device-grade and have stricter terms.

- **Dexcom** — Dexcom API (real-time and EGV API), separate partner / clinical-use access. CGM real-time data requires Dexcom partner agreement.
- **Abbott LibreView** — clinician-facing dashboard with API access for partners.
- **Medtronic CareLink** — pump + CGM data; partner access required.

For CGM data feeding clinical decision support, verify the **labeled use** of the signal you pull and the partner agreement scope.

---

## BLE GATT Health Device Profiles

For direct BLE integrations (without a vendor cloud), the Bluetooth SIG defines standard profiles and services. Use the standard profile when available — vendors generally implement them.

| Profile / Service | Acronym | Used for |
|-------------------|---------|----------|
| Health Device Profile | HDP | Generic medical device data (older; less common in new builds) |
| Blood Pressure | BLP / BPS | BP cuffs |
| Glucose | GLP / GLS | Glucometers |
| Body Composition | BCS | Smart scales |
| Heart Rate | HRP / HRS | HR monitors, chest straps |
| Pulse Oximeter | PLXS | SpO2 |
| Continuous Glucose Monitor | CGMS | CGMs (some) |
| Weight Scale | WSS | Smart scales |

Engineering pattern:

1. Discover the device, validate the advertised service UUIDs.
2. Pair / bond — capture the device address and bond key.
3. Subscribe to notifications/indications on the relevant characteristics.
4. Parse the standardized payloads (per Bluetooth SIG specifications).
5. Map values + units to FHIR `Observation` with `Device` and `DeviceMetric` as appropriate.

Pitfalls: vendor extensions / proprietary services, timestamp encoding (some use device-local with no zone), reading bookmarks for catch-up after disconnect.

---

## FHIR Modeling

Use these resources consistently:

- `Observation` — the reading. Always include `effectiveDateTime`, `code` (LOINC, verified), `valueQuantity` with UCUM unit, `device` reference, and `subject`.
- `Device` — the source device. Include manufacturer, model, UDI when available, identifier (serial), and a stable reference.
- `DeviceUseStatement` — patient's use of the device over a time window (good for "Patient X is using a Dexcom G7 from 2025-09-01 to present").
- `Provenance` — capture the chain (device → HealthKit → app → cloud → FHIR server) for clinician trust.

For BP, model as a single `Observation` with `component`s for systolic, diastolic, mean — not as separate observations.

---

## Data Quality and Artifacts

Wearable data is noisy. Filter or flag — do not silently accept.

- **Out-of-physiologic-range** values (HR = 0, glucose = 800 mg/dL, SpO2 = 50%).
- **Motion artifacts** on PPG-based HR/SpO2 during exercise.
- **Manual entries** vs. **sensor entries** — preserve the distinction (`Observation.method` or extension).
- **Backfilled data** — when a watch syncs after days offline; mark the receive vs. effective times.
- **Duplicate sources** — Apple Watch HR and Whoop HR for the same patient and the same minute; deduplicate or annotate.
- **Unit confusion** — kg vs. lb, mg/dL vs. mmol/L; never assume — read the unit and convert explicitly with UCUM.

---

## Consent and Revocation

- Every wearable source requires an explicit, granular consent from the patient.
- Persist consent as a FHIR `Consent` resource: scope, sources, data types, expiration.
- Provide a one-tap revocation that disconnects the source and stops further ingest.
- On revocation: continue to retain past data per the medical record retention rules — do not silently delete the chart history. Stop pulls, mark the source as revoked, log the event.

---

## BAA Gaps (Critical)

Most consumer wearable vendors **do not sign BAAs** with most customers. Implications:

- **Apple HealthKit** — Apple generally does not sign a BAA for HealthKit data on the user's device. Data on-device under the user's control is generally not a HIPAA-regulated disclosure from Apple. Once **your** app pulls it server-side, **your** app becomes the regulated handler.
- **Google Health Connect** — similar pattern; Health Connect lives on the user's device.
- **Fitbit, Garmin, Oura, Whoop, Withings** — typically no BAA for standard developer access. If you receive the data on behalf of a covered entity, you are the BA for the covered entity, not the wearable vendor's BA.
- **Dexcom and other regulated device vendors** — partner agreements that may include BAA-equivalent terms for clinical use cases. Verify per vendor.

Engineering implication: design so that the device vendor's cloud doesn't store PHI on your behalf. Pull the data, transform, and store in your own BAA-covered infrastructure. Avoid sending PHI back to the vendor's cloud beyond what the user already shares.

---

## Wellness vs. Medical-Grade in Practice

When mixing wearable data with clinical decisions, build hard rails:

- Display wearable data with **clear source labeling** ("Apple Watch — not for diagnosis").
- Don't auto-trigger clinical interventions from non-cleared signals; require a clinician review.
- For FDA-cleared signals (Apple Watch AFib notification, Dexcom CGM), document the cleared indication and label its display accordingly.
- See `fda-samd` for whether your downstream use of wearable data turns your software into a SaMD.

---

## Reference Architecture Sketch

1. Mobile app: HealthKit / Health Connect read with per-type permission
2. Vendor cloud connectors: OAuth + scheduled / webhook pulls
3. Direct BLE: standard GATT profile parsers
4. Normalization layer: vendor schema → FHIR `Observation` + `Device` + `Provenance`
5. Data-quality gate: range, dedup, unit, artifact flags
6. Storage: BAA-covered FHIR store
7. Display: per-source labeling, wellness-vs-medical badging
8. Consent + revocation registry

---

## Task-Specific Questions

1. Which platforms are in scope (Apple HealthKit / Health Records, Google Health Connect, Fitbit, Garmin, Oura, Whoop, Withings, Dexcom, Libre, Medtronic, direct BLE)?
2. Is this for wellness / lifestyle use, or are you using the data for clinical decisions — and if clinical, which signals are FDA-cleared for that use (Apple Watch ECG/AFib, Dexcom G-series, Libre 3, etc.)?
3. What is the FHIR mapping target — US Core Observation, IPS, a custom profile? Which terminology binding (LOINC for codes, UCUM for units) are you using?
4. Which vendor relationships still have BAA gaps, and how is your architecture compensating (no PHI to vendor cloud, redact on egress, store only in your BAA-covered FHIR store)?
5. What is the consent and revocation flow — granular per data type, per source, or all-or-nothing? Where does the patient grant and revoke (portal, mobile app, in-vendor)?
6. Are you doing direct BLE (which standard GATT profiles — Blood Pressure, Glucose, Heart Rate, Pulse Ox, Body Composition) or only vendor-cloud pulls?
7. How is provenance preserved end-to-end (vendor / HKSourceRevision / app → FHIR Provenance) so clinicians can see "BP came from Withings via HealthKit," not just "BP"?

---

## Related Skills

- **healthcare-context**: drives BAA inventory, mobile platforms, jurisdictions
- **remote-patient-monitoring**: when wearable data is part of a billable RPM program
- **fhir-integration**: `Observation`, `Device`, `Provenance` shape
- **smart-on-fhir**: Apple Health Records patient launch flow
- **patient-portal**: where the patient grants and revokes wearable consent
- **fda-samd**: when downstream use of wearable data triggers SaMD regulation
- **hipaa-compliance**: BAA gaps and how to compensate
