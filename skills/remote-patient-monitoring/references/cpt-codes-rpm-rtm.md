# CPT Codes for RPM and RTM

This reference summarizes the CPT code families used for Remote Patient Monitoring (RPM) and Remote Therapeutic Monitoring (RTM). **CMS updates the descriptors, payment rules, supervision requirements, and concurrency restrictions for these codes every year in the Medicare Physician Fee Schedule (PFS) Final Rule. Verify current values, descriptors, and payment rules from CMS and your payer mix each year before relying on this content for billing or product design.**

This is engineering-design context only. Always coordinate with your billing / compliance team and your coders before billing.

## RPM (Physiologic) — the 99xxx family

RPM CPT codes cover **physiologic** parameters (BP, weight, glucose, SpO2, HR, etc.). The patient uses a device that meets the definition of a **medical device** under FFDCA § 201(h) — generally a 510(k)-cleared device.

### 99453 — Initial setup and patient education

- One-time per **episode of care**.
- Covers: setting up the device, educating the patient on its use, and the initial onboarding.
- Engineering evidence: enrollment date, device shipped / fulfilled date, patient education session documented (often signed off by the staff member), device pairing event captured.

### 99454 — Device supply with daily recording / programmed alert transmissions, per 30 days

- Per **30-day** period.
- Requires a minimum number of days with at least one transmitted reading within the rolling 30-day period — historically the **16-day rule**: at least **16 days with readings out of 30**. **Verify current.**
- Engineering evidence: per-patient, per-rolling-30-day-window count of distinct days with at least one valid reading. Readings must be actual measured values, not device heartbeats or pings.

### 99457 — First 20 minutes of treatment management per calendar month

- Per **calendar month** (not rolling 30 days — different from 99454).
- Requires **interactive communication** with the patient or caregiver at least once during the month — typically a phone call, video visit, or other live exchange. Asynchronous messages alone have historically not satisfied the interactive-communication requirement. Verify current.
- 20 minutes of clinical staff / physician / QHP time, cumulative within the month.
- Engineering evidence: timer entries (start / stop / duration / clinician / patient) and at least one logged interactive-communication event in the month.

### 99458 — Each additional 20 minutes of treatment management per calendar month

- Adds on to 99457.
- Each additional 20-minute block, billed once per occurrence within the month.
- Engineering evidence: continued time accumulation past the first 20 minutes; same interactive-communication requirement applies for the overall month.

### 99091 — Collection and interpretation of physiologic data by physician / QHP, 30 minutes per 30 days

- Pre-dates the 99453-99458 family. **Concurrency with 99457 / 99458 has been a source of CMS scrutiny — verify the current rule each year**, as the alignment between 99091 and 99457/99458 has shifted.
- Engineering evidence: physician / QHP time spent reviewing and interpreting data, separate from time spent in interactive communication.

## RTM (Therapeutic) — the 98xxx family

RTM CPT codes cover **therapeutic** monitoring — respiratory, musculoskeletal, medication adherence, CBT adherence. The device must be a medical device, and the data may be **non-physiologic** (e.g., adherence signals).

The RTM family includes (verify current code numbers and descriptors each year — these have evolved since the introduction of the family):

- **98975** — RTM initial setup and patient education
- **98976** — RTM device supply for monitoring **respiratory** system, per 30 days
- **98977** — RTM device supply for monitoring **musculoskeletal** system, per 30 days
- **989xx** — RTM device supply for **CBT** / cognitive behavioral therapy adherence (verify current number)
- **989xx** — RTM device supply for **medication adherence** (verify current number)
- **98980** — First 20 minutes of RTM treatment management services per calendar month
- **98981** — Each additional 20 minutes of RTM treatment management per calendar month

Key RTM differences from RPM:

- RTM data may be non-physiologic (adherence, technique, patient-reported responses) where RPM is physiologic.
- The list of eligible billing practitioners for RTM has historically been broader and includes PTs, OTs, SLPs, and clinical psychologists in addition to physicians and QHPs — verify current.
- 16-day-equivalent rules apply for the device-supply codes; verify current days threshold and the rolling-window definition.

## Codes that often interact with RPM / RTM

| Code | Use |
|------|-----|
| **CCM** (99490, 99439, 99487, 99489, 99491) | Chronic Care Management — separately billable in many cases, with concurrent-billing rules per CMS. Verify current. |
| **PCM** (99424, 99425, 99426, 99427) | Principal Care Management — for a single chronic condition. Verify current. |
| **TCM** (99495, 99496) | Transitional Care Management — post-discharge. |
| **BHI** (99492, 99493, 99494, 99484) | Behavioral Health Integration / Collaborative Care. |
| **Cardiac monitoring** (93224-93237 and others) | Independent of RPM; often confused at design time. |

Concurrent billing of RPM and these adjacent care-management families has annual rules. Build a per-month per-patient "what was billed" view so coders can audit.

## Engineering implications checklist

- [ ] Per patient, per rolling 30-day window: distinct days with at least one transmitted reading. Readings must be measured values, not pings.
- [ ] Per patient, per calendar month: cumulative clinical time, with **start / stop / duration / clinician / patient / activity-substance** captured for every timer entry. CMS audits look at the substance of the time.
- [ ] Per patient, per calendar month: at least one logged interactive-communication event (phone, video, etc.) for 99457 / 99458 / 98980 / 98981 eligibility.
- [ ] Per patient: device clearance status (manufacturer, 510(k) number, labeled indication) and confirmation it meets FFDCA § 201(h).
- [ ] Per patient: enrollment date, initial-education documentation (for 99453 / 98975).
- [ ] Per patient: order from the billing practitioner.
- [ ] Per patient: separate consent to RPM / RTM (typically verbal documented in the chart — verify current).
- [ ] Per encounter: maintain the linkage from billable code → supporting evidence in the audit log.
- [ ] Annual review: re-validate codes, descriptors, supervision rules, and concurrency restrictions against current CMS PFS Final Rule.

## Common audit findings (don't repeat these)

- Billing 99454 without 16+ days of actual readings.
- Counting device heartbeats / connectivity pings as "readings."
- Missing 99453 setup and education documentation.
- Using a consumer wellness device that does not meet FFDCA § 201(h).
- Time logs without substance (just a timer, no description of what was done).
- Time logs without an interactive-communication event in the month.
- Billing 99457 and 99091 in the same period when current rules don't allow it.
- Two practitioners both billing 99457 / 99458 for the same patient in the same month.
- Billing RTM when the device is general wellness, not a medical device.

## Verify-current sources

- Medicare Physician Fee Schedule (PFS) Final Rule — published annually in late fall, effective Jan 1
- CMS FAQs on RPM and RTM
- AMA CPT Codebook — current year
- Your local MAC (Medicare Administrative Contractor) and commercial-payer policies
- Industry coding guidance from coding organizations
