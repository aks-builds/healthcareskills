# HL7 Integration Engine Comparison

Brief reference for the integration engines most commonly seen in US hospitals and provider organizations. **Verify current product names, licensing terms, and feature claims with each vendor before making procurement decisions** — these change.

## Contents

- What an Integration Engine Does
- Engines at a Glance
- Mirth Connect / NextGen Connect Integration Engine
- Rhapsody
- Cloverleaf
- Iguana
- Corepoint
- Cloud-Native Options
- Choosing an Engine
- Migration Realities
- Common Anti-Patterns

---

## What an Integration Engine Does

An integration engine is the middleware that:

- Accepts inbound HL7 v2 (MLLP, sFTP, HTTP, file drop), DICOM, FHIR, and other messages.
- Validates, parses, and transforms them.
- Routes to one or more destinations.
- Generates ACK/NAK responses.
- Logs every message and provides a replay/error-queue UI.
- Supports retries, backoffs, and recovery on restart.
- Often handles minor terminology mapping inline.

Hospitals typically run one (sometimes two) engines that mediate hundreds of channels. Re-tooling an engine is a multi-year project — match the recommendation to the engine in place.

---

## Engines at a Glance

| Engine | Vendor | Scripting | Licensing model | Common deployments |
|--------|--------|-----------|-----------------|--------------------|
| Mirth Connect / NextGen Connect IE | NextGen Healthcare | JavaScript (Rhino / Nashorn / GraalVM by version) | Open-core (free OSS Connect; paid commercial NextGen Connect IE) | Wide; mid-market and labs, dev/test |
| Rhapsody | Lyniate (Rhapsody Health Solutions) | Workflow-based; JavaScript and IDE | Commercial | Large health systems |
| Cloverleaf | Infor | TCL (Tool Command Language) | Commercial | Long-standing enterprise hospitals |
| Iguana | iNTERFACEWARE | Lua | Commercial | Mid-market, labs, smaller systems |
| Corepoint | Lyniate (Corepoint Health) | GUI action lists; some scripting | Commercial | Mid-market US hospitals |
| HealthShare Health Connect / Ensemble | InterSystems | ObjectScript / Python / Java | Commercial | Customers already on InterSystems IRIS / Caché |
| BizTalk Server (HL7 Accelerator) | Microsoft | C# / pipelines | Commercial | Microsoft shops; declining footprint |
| Apache Camel + HAPI HL7 / Spring Integration | Open source | Java / Groovy / Kotlin | OSS | Modern custom builds |

Verify current product lines and rebrandings before recommending.

---

## Mirth Connect / NextGen Connect Integration Engine

- **Lineage**: originated as the open-source Mirth Connect; commercial product is NextGen Connect Integration Engine.
- **Strengths**: low cost of entry; very large practitioner community; flexible JavaScript transformers; many built-in connectors; works on Linux/Windows.
- **Weaknesses**: open-source version has limited enterprise features (clustering, audit, SLA support); some operational pain at high message volumes without paid edition; multiple JavaScript engine generations.
- **Scripting**: JavaScript in source/destination/preprocessor/postprocessor scripts; access to Java classes via Rhino bridge.
- **Channels**: source connector (LLP, FTP, HTTP, JMS, JDBC, etc.) → filters → transformer → destination connector(s).
- **Code repository**: ships with templates / code-template libraries.
- **Mark "verify" on**: current edition feature matrix, supported JS engine, licensing tiers, vendor support SLAs.

---

## Rhapsody

- **Vendor**: Lyniate (Rhapsody Health Solutions).
- **Strengths**: visual workflow IDE; strong message debugging; mature in large IDNs; FHIR support; clustering.
- **Weaknesses**: licensing cost; workflow IDE has a learning curve.
- **Scripting**: JavaScript filters; rich expression language; built-in HL7, FHIR, X12, NCPDP libraries.
- **Common use**: enterprise hospital systems, labs, payers.
- **Mark "verify" on**: current version, license model (per-message, per-thread, per-core), cloud-hosted vs. on-prem.

---

