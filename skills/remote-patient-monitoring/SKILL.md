---
name: remote-patient-monitoring
description: When the user wants to design, build, or bill a remote patient monitoring (RPM) program. Also use when the user mentions "RPM," "remote patient monitoring," "RTM," "remote therapeutic monitoring," "CPT 99453," "99454," "99457," "99458," "99091," "98975," "98976," "98977," "98980," "98981," "16-day rule," "BP cuff integration," "glucometer integration," "CGM," "pulse oximeter," "weight scale," "ECG patch," "smart inhaler," "peak flow," "device telemetry," "FHIR Observation ingest," "cellular device," "BLE device," or "patient device pairing." For consumer wearables (Apple/Google/Fitbit/Garmin/Oura/Whoop) see wearables-integration. For video visits paired with RPM see telehealth-platform.
metadata:
  version: 1.0.0
---

# Remote Patient Monitoring

You are an expert in remote patient monitoring (RPM) and remote therapeutic monitoring (RTM) systems. Your goal is to help engineers ship RPM that is clinically actionable, audit-ready for CMS reimbursement, and resilient against real-world device, network, and patient-behavior issues.

## Initial Assessment

Read `.agents/healthcare-context.md` first (fall back to `.claude/healthcare-context.md`). Use it to determine:

- Whether the user is a provider organization billing RPM, a digital health company building the platform, or a device manufacturer.
- Conditions being monitored and patient population.
- EHR and identity stack (for write-back and patient pairing).
- Cloud and BAA posture.

If missing, ask: who bills, what conditions, which devices, single-site or multi-site.

---

## Device Categories

| Category | Typical use | Notes |
|----------|-------------|-------|
| Blood pressure cuff (upper arm, wrist) | HTN, pregnancy, CHF | Validated arm cuffs preferred; wrist devices less accurate |
| Glucometer (BGM) | DM1, DM2, gestational | Fingerstick; per-reading data |
| Continuous glucose monitor (CGM) | DM1, DM2 (intensive) | Dexcom, Abbott Libre, Medtronic — see also `wearables-integration` |
| Weight scale | CHF, weight management, eating disorders | Cellular or BLE |
| Pulse oximeter | COPD, post-COVID, OSA, sleep | Spot vs. continuous |
| ECG patch / handheld | Arrhythmia detection | Zio, BodyGuardian, KardiaMobile |
| Peak flow / spirometer | Asthma, COPD | Effort-dependent; data-quality flags matter |
| Smart inhaler / cap | Asthma, COPD adherence | Adherence + technique signals |
| Thermometer (smart) | Pediatric infection, post-op | Episodic |
| Activity / sleep / HR | Cardiovascular, behavioral | Usually wearable; see `wearables-integration` |

For each device, capture **clearance status** (FDA-cleared medical device vs. general wellness), accuracy spec, sampling rate, and battery model.

---

## FDA Cleared vs. Wellness

- A device used to **diagnose, treat, monitor, or mitigate** disease is generally a medical device. Without 510(k)/De Novo/PMA clearance, you cannot bill RPM CPT codes that require a medical device.
- A device marketed only for "general wellness" (e.g., consumer step counters) is **not** appropriate for the RPM CPT codes that require a medical device.
- **CPT 99453/99454** specifically require a device defined as a medical device under section 201(h) of the FFDCA. Confirm device clearance and labeled use match the billed code.
- For RTM (98975+), the device must be a medical device that monitors a non-physiological **therapeutic** parameter (e.g., respiratory, musculoskeletal, medication adherence) — see RTM section.

Always verify device 510(k) number, predicate, and labeled indication before integrating it into a billable workflow.

---

## Connectivity Architectures

### Cellular (LTE-M / NB-IoT)

- Device has its own SIM; uploads readings directly to the platform.
- Pros: no app, no Wi-Fi, no phone — best for elderly / low-tech patients.
- Cons: hardware cost per unit; coverage dead zones; carrier dependency.

### BLE-to-app-to-cloud

- Device pairs with the patient's smartphone via BLE; app forwards to cloud.
- Pros: lower hardware cost; richer in-app context.
- Cons: requires a working phone, OS upgrades, BLE pairing UX issues; data loss when app is killed.

### Wi-Fi

- Device joins home Wi-Fi.
- Pros: low cost, no SIM.
- Cons: provisioning is the hardest UX problem in RPM.

### Hybrid / hub

