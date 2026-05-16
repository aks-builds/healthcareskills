---
name: hl7-v2
description: When the user wants to design, parse, generate, or troubleshoot HL7 v2.x pipe-delimited messages. Use when the user mentions "HL7 v2," "HL7 2.x," "ADT," "ORM," "ORU," "MDM," "SIU," "DFT," "MSH," "PID," "OBX," "OBR," "ACK," "NAK," "MLLP," "Mirth," "NextGen Connect Integration Engine," "Rhapsody," "Cloverleaf," "Iguana," "Corepoint," "integration engine," "z-segment," or specific trigger events like "A01," "A08," "R01," "O01," "T02." For modern REST/JSON exchange, see fhir-integration. For DICOM imaging messages, see dicom-imaging. For CDA clinical documents, see cda-ccda.
metadata:
  version: 1.0.0
---

# HL7 v2 Messaging

You are an expert in HL7 v2.x messaging — the pipe-delimited standard that still moves the vast majority of clinical traffic between EHRs, labs, pharmacies, registration systems, and ancillary apps. Your goal is to help engineers and integration analysts design interfaces, write integration-engine channels, and debug malformed messages without inventing field positions or trigger events that don't exist in the version they're targeting.

## Initial Assessment

Check `.agents/healthcare-context.md` (fallback: `.claude/healthcare-context.md`) before answering. Focus on:

- **HL7 v2 version**: 2.3.1, 2.5.1, 2.6, 2.7, 2.8 — segments, fields, and component layouts change.
- **EHR and ancillary vendors**: Epic, Oracle Health (Cerner), Meditech, lab/pharmacy systems each have idiosyncratic profiles and z-segments.
- **Integration engine**: Mirth/NextGen Connect, Rhapsody, Cloverleaf, Iguana, Corepoint, custom — affects how channels, scripts, and routing are described.
- **Connectivity**: MLLP/TCP, sFTP files, VPN, leased line.

If the file is missing, ask only what is needed (version, sending/receiving system, message types) and offer to save it.

---

## Message Structure

HL7 v2 is a flat, line-delimited text format. Each line is a **segment** beginning with a three-letter code, followed by fields.

```
MSH|^~\&|SENDING_APP|SENDING_FAC|RECEIVING_APP|RECEIVING_FAC|20260101120000||ADT^A01^ADT_A01|MSGCTRL00001|P|2.5.1
EVN|A01|20260101120000
PID|1||MRN-123456^^^HOSPITAL^MR||DOE^JANE^A||19850203|F|||123 MAIN ST^^METROPOLIS^IL^60601^USA
PV1|1|I|2W^201^A^MAIN^^^^^^^^|EL||||1234567^SMITH^JOHN^^^DR
```

### Delimiters (declared in MSH-1 and MSH-2)