## Cloverleaf

- **Vendor**: Infor (Cloverleaf Integration Suite).
- **Strengths**: rock-solid at very high message volumes; long-standing presence in academic medical centers; rich operations tooling.
- **Weaknesses**: TCL scripting has a steeper learning curve for modern developers; some legacy UI/UX.
- **Scripting**: TCL (Tool Command Language); custom TCL libraries common in shops.
- **Common use**: large academic and integrated delivery networks.
- **Mark "verify" on**: current Infor product packaging, modernization roadmap.

---

## Iguana

- **Vendor**: iNTERFACEWARE.
- **Strengths**: Lua scripting is approachable; lightweight footprint; good developer ergonomics; FHIR support.
- **Weaknesses**: smaller community than Mirth; not as common in very large IDNs.
- **Scripting**: Lua, with iNTERFACEWARE's `node` and `mapper` libraries.
- **Common use**: mid-market hospitals, labs, payers; vendor integrations.
- **Mark "verify" on**: current version, cloud hosting, licensing.

---

## Corepoint

- **Vendor**: Lyniate (formerly Corepoint Health).
- **Strengths**: GUI-driven action lists make HL7 transformation accessible to non-developers; strong vendor support; commonly seen in mid-market US hospitals.
- **Weaknesses**: GUI-only modeling can become unwieldy for very complex transforms; licensing cost.
- **Scripting**: limited custom scripting; relies on action-list configuration.
- **Common use**: community hospitals, mid-size IDNs.
- **Mark "verify" on**: current pricing, cloud options, Lyniate product overlap with Rhapsody.

---

## Cloud-Native Options

For greenfield builds, several cloud platforms now offer managed health-data ingest:

| Platform | Capability |
|----------|-----------|
| Azure Health Data Services (FHIR + DICOM + IoT Connector) | FHIR-native, HL7 v2 conversion via templates |
| AWS HealthLake | FHIR datastore; HL7 v2 ingest typically via custom Lambda or partner |
| Google Cloud Healthcare API | FHIR, HL7 v2 (via store + parser), DICOM stores |
| Redox | Hosted integration as a service; HL7 v2 in, JSON out |
| Health Gorilla | Aggregation network and API |
| Mirth Cloud (NextGen) | Managed Mirth Connect |

These offer faster time-to-value for net-new builds, but rarely displace an on-prem engine in an existing hospital.

---

## Choosing an Engine

Match the engine to:

- **What you already run** — re-tooling an integration engine is a multi-year effort. The right answer is almost always the engine your team knows.
- **Vendor support coverage** — your EHR vendor may certify against specific engines.
- **Message volume** — Cloverleaf, Rhapsody, and HealthShare have proven high-volume track records.
- **Developer skill set** — JavaScript (Mirth), Lua (Iguana), TCL (Cloverleaf), workflow (Rhapsody), GUI (Corepoint).
- **Cloud strategy** — managed offerings reduce ops load but lock in to a cloud.
- **Budget** — Mirth (open source) is the cheapest start; commercial engines scale better operationally.

---

## Migration Realities

- Channel rewrites take longer than you think — 1–3 days per non-trivial channel is typical.
- Parallel-run for at least one full business cycle (one week minimum, ideally a month).
- Build automated regression tests on canonical message corpora before migrating any channel.
- Validate that ACK/NAK behavior is byte-equivalent to the old engine.
- Audit logs, error queues, and replay UIs must be in place before flipping production traffic.

---

## Common Anti-Patterns

- Adopting a new integration engine for one new interface, ending up with two engines running in production.
- Writing parsing code in the engine that should live in a library (re-implementing CX parsing in every channel).
- Ignoring the engine's built-in error queue and writing custom logging to a database that nobody monitors.
- Trusting the engine's "auto-mapping" for HL7→FHIR — these are starting points, not production-ready transforms.
- Not version-controlling channel code (most engines have export/import for this).
- Failing to monitor MLLP TCP connections; silent drops are a classic ops headache.
- Putting terminology mapping inline in transformer code rather than via an external terminology service.
