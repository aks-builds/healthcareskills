# HL7 v2 Message Type Reference

Practical cheat sheet for the message types and trigger events you'll meet most often, with required and commonly-present segments. **Always verify against the specific HL7 v2 version chapter** — segment cardinality and field definitions shift across 2.3.1, 2.5.1, 2.6, 2.7, and 2.8.

## Contents

- ADT — Admission, Discharge, Transfer
- Orders (ORM, OMG, OML, OMI, OMP)
- Results (ORU, OUL)
- Documents (MDM)
- Scheduling (SIU)
- Financial (BAR, DFT)
- Pharmacy Administration (RDE, RAS)
- Vaccinations (VXU)
- Master Files (MFN)
- Acknowledgments (ACK)
- Common Segments Glossary
- Cross-Version Notes

---

## ADT — Admission, Discharge, Transfer

ADT messages carry registration, encounter, and demographic events. They're the highest-volume HL7 v2 traffic in most hospitals.

### Common Trigger Events

| Event | Meaning | Required segments |
|-------|---------|-------------------|
| A01 | Admit/visit notification | MSH, EVN, PID, PV1 |
| A02 | Transfer a patient | MSH, EVN, PID, PV1 |
| A03 | Discharge/end visit | MSH, EVN, PID, PV1 |
| A04 | Register a patient (outpatient) | MSH, EVN, PID, PV1 |
| A05 | Pre-admit a patient | MSH, EVN, PID, PV1 |
| A06 | Change outpatient to inpatient | MSH, EVN, PID, PV1 |
| A07 | Change inpatient to outpatient | MSH, EVN, PID, PV1 |
| A08 | Update patient information | MSH, EVN, PID, PV1 |
| A11 | Cancel admit/visit | MSH, EVN, PID, PV1 |
| A12 | Cancel transfer | MSH, EVN, PID, PV1 |
| A13 | Cancel discharge/end visit | MSH, EVN, PID, PV1 |
| A28 | Add person information (MPI add) | MSH, EVN, PID |
| A29 | Delete person information | MSH, EVN, PID |
| A31 | Update person information (MPI update) | MSH, EVN, PID |
| A40 | Merge patient — identifier list | MSH, EVN, PID, MRG |
| A47 | Change patient identifier list | MSH, EVN, PID, MRG |
| A60 | Update allergy information | MSH, EVN, PID, IAM |

### Common Optional Segments

- PD1 — patient additional demographics (PCP, language, etc.)
- NK1 — next of kin / emergency contact
- ROL — roles (attending, referring provider)
- IN1 / IN2 / IN3 — insurance
- GT1 — guarantor
- AL1 / IAM — allergies (AL1 deprecated in favor of IAM in newer versions)
- DG1 — diagnosis on the encounter
- PR1 — procedures
- OBX — observations attached to encounter
- ACC — accident information

### Key Rules

- A40 (merge) MUST include MRG with the surviving and merged identifiers.
- A08 (update) is a full replacement of demographics — downstream systems should replace, not merge.
- A28 / A31 distinguish person events from encounter events; some sites collapse them into A04 / A08.

---

## Orders

| Type | Trigger | Use |
|------|---------|-----|
| ORM | O01 | General order (deprecated in 2.5+ for new lab/pharmacy; still common in legacy) |
| OMG | O19 | General clinical order |
| OML | O21 | Lab order (replaces ORM for labs in 2.5+) |
| OMI | O23 | Imaging order |
| OMP | O09 | Pharmacy/treatment order |
| ORG / ORL / ORI / ORP | O20 / O22 / O24 / O10 | General / lab / imaging / pharmacy order response |

### Common Segments

| Segment | Role |
|---------|------|
| MSH | Header |
| PID | Patient identifier and demographics |
| PV1 | Visit |
| ORC | Common order — order control code (NW, OK, CA, DC, etc.), placer/filler order numbers, order status |
| OBR | Observation request — universal service ID, specimen, collector, dates |
| OBX | Observation/result OR ask-at-order-entry questions on orders |
| NTE | Note |
| SPM | Specimen (2.5+) |
| TQ1 / TQ2 | Timing/Quantity (replaces deprecated TQ in 2.5+) |
| DG1 | Diagnosis associated with order |
| AL1 / IAM | Allergy info |

