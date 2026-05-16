# RPM Device Integration Patterns

This reference covers the three connectivity architectures used in RPM (cellular hub, BLE-to-phone-to-cloud, direct LTE per device) and the data-ingestion pattern from device telemetry into a FHIR `Observation`. Choose the architecture that fits the patient, not the engineering team.

## Architecture 1 — Cellular hub

A small dedicated device (the **hub**) lives in the patient's home, has its own cellular SIM (LTE-M or NB-IoT), and aggregates one or more peripheral devices over BLE or proprietary RF. The hub uploads to the platform.

```
[BP cuff] --BLE--> [hub] --LTE-M--> [platform ingest]
[scale]   --BLE--> [hub]
[pulse ox]--BLE--> [hub]
```

### Strengths

- No phone, no Wi-Fi, no app required of the patient.
- Best fit for elderly / low-tech populations.
- One support burden (the hub) for the whole device set.
- Hub can buffer readings during connectivity loss.

### Weaknesses

- Highest per-patient hardware cost (hub + each peripheral).
- Carrier coverage matters; LTE-M / NB-IoT dead zones exist.
- Logistics: hubs must be shipped, returned, retired, retired-from-billing.
- Single point of failure in the home — if the hub dies, everything stops.

### When to use

- Medicare FFS HTN / CHF / DM programs targeting older patients.
- Programs serving rural or low-broadband populations.
- Multi-device chronic care where a single point of telemetry is operationally simpler.

## Architecture 2 — BLE-to-phone-to-cloud

The device pairs over BLE with the patient's smartphone; a mobile app receives the readings, formats them, and forwards to the platform.

```
[BP cuff] --BLE--> [patient phone app] --HTTPS--> [platform ingest]
```

### Strengths

- Lowest per-patient hardware cost (the patient supplies the phone).
- Richer in-app context — patient can see readings immediately, get coaching content.
- Easier to roll out updates (push a new app version).
- Lower returns / logistics burden.

### Weaknesses

- Requires a working smartphone and an awake / foregrounded app for many BLE stacks.
- iOS and Android background BLE behavior differs and changes across OS versions.
- App kill, OS upgrade, BLE pairing UX failure → data loss.
- Phone replacement is a re-pairing event the workflow must handle.

### When to use

- Programs targeting younger / tech-comfortable patients.
- Programs with a strong companion app (coaching, education, telehealth).
- Programs that can tolerate occasional data gaps from app issues.

## Architecture 3 — Direct LTE per device

Each peripheral device has its own cellular SIM and uploads directly. No hub, no phone.

```
[BP cuff with LTE-M] --LTE-M--> [platform ingest]
[scale with LTE-M]   --LTE-M--> [platform ingest]
```

### Strengths

- No phone, no hub.
- Per-device isolation — one device failure does not stop others.
- Devices can be shipped pre-provisioned and "just work" out of the box.

### Weaknesses

- Highest per-unit hardware cost (cellular radio + SIM per device).
- Per-device carrier coverage and per-SIM carrier dependency.
- Devices are larger / heavier; battery life often shorter.

### When to use

- Single-device programs (one BP cuff per patient) where the all-in-one experience is worth the unit cost.
- Programs in geographies with reliable LTE-M / NB-IoT coverage.

## Architecture 4 — Wi-Fi

The device joins the patient's home Wi-Fi. Lower hardware cost than cellular but **provisioning is the hardest UX problem in RPM**. Wi-Fi has largely fallen out of favor for chronic-care RPM compared to cellular and BLE-to-phone, though it persists in some niche scale and BP devices.

## Hybrid

Many programs combine architectures. A common pattern: cellular hub for the primary chronic condition + BLE-to-phone for an optional wearable. Each path is its own ingestion pipeline.

## Architecture comparison

| Dimension | Cellular hub | BLE-to-phone | Direct LTE per device | Wi-Fi |
|-----------|--------------|--------------|------------------------|-------|
| Patient effort | Low | Medium | Low | Medium-high (provisioning) |
| Phone required | No | Yes | No | Sometimes (provisioning) |
| Per-unit cost | Medium-high | Low | High | Low |
| Resilience to phone issues | High | Low | High | Medium |
| Best for elderly / low-tech | Strong | Weak | Strong | Weak |
| Easy multi-device | Strong | Medium | Weak | Medium |
| Returns logistics | Hub-centric | Device-centric | Device-centric | Device-centric |