- Patient has a small gateway (BLE+cellular) that aggregates devices.
- Common for multi-device chronic care programs.

Architectural rule: pick the connectivity that fits the patient, not the engineering team.

---

## Data Ingestion (FHIR-First)

Model device readings as FHIR `Observation` resources, with `Device` and `DeviceUseStatement` references.

- `Observation.subject` → `Patient`
- `Observation.device` → `Device` (manufacturer, model, serial, UDI)
- `Observation.effectiveDateTime` → reading timestamp at the device (preserve, do not overwrite with server receive time)
- `Observation.code` → LOINC (look up the current LOINC code for each parameter — do not guess)
- `Observation.valueQuantity` → value + UCUM unit
- `Observation.bodySite`, `Observation.method`, `Observation.component` (for BP — systolic + diastolic + MAP)

Batching:

- Most cellular devices upload every reading immediately.
- BLE-to-app devices may batch — define a maximum acceptable latency (e.g., 1 hour for HTN, near-real-time for CGM).
- Use FHIR `Bundle` (`type: transaction` or `batch`) for multi-reading uploads.
- Idempotency: include a device-side identifier in `Observation.identifier` so a duplicated POST does not double-count.

Timestamp discipline:

- Always record device-local time AND server receive time.
- Be explicit about timezone — store offsets, not naive timestamps.
- Detect clock drift (device timestamps in the future, large gaps).

---

## Patient Identification + Device Pairing

The most common source of bad data is **the wrong patient on the device**.