### ORC Order Control Codes (subset)

| Code | Meaning |
|------|---------|
| NW | New order |
| OK | Order accepted |
| UA | Unable to accept |
| CA | Cancel order request |
| CR | Cancelled as requested |
| DC | Discontinue order request |
| OD | Order discontinued |
| RP | Order replace request |
| RU | Replaced unsolicited |
| RO | Replacement order |
| HD | Hold order request |
| RL | Release previous hold |

Always confirm the OC table in the target version.

---

## Results

| Type | Trigger | Use |
|------|---------|-----|
| ORU | R01 | Unsolicited observation result (labs, vitals streaming, radiology) |
| OUL | R21 | Unsolicited lab observation (specimen-oriented, 2.5+) |
| ORU | R30 | Unsolicited point-of-care observation |

### Common Segments

| Segment | Role |
|---------|------|
| MSH | Header |
| PID | Patient |
| PV1 | Visit (optional but common) |
| ORC | Common order (when result has originating order) |
| OBR | Observation request / panel header |
| OBX | Individual result |
| NTE | Note |
| SPM | Specimen |

### OBX Field Highlights

| Field | Purpose |
|-------|---------|
| OBX-2 | Value type (NM, ST, TX, FT, CE, CWE, CD, DT, TS, ED, RP) |
| OBX-3 | Observation identifier (LOINC code + system) |
| OBX-5 | Observation value |
| OBX-6 | Units (UCUM) |
| OBX-7 | References range |
| OBX-8 | Abnormal flags (N, L, H, LL, HH, A, AA, etc.) |
| OBX-11 | Observation result status (F = final, C = corrected, P = preliminary, X = cancelled) |
| OBX-14 | Date/time of observation |
| OBX-19 | Date/time of analysis |

---

## Documents

| Type | Trigger | Use |
|------|---------|-----|
| MDM | T01 | Original document notification (metadata only) |
| MDM | T02 | Original document notification + content |
| MDM | T04 | Document edit notification (metadata only) |
| MDM | T08 | Document edit notification + content |
| MDM | T11 | Document cancel notification |

### Common Segments

- MSH, EVN, PID, PV1, TXA (document type, status, identifiers, author)
- OBX (carries content — typically base64-encoded ED for C-CDA payloads)

C-CDA is most often delivered as ED (encapsulated data) in OBX-5 of an MDM^T02.

---

## Scheduling (SIU)

| Trigger | Event |
|---------|-------|
| S12 | Notification of new appointment booking |
| S13 | Notification of appointment rescheduling |
| S14 | Notification of appointment modification |
| S15 | Notification of appointment cancellation |
| S17 | Notification of appointment deletion |
| S26 | Patient did not show up / no-show |

Common segments: MSH, SCH (schedule), PID, PV1, RGS (resource group), AIG/AIL/AIP (general resource / location / personnel), NTE.

---

## Financial

### BAR (Add/Change Billing Account)

| Trigger | Event |
|---------|-------|
| P01 | Add patient account |
| P02 | Purge patient account |
| P05 | Update patient account |
| P12 | Update diagnosis/procedure (DRG-related) |

### DFT (Detail Financial Transactions)

| Trigger | Event |
|---------|-------|
| P03 | Post charge / payment |
| P11 | Post adjustment / detail transaction |

Common segments: MSH, EVN, PID, PV1, FT1 (financial transaction), DG1 (diagnosis), PR1 (procedure), GT1 (guarantor), IN1/IN2/IN3 (insurance).

---

## Pharmacy Administration

