# CGM Vendor API Comparison

This reference compares the API surfaces of the three major US continuous glucose monitor (CGM) vendors: **Dexcom**, **Abbott** (FreeStyle Libre / LibreView / LibreLinkUp), and **Medtronic** (CareLink). CGM data is medical-device-grade in most clearances; partner agreements and labeled-use terms govern what you can do with the data.

**Verify all of the following with the vendor's current partner documentation before designing or shipping. APIs, partner programs, model coverage, and labeled-use scope all change.**

## High-level comparison

| Dimension | Dexcom | Abbott (Libre) | Medtronic (CareLink) |
|-----------|--------|----------------|----------------------|
| Primary US consumer CGM | Dexcom G6 / G7 / Stelo (consumer) | FreeStyle Libre 2 / 3 / Libre 3 Plus / Lingo (consumer) | Guardian / Simplera (paired with pump) |
| API type | REST + OAuth 2.0 | REST + OAuth 2.0 (partner / clinical access) | Partner / direct integration |
| Real-time data availability | Available via partner agreement | Limited real-time partner access; LibreView is clinician-facing | Pump + CGM data via partner |
| Clinical use access | Separate clinical / partner program | Separate clinical / partner program | Partner access typically required |
| Consumer share path | Dexcom Share, Dexcom Clarity | LibreView / LibreLinkUp | CareLink Personal / CareLink Connect |

## Dexcom

### Products

- **Dexcom G6** — earlier generation, widely deployed.
- **Dexcom G7** — current generation as of recent releases.
- **Dexcom Stelo** — over-the-counter CGM for non-insulin-using adults. OTC clearance is a distinct regulatory pathway. Verify current.

### API surfaces

- **Dexcom API (developer.dexcom.com)** — OAuth 2.0; standard endpoints for EGVs (Estimated Glucose Values), events, alerts, calibrations, and device information. Historically intended for research and non-clinical use; clinical use requires a separate agreement.
- **Real-time data** — historically required a separate partner program with stricter terms.
- **Dexcom Clarity** — clinician-facing dashboard with its own access path.
- **Dexcom Share** — patient share path; not a developer API per se.

### Considerations

- The Dexcom API has historically had **rate limits** and a sandbox vs. production environment distinction.
- Real-time data has a lag of a few minutes from the device.
- Labeled use of the signal you pull matters — research labeling is different from a clinically-deployed feature.

## Abbott (FreeStyle Libre / LibreView / LibreLinkUp / Lingo)

### Products

- **FreeStyle Libre 2** — earlier generation; flash CGM with optional alerts.
- **FreeStyle Libre 3** and **Libre 3 Plus** — real-time continuous reading, smaller form factor, Bluetooth.
- **Lingo** — consumer wellness CGM; targeted at metabolic-health / wellness use; distinct from medical clearance.
- **LibreView** — clinician-facing dashboard.
- **LibreLinkUp** — caregiver / follower app.

### API surfaces

- **LibreView API** — partner / clinical access for clinics, devices, glucose data; HIPAA-eligible paths under partner agreement.
- **Abbott partner programs** — separate developer / partner channels for clinical and research integrations.

### Considerations

- The consumer apps (FreeStyle LibreLink, LibreLinkUp) do not have a public developer API; clinical / partner access is the path for cloud integration.
- US versus EU API surfaces can differ — verify per region.
- Lingo is wellness-labeled; do not use Lingo data for clinical decisions.

## Medtronic (CareLink)

### Products

- **Guardian Sensor** family — CGM paired with insulin pumps.
- **MiniMed pumps** — closed-loop / hybrid closed-loop integration with the CGM.
- **Simplera** — newer CGM in Medtronic's lineup.
- **CareLink Personal / CareLink Connect** — patient and family apps.
- **CareLink for Professionals** — clinician dashboard.

### API surfaces

- **Partner API access** — typically required and limited. Medtronic has been more closed historically than Dexcom or Abbott for third-party API access.
- The pump + CGM closed-loop context means more device-state data is in scope (basal rates, boluses, low-glucose-suspend events) than CGM-only systems.

### Considerations

- Partner program enrollment can be lengthy.
- Closed-loop algorithm state is proprietary; integrators typically receive CGM and event data but not algorithm internals.

## Shared engineering considerations

### Labeled use vs. signal pulled

- Each CGM has a **labeled indication** under its FDA clearance. Using the signal outside that indication can put both the patient and your software in regulatory difficulty.
- For example, a CGM cleared for use in adults with diabetes is not, on its own, cleared for use in a different population or for a different clinical purpose. Confirm.

### Partner agreements

- Real-time CGM data, in particular, generally requires a partner agreement with terms covering: use case, data residency, security, BAA where applicable, audit, breach notification, and labeled use.
- Plan for partner agreement timelines (months, not days) when scoping.

### BAA

- Vendors with clinical / partner programs may offer BAA-equivalent terms. Verify the scope.
- For consumer-app-only API access, BAAs are typically not offered — design so PHI is not sent to the vendor cloud beyond what the user already shares.

### Latency

- CGM-to-cloud latency varies. Real-time alerts for clinical decision support need to know the latency characteristics — a five-minute lag is usually fine for trend display, not fine for time-critical alerting.

### Data shape

Model CGM data as FHIR `Observation` with:

- `code` → LOINC glucose code (verify; CGM may have a specific LOINC distinct from BGM)
- `valueQuantity` → value + UCUM (`mg/dL` or `mmol/L` — always read the unit)
- `effectiveDateTime` → device-reported timestamp
- `device` → `Device` with manufacturer, model, sensor lot if available
- `component[]` → trend arrow direction, rate-of-change if exposed
- `category` → `vital-signs` or a CGM-specific category if the IG provides one

### Provenance and source labeling

- Capture the vendor (Dexcom / Abbott / Medtronic), the specific product (G7, Libre 3, Guardian), and the API path (partner real-time vs. retrospective).
- Display source in clinician views — "Dexcom G7 via partner API" is more useful than "glucose 123."

## Pump and closed-loop data (Medtronic + others)

For pump-integrated CGMs:

- Basal / bolus / suspend / corrective events are part of the dataset.
- Model as `MedicationAdministration` or a pump-specific profile if your IG provides one.
- Closed-loop algorithm state is typically proprietary and not exposed.

## Consumer / wellness CGM caveat

- Stelo (Dexcom) and Lingo (Abbott) are positioned for non-medical / wellness or OTC contexts.
- Do not use wellness-labeled data for clinical decision support. The labeling and the engineering controls differ.
- For RPM billing under CMS 99453/99454, a wellness-labeled CGM may not satisfy the medical-device requirement. Verify clearance status of the specific product.

## Verify-current sources

- Each vendor's developer / partner portal (Dexcom developer site, Abbott / LibreView clinical, Medtronic developer programs)
- Each vendor's current FDA-cleared indications and labeling
- Your contracting team for current partner agreements and BAA terms
- LOINC release notes for glucose / CGM codes
- HL7 implementation guides for CGM / glucose monitoring where available