## Data ingestion to FHIR Observation

Regardless of architecture, the normalized data shape should be FHIR `Observation` with linked `Device` and `DeviceUseStatement` resources, written into a BAA-covered FHIR store.

### Observation skeleton

```
Observation
  subject -> Patient
  device -> Device (manufacturer, model, serial, UDI)
  effectiveDateTime -> reading timestamp at the device (preserve, do not overwrite)
  code -> LOINC (look up current LOINC per parameter — do not guess)
  valueQuantity -> value + UCUM unit
  bodySite -> coded body location where relevant
  method -> coded method (e.g., automatic vs. manual cuff)
  component[] -> for paired values (BP systolic + diastolic + MAP)
  identifier[] -> device-side reading identifier for idempotency
```

### Field-by-field guidance

- **`effectiveDateTime`** — the time on the device. Preserve with full timezone offset. Do not normalize to UTC and lose the offset.
- **Server receive time** — store separately (extension, audit field, or `Observation.issued`). Used to detect backfill and clock-drift.
- **`code`** — look up the current LOINC code. Do not invent or guess. Common BP code: `85354-9` (Blood pressure panel) with components for systolic and diastolic. Verify against the LOINC release used by your downstream consumers.
- **`valueQuantity.unit`** — always include UCUM. `mm[Hg]` for BP, `mg/dL` for glucose, `kg` or `[lb_av]` for weight. Never assume — convert explicitly.
- **`component`** — for BP, use `Observation.component` to nest systolic and diastolic. Do not emit separate Observations for each.
- **`identifier`** — include a device-side ID so a duplicated POST does not double-count.
- **`device`** — reference a `Device` resource that captures manufacturer, model, serial, UDI, and version. `DeviceUseStatement` captures the patient's use of the device over a time window.
- **`subject`** — the `Patient`. Verify the device-to-patient pairing was attested and is current.

### Batching

- **Cellular and direct LTE** devices typically upload each reading immediately.
- **BLE-to-phone** devices may batch when the app comes to the foreground after offline time.
- Use FHIR `Bundle` (`type: transaction` or `batch`) for multi-reading uploads.
- Define a maximum acceptable latency per parameter — for CGM, near-real-time; for HTN, an hour is usually fine.

### Idempotency

- Include `Observation.identifier` with a device-side unique key.
- The ingest endpoint should treat a duplicate `identifier` as a no-op (or as an upsert with confirmation of identical content).
- Network retries and device retries are normal — never double-count from them.

### Timestamps and clock drift

- Detect: device timestamps in the future relative to server time, large reverse jumps, identical timestamps for many readings.
- Flag on ingest and surface to operations.
- Do not silently rewrite the timestamp — preserve the original and emit a quality flag.

### Quality flags

Common ingest-time quality flags worth capturing as extensions or `Observation.dataAbsentReason`:

- Out-of-physiologic-range (HR = 0, SpO2 = 50%, glucose = 800 mg/dL)
- Motion artifact (PPG-based HR/SpO2 during clear motion)
- Manual entry vs. sensor (use `Observation.method` or extension)
- Backfilled data (effective time precedes receive time by more than a defined threshold)

### Device lifecycle

- `Device` resource per physical device (UDI, serial, manufacturer, model).
- `DeviceUseStatement` per patient-device pairing window (status, period, indication).
- On device replacement, transfer history via Patient + Device linkage, not by reusing the device serial.

## Audit and write-back

- Emit a FHIR `AuditEvent` for every ingestion event, with the source (which architecture, which device, which app version), patient, and ingested resource references.
- Write back to the EHR via FHIR `Observation` (push) or HL7 v2 `ORU^R01` for legacy EHRs.
- For billing-evidence purposes, also write to a per-patient-per-month rollup that supports the CPT-code audit trail.

## Verify-current sources

- LOINC release notes (typically June and December)
- UCUM specification (current version)
- US Core IG `Vital Signs` and `Observation` profiles
- IEEE 11073 nomenclature for device data (where applicable)
- Manufacturer integration guides (current API / firmware version per device)
- HL7 SMART on FHIR / Bulk Data IGs for higher-volume ingestion