| Type | Trigger | Use |
|------|---------|-----|
| RDE | O11 | Pharmacy/treatment encoded order |
| RDS | O13 | Pharmacy/treatment dispense |
| RGV | O15 | Pharmacy/treatment give |
| RAS | O17 | Pharmacy/treatment administration |
| RRE | O12 | Pharmacy/treatment encoded order acknowledgment |

Segments: ORC, RXE (encoded order), RXR (route), RXC (component), RXA (administration), RXD (dispense), RXG (give), TQ1/TQ2.

---

## Vaccinations

| Type | Trigger | Use |
|------|---------|-----|
| VXU | V04 | Unsolicited vaccination record update |
| VXQ | V01 | Query for vaccination record (deprecated; use QBP) |
| VXR | V03 | Vaccination query response (deprecated) |

Segments: MSH, PID, PD1, NK1, ORC, RXA (the actual administration), RXR (route), OBX, NTE.

CVX codes go in RXA-5. MVX codes (manufacturer) go in RXA-17.

---

## Master Files (MFN)

For exchanging master-file data (charge masters, lab codes, etc.).

| Trigger | Event |
|---------|-------|
| M02 | Master file - staff |
| M03 | Master file - test/observation |
| M04 | Master file - charge description master |
| M05 | Master file - patient location |
| M06 | Master file - clinical study |
| M07 | Master file - clinical study schedule |
| M08–M11 | Test/observation, observation batteries, etc. |

Segments: MSH, MFI (file ID), MFE (entry), plus type-specific segments.

---

## Acknowledgments

ACK^XXX where XXX is the original trigger event (or `ACK` plain in older interfaces).

| Segment | Role |
|---------|------|
| MSH | Header (reverse sender/receiver) |
| MSA | AA (accept) / AE (app error) / AR (app reject); CA/CE/CR for commit-level in enhanced mode |
| ERR | Detailed error info (location, error code, severity) |

Enhanced ACK (MSH-15/16 = AL/NE/SU/ER) splits acknowledgment into a commit ACK (received and parsed) and an application ACK (processed). Recommended for asynchronous interfaces.

---

## Common Segments Glossary

| Segment | Carries |
|---------|---------|
| MSH | Message header — delimiters, sender/receiver, message type, control ID, version, processing ID |
| EVN | Event type — trigger, datetime, operator, facility |
| PID | Patient identifier and demographics |
| PD1 | Patient additional demographics |
| PV1 | Patient visit — class, location, attending |
| PV2 | Patient visit additional |
| NK1 | Next of kin |
| GT1 | Guarantor |
| IN1 / IN2 / IN3 | Insurance |
| AL1 / IAM | Allergy (AL1 deprecated) |
| DG1 | Diagnosis |
| PR1 | Procedure |
| ORC | Common order |
| OBR | Observation request |
| OBX | Observation/result |
| NTE | Note |
| SPM | Specimen |
| TQ1 / TQ2 | Timing/quantity |
| SCH | Schedule |
| AIG / AIL / AIP | Appointment resource segments |
| TXA | Transcription/document header |
| FT1 | Financial transaction |
| RXE / RXR / RXC / RXA / RXD / RXG | Pharmacy segments |
| MFI / MFE | Master file segments |
| MRG | Merge identifiers (for A40, A47) |
| MSA / ERR | Acknowledgment segments |
| ZXX | Z-segments (vendor-defined) |

---

## Cross-Version Notes

| Change | Versions |
|--------|----------|
| OML / OMG / OMP introduced; ORM still allowed but deprecated for new use | 2.5+ |
| OUL introduced for specimen-oriented results | 2.5+ |
| TQ deprecated in favor of TQ1 / TQ2 | 2.5+ |
| AL1 deprecated in favor of IAM | 2.5+ |
| Many CE fields replaced with CWE (Coded with Exceptions) | 2.5+ |
| New extended demographic segments (PD1) | 2.4+ |
| HL7 v2.8 reorganized message structures, added segments | 2.8 |

Z-segments and vendor-specific quirks are common across all versions — always exchange a written conformance profile and an IHE Integration Statement (where applicable) before going live.