| Char | Default | Purpose |
|------|---------|---------|
| `|` | field separator (MSH-1) | Separates fields |
| `^` | component separator | Separates components within a field |
| `&` | subcomponent separator | Separates subcomponents within a component |
| `~` | repetition separator | Separates repetitions of a field |
| `\` | escape character | Escapes reserved characters in data |

Empty fields are simply `||`. Re-encoding without preserving delimiters declared in MSH is a classic source of corruption.

### Segment / Field / Component / Subcomponent / Repetition

- Segment: a single line (`PID|...`).
- Field: separated by `|`.
- Component: separated by `^`.
- Subcomponent: separated by `&`.
- Repetition: separated by `~` (e.g., multiple addresses or identifiers in a single field).

### MSH Header (the only segment with field-0 implicit)

MSH-1 *is* the field separator, so PID-1 is "Set ID," but MSH-1 is the `|` character itself. MSH-9 carries the message type (`ADT^A01^ADT_A01`), MSH-10 the message control ID, MSH-11 the processing ID (`P` production, `T` test, `D` debug), and MSH-12 the version.

---

## Common Message Types and Trigger Events

### ADT — Admission, Discharge, Transfer

| Event | Meaning |
|-------|---------|
| A01 | Admit/visit notification |
| A02 | Transfer a patient |
| A03 | Discharge/end visit |
| A04 | Register a patient (outpatient) |
| A05 | Pre-admit a patient |
| A08 | Update patient information |
| A11 | Cancel admit/visit |
| A13 | Cancel discharge/end visit |
| A28 | Add person information (MPI add) |
| A31 | Update person information (MPI update) |
| A40 | Merge patient — patient identifier list |
| A60 | Update allergy information |

### Orders and Results

| Type | Trigger | Use |
|------|---------|-----|
| ORM | O01 | General order (deprecated in 2.5+ for new lab/pharmacy, but still common) |
| OMG | O19 | General clinical order |
| OML | O21 | Lab order (replaces ORM for labs in 2.5+) |
| OMI | O23 | Imaging order |
| OMP | O09 | Pharmacy/treatment order |
| ORU | R01 | Unsolicited observation result (labs, vitals streaming) |
| OUL | R21 | Unsolicited lab observation (specimen-oriented, 2.5+) |
| ORG | O20 | General clinical order response |

### Documents, Scheduling, Financial

| Type | Trigger | Use |
|------|---------|-----|
| MDM | T02 | Original document notification + content |
| MDM | T08 | Document edit notification + content |
| SIU | S12 / S14 / S15 / S26 | Schedule new appt / modification / cancellation / no-show |
| BAR | P01 / P02 / P05 | Add / purge / update patient accounts |
| DFT | P03 / P11 | Post detail financial transaction (charges) |
| RDE | O11 | Pharmacy/treatment encoded order |
| RAS | O17 | Pharmacy/treatment administration |
| VXU | V04 | Unsolicited vaccination record update |

Always verify trigger events against the version's *Standard Message* chapters — vendors sometimes use trigger events outside the spec.

---

## ACK / NAK and Enhanced Acknowledgments

Receivers respond with an `ACK` message carrying an `MSA` (Message Acknowledgment) segment.

| MSA-1 | Meaning |
|-------|---------|
| AA | Application Accept |
| AE | Application Error |
| AR | Application Reject |
| CA / CE / CR | Commit-level (enhanced acknowledgment mode) |

Enhanced ACK mode (controlled by MSH-15/16 `AL`, `NE`, `SU`, `ER`) splits acknowledgment into a commit ACK (the message was received and parsed) and an application ACK (the application processed it). This is the recommended pattern for asynchronous interfaces.

```
MSH|^~\&|RECV_APP|RECV_FAC|SEND_APP|SEND_FAC|20260101120005||ACK^A01^ACK|ACK00001|P|2.5.1
MSA|AA|MSGCTRL00001
```

NAK (rejection) uses `AR` or `AE` with `ERR` providing structured error info (location, error code, severity).

---

## Z-Segments and Conformance Profiles

- **Z-segments** (segments beginning with `Z`) are vendor-defined extensions. They are valid HL7 but not portable across systems unless both ends agree.
- A **conformance profile** (HL7 v2 Conformance Profile / Static Definition) documents which segments/fields/values a specific interface supports — usage codes `R` (required), `O` (optional), `RE` (required but may be empty), `C` (conditional), `X` (not supported).
- Always exchange a written conformance profile (or vendor-published spec) before building.

---

## Encoding Pitfalls

The escape character `\` is used to encode reserved chars and to embed formatting:

| Escape | Meaning |
|--------|---------|
| `\F\` | Field separator literal |
| `\S\` | Component separator literal |
| `\T\` | Subcomponent separator literal |
| `\R\` | Repetition separator literal |
| `\E\` | Escape character literal |
| `\X##\` | Hex code (e.g., `\X0A\` = LF) |
| `\.br\` | Line break in formatted text |

Other pitfalls:

- Mixing line endings (`\r` vs. `\r\n` vs. `\n`). The spec requires `\r` (CR) as segment terminator; many tools forgive `\n`, others do not.
- UTF-8 vs. ASCII vs. ISO-8859-1 — declared in MSH-18 (`UNICODE UTF-8`).
- Truncation: a trailing `|` is significant (an explicit empty field). Trimming it can change cardinality.
- Date/time: format `YYYYMMDDHHMMSS[.SSSS][+/-ZZZZ]` — always include offset if local time differs from UTC.
- Identifier types in CX (e.g., PID-3): the `assigning authority` (third component) and `identifier type code` (fifth component) drive correlation.

---

## MLLP — Minimal Lower Layer Protocol

MLLP frames each message between control bytes over TCP:

```
<VT> message <FS><CR>
```

| Byte | Hex | Role |
|------|-----|------|
| VT | 0x0B | Start block |
| FS | 0x1C | End block |
| CR | 0x0D | Carriage return after end block |

MLLP is plain TCP; for security wrap it in TLS (MLLP-S) or run over an IPsec VPN. Most integration engines speak MLLP natively.

---

## Integration Engines

| Engine | Notes |
|--------|-------|
| Mirth Connect / NextGen Connect Integration Engine | Open-core, JavaScript-based transformers, Channels with Source/Destination connectors |
| Rhapsody (Lyniate) | Workflow-based, IDE-driven mapping |
| Cloverleaf (Infor) | TCL-based, long-standing enterprise option |
| Iguana (iNTERFACEWARE) | Lua-based, lightweight |
| Corepoint (Lyniate) | GUI-driven, action lists, popular in mid-market US hospitals |

When recommending solutions, match the engine the customer already runs — re-tooling integration engines is rarely worth it.

---

## End-to-End Example: ADT^A08 (Patient Update)

```
MSH|^~\&|EPIC|HOSPITAL_A|LAB_SYS|HOSPITAL_A|20260301091500||ADT^A08^ADT_A01|MSG00042|P|2.5.1
EVN|A08|20260301091500|||USER123^SMITH^JOHN
PID|1||MRN-123456^^^HOSPITAL_A^MR~SSN-XXX-XX-1234^^^USA^SS||DOE^JANE^A||19850203|F||2106-3^White^HL70005|123 MAIN ST^^METROPOLIS^IL^60601^USA||(555)555-1212^PRN^PH||EN^English^HL70296|M^Married^HL70002
PD1|||CLINIC_NORTH^^12345
NK1|1|DOE^JOHN|SPO^Spouse|123 MAIN ST^^METROPOLIS^IL^60601|(555)555-1234
PV1|1|O|CLINIC_NORTH^EXAM2||||9999^WELBY^MARCUS^^^DR|||MED|||||||SELF
```

Receiver responds:

```
MSH|^~\&|LAB_SYS|HOSPITAL_A|EPIC|HOSPITAL_A|20260301091501||ACK^A08^ACK|ACK00042|P|2.5.1
MSA|AA|MSG00042
```

## Example: ORU^R01 (Lab Result)

```
MSH|^~\&|LIS|LAB_A|EHR|HOSPITAL_A|20260301100000||ORU^R01^ORU_R01|MSG00099|P|2.5.1
PID|1||MRN-123456^^^HOSPITAL_A^MR||DOE^JANE^A||19850203|F
OBR|1|ORDER-77^EHR|ACC-555^LIS|85025^CBC W/ AUTO DIFF^CPT|||20260301085500|||||||||9999^WELBY^MARCUS||||||20260301095500|||F
OBX|1|NM|6690-2^Leukocytes^LN||7.4|10*3/uL^^UCUM|4.5-11.0|N|||F
OBX|2|NM|789-8^Erythrocytes^LN||4.6|10*6/uL^^UCUM|4.1-5.6|N|||F
OBX|3|NM|718-7^Hemoglobin^LN||13.9|g/dL^^UCUM|12.0-16.0|N|||F
```

Note: codes shown (LOINC, CPT) are illustrative — always look up current codes from the source of truth before using in production.

---

## Common Pitfalls

- Building parsers that split on `|` without first reading MSH-1 to confirm the actual separator.
- Treating PID-3 as a single MRN — it is a **repeating** field of CX identifiers; the right MRN is keyed on assigning authority and ID type.
- Ignoring trailing empty fields when comparing messages — they are semantically significant.
- Assuming z-segments are stable across upgrades.
- Failing to negotiate enhanced ACK behavior before going live — silent message loss is otherwise easy.
- Not handling A40 (merge) correctly: downstream systems must reconcile MRNs and re-link historical data.

---

## Task-Specific Questions

1. Which HL7 v2 version is in play — 2.3.1, 2.5.1, 2.6, 2.7, or 2.8? (Segment definitions, field cardinality, and trigger events differ.)
2. Which message types and trigger events are in scope (ADT A01/A03/A04/A08/A11/A28/A40, ORM/OML, ORU R01, MDM T02/T08, SIU, DFT, VXU, etc.)?
3. What integration engine is running the channel — Mirth/NextGen Connect, Rhapsody, Cloverleaf, Iguana, Corepoint, or a custom parser?
4. What is the transport — MLLP over TCP (and is it wrapped in TLS or VPN), sFTP file-drop, HTTPS POST, or something else?
5. Who are the sending and receiving systems (EHR vendor, lab/pharmacy/imaging system) and is there a published conformance profile or vendor spec to follow?
6. Are there z-segments, vendor extensions, or local code tables in use that we need to account for or document?
7. Is enhanced acknowledgment (MSH-15/16) negotiated, and what is the expected ACK/NAK behavior?

---

## Related Skills

- **fhir-integration** — modern REST/JSON alternative; many programs run v2 and FHIR in parallel
- **ehr-integration** — vendor-specific channels (Epic Bridges, Cerner OPENLink, etc.) that ride on v2
- **terminology-services** — code system bindings for OBX, DG1, RXE, etc.
- **cda-ccda** — MDM^T02 messages typically transport C-CDA documents in OBX-5
- **ihe-profiles** — IHE PIX/PDQ/LAB profiles map directly to ADT and OUL message exchanges
- **audit-logging** — every received and sent v2 message should produce an audit record (ATNA-compatible)
- **hipaa-compliance** — MLLP transport security and minimum-necessary scoping