- Pair device-to-patient at enrollment with an attested human step (label the device, scan a QR code from the patient's chart).
- For shared-household scenarios (two parents both with HTN), give each patient a distinct, labeled device.
- Detect pairing anomalies (sudden 20-cm height change, weight jumping by 50 lb between readings, two patients reading simultaneously on the same SIM).
- On device replacement, transfer history via the Patient + Device link, not the device serial.

---

## Alerting and Escalation

Define alerts in clinical terms, not engineering terms.

- **Threshold alerts**: parameter outside a per-patient threshold (e.g., SBP > 180 or < 90).
- **Trend alerts**: sustained drift (e.g., 3 consecutive AM weights up 2 lb).
- **Non-adherence alerts**: missing readings vs. expected cadence.
- **Data-quality alerts**: artifact, sensor disconnect, cuff error.

Escalation rules:

1. Define **who** reviews each alert type (RN, MA, pharmacist, physician).
2. Define **SLA** for review (e.g., 1 business hour for critical, 24 hours for routine).
3. Define **action options** — call patient, message via portal, schedule visit, dispatch EMS for life-threatening.
4. Document the **disposition** of every alert. CMS audits look for this.

Avoid alert fatigue: tune thresholds per patient and per condition; track alert→action conversion rate; mute alerts the clinician has already addressed.

---

## Clinical Staff Workflow

- Surface a panel-level dashboard (all RPM patients sorted by risk / non-adherence / new alerts).
- For each patient: a 1-screen view with current device, latest reading, trend, medication, last contact.
- **Time tracking** for billable codes is critical (see Reimbursement below). Capture clinician time per patient per calendar month with a clear start/stop affordance and a free-text justification.
- Generate documentation that maps cleanly to the encounter note.

---

## Reimbursement (US — CMS, **verify current values**)

CMS publishes annual updates to RPM and RTM codes. Always check the current Medicare Physician Fee Schedule and your payer mix. Do **not** rely on the descriptions below for billing — they are summaries to anchor design, not coding guidance.

### RPM (physiologic) — common codes (verify each year)

- **99453** — initial set-up and patient education on device use (one-time per episode of care).
- **99454** — supply of device(s) with daily recordings or programmed alert transmissions; per 30 days. Requires a defined number of days of readings in the period — historically 16+ days (the **16-day rule**) within 30 days. Verify current.
- **99457** — first 20 minutes of treatment management services per calendar month with interactive communication.
- **99458** — each additional 20 minutes of treatment management services per calendar month.
- **99091** — physician/QHP collection and interpretation of data, per 30 days. Generally not billed concurrently with 99457/99458; check current rule.

### RTM (Remote Therapeutic Monitoring) — verify each year

RTM codes (commonly written as 989x0 — verify the precise current numbers, e.g., 98975 / 98976 / 98977 / 98980 / 98981) cover therapeutic monitoring (respiratory, musculoskeletal, CBT adherence, medication adherence). Differences from RPM include the type of data (non-physiologic in some cases) and the eligible billing practitioners.

### The 16-day rule

Historically, 99454 requires **at least 16 days of readings within a rolling 30-day window**. If the patient transmits fewer than 16 days, you cannot bill 99454 for that period. Engineering implications:

- Track days-with-at-least-one-reading per rolling 30-day window per patient.
- Surface to the clinical team which patients are at risk of failing the threshold so they can intervene before the end of the period.
- Do not silently fabricate days from device heartbeats — readings must be actual measured values, not pings.

### Eligible conditions

CMS does not enumerate conditions — RPM is billable when medically necessary. Common conditions: HTN, DM, CHF, COPD, post-discharge, pregnancy, weight management, OSA. Document medical necessity in the note.

### Eligible providers

- **99453, 99454** — generally billed by the practitioner (physician/QHP) supervising the program; auxiliary personnel can deliver under general supervision (verify current).
- **99457, 99458** — billed by the practitioner; auxiliary personnel may furnish under general supervision (verify current).
- **RTM** — historically a different set of eligible billers including PTs, OTs, SLPs, and clinical psychologists; verify each year.

---

## Audit and Documentation

CMS RPM audits commonly review:

- Patient consent to RPM (separate consent, often verbal documented in the chart).
- Initial set-up documentation (99453).
- Device-medical-device status.
- 16-day reading evidence per 30-day period.
- Time logs with start/stop and substance of activity (99457/99458).
- Order from the billing practitioner.
- Documentation that the device was provided to the patient and the patient was educated.
- Live interactive communication with the patient at least once in the month.

Engineering: emit an auditable trail that maps each billable code to its supporting evidence. The audit defense is the evidence trail, not the dashboard view.

---

## Common Audit Findings

- Billing 99454 without 16+ days of readings.
- Missing initial education documentation (99453).
- Using a consumer wellness device that is not a 510(k)-cleared medical device.
- Time logs that don't show interactive communication.
- Billing 99457 and 99091 in the same period when the rules don't allow it (verify current).
- Same calendar-month 99457/99458 from multiple practitioners.
- Device readings auto-generated when the patient was clearly not using the device.

---

## Reference Architecture Sketch

1. Enrollment + consent capture (FHIR `Consent`)
2. Device fulfillment + pairing
3. Device telemetry ingest (cellular endpoint or BLE-app SDK)
4. Normalization to FHIR `Observation` + `Device`
5. Rules engine (thresholds, trend, non-adherence)
6. Alert queue + clinician disposition
7. Time tracking + documentation
8. Write-back to EHR (FHIR or HL7 v2 ORU)
9. Billing data export (claim-ready summary per patient per month)
10. Audit log (`AuditEvent`)

---

## Task-Specific Questions

1. Which device categories are in scope (BP cuff, BGM, CGM, weight scale, pulse ox, ECG patch, smart inhaler, peak flow)? FDA-cleared medical devices or wellness-grade?
2. What is the connectivity architecture — cellular per device, BLE-to-app, Wi-Fi, or hub/gateway? Is there a target patient demographic (elderly, low-tech) that constrains the choice?
3. Who is doing clinical monitoring — in-house RN/MA team, third-party monitoring service, or hybrid? What are the staffing hours and SLA per alert tier?
4. Which CPT codes are you targeting (RPM 99453/99454/99457/99458/99091 and/or RTM 989xx)? Have you verified the current values, descriptors, and supervision rules for the payer mix?
5. What are the patient eligibility criteria — conditions, payer (Medicare FFS, MA, Medicaid, commercial), enrollment workflow, consent capture?
6. How are alerts routed — by severity, by patient panel, by clinician role? What is the documented escalation tree (call patient, secure message, schedule visit, 911)?
7. Is there an EHR write-back requirement (FHIR Observation, HL7 v2 ORU^R01) and what is the billing data export shape per month per patient?

---

## Related Skills

- **healthcare-context**: drives population, billing role, jurisdictions
- **wearables-integration**: consumer wearable telemetry vs. RPM-cleared devices
- **fhir-integration**: `Observation`, `Device`, `DeviceUseStatement` modeling
- **hl7-v2**: ORU^R01 result write-back to legacy EHRs
- **telehealth-platform**: pairing video visits with RPM data view
- **billing-claims**: claim generation and 837P shape
- **fda-samd**: when the RPM platform itself enters SaMD territory (alerts that drive treatment)
- **hipaa-compliance**: BAA with device manufacturer + cellular carrier
