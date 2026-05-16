# Imaging AI Orchestrators

Reference for the orchestrator / marketplace layer that sits between PACS/RIS and AI models. Verify all product names, capabilities, and FDA-clearance specifics against current vendor documentation and the FDA databases — orchestrators rebrand, acquire competitors, and change feature sets frequently.

## Contents

- Why an Orchestrator
- Categories
- Vendor Profiles (verify)
- Capability Checklist When Evaluating
- Integration Patterns
- Common Pitfalls

---

## Why an Orchestrator

A radiology AI deployment without an orchestrator means each model gets its own DICOM router, license server, results-return pipeline, audit trail, monitoring agent, and conversation with the PACS administrator. With three models on three modalities, that scales badly.

An orchestrator centralizes:

- **DICOM routing**: receive study from PACS / modality / VNA; decide which model(s) apply; route the study.
- **Containerized model execution**: standard runtime (often Docker) with a defined input/output contract.
- **Results return**: SR, SEG, secondary capture, FHIR, or HL7 v2 ORU back to PACS / RIS / EHR.
- **License and entitlement management**: which models are licensed at which sites for which use cases.
- **Audit and logging**: who ran what model on which study, when, with what version.
- **Marketplace**: catalog of available models, often with FDA-cleared / CE-marked attestations (verify per model).
- **Monitoring**: latency, failure rate, throughput; sometimes performance metrics with feedback hooks.

The institution integrates the **orchestrator** once with PACS / RIS / EHR. Each new model is then a configuration change, not a new integration project.

---

## Categories

Three rough categories. The boundaries blur; products move between categories over time.

### EHR / Voice-Aligned

Orchestrators tied closely to a major reporting / voice-recognition vendor's ecosystem. Tight integration with dictation, structured reporting, and the radiologist's daily workflow.

### PACS / Imaging-Vendor-Aligned

Orchestrators offered by major PACS / imaging-vendor portfolios. Strong native integration with that vendor's PACS, viewer, and modality fleet; varying support for third-party PACS.

### Vendor-Neutral / Independent

Orchestrators that aim to work across PACS vendors, often emphasizing multi-vendor model marketplaces and on-prem deployment.

---

## Vendor Profiles (verify)

The following profiles describe widely-discussed orchestrators. **Verify current product names, ownership, capabilities, FDA clearance specifics, and integration support against vendor documentation before any committed decision.**

### Nuance Precision Imaging Network (PIN)

- **Category**: EHR / voice-aligned (Microsoft / Nuance).
- **Notes (verify)**: marketplace of AI models with integration into Nuance's reporting workflow (PowerScribe). Wide modality coverage in published claims. Cloud-centric architecture; verify on-prem options for institutions with cloud constraints.
- **Strengths (verify)**: depth of reporting-workflow integration; reach across US radiology departments via PowerScribe footprint.
- **Watch**: confirm current model catalog and FDA status of each individual model — orchestrator presence does not imply clearance of any specific model.

### Blackford Platform

- **Category**: vendor-neutral / independent (Bayer-owned at time of writing — verify).
- **Notes (verify)**: orchestrator with multi-PACS support; broad marketplace of third-party AI applications.
- **Strengths (verify)**: PACS-vendor neutrality; on-prem deployment options.
- **Watch**: ownership changes; confirm current marketplace scope.

### Bayer Calantic

- **Category**: vendor-neutral with Bayer-portfolio alignment (verify current relationship to Blackford).
- **Notes (verify)**: digital platform for radiology AI, positioned for multi-vendor deployments. Bayer has historically emphasized contrast-adjacent and imaging-adjacent AI.
- **Strengths (verify)**: integration with Bayer's imaging informatics portfolio.
- **Watch**: confirm current positioning relative to Blackford.

### Sirona Medical / DeepHealth

- **Category**: a newer-generation cloud-native radiology platform (RIS / PACS / orchestrator hybrid).
- **Notes (verify)**: emphasis on a unified radiologist workstation including AI; DeepHealth is part of RadNet (verify current product lineup and any rebrands).
- **Strengths (verify)**: cloud-native architecture, native integration of AI with reading.
- **Watch**: confirm scope (full platform vs. orchestrator only) and PACS-vendor neutrality.

### GE HealthCare Edison

- **Category**: PACS / imaging-vendor-aligned (GE).
- **Notes (verify)**: GE's AI platform spanning modalities and adjacent applications; depth on GE-imaging fleet.
- **Strengths (verify)**: native integration with GE modalities and Centricity / Edison True PACS.
- **Watch**: third-party PACS support depth.

### Philips ISP / HealthSuite Imaging

- **Category**: PACS / imaging-vendor-aligned (Philips).
- **Notes (verify)**: Philips's advanced visualization and AI platform; IntelliSpace Portal historical; current product lineup branding shifts to HealthSuite Imaging in some markets — verify.
- **Strengths (verify)**: integration with Philips PACS / advanced visualization.
- **Watch**: confirm current product naming and on-prem vs. cloud posture.

### Others to Confirm in Current Market

The following names appear regularly in market scans; capabilities and even existence shift fast — verify:

- Aidoc (originally a clinical AI vendor; has expanded into orchestration / marketplace plays — verify).
- Rad AI (predominantly reporting / workflow AI — verify orchestration role).
- Enlitic / TeraRecon / Intelerad / Sectra-adjacent platforms (each has evolving AI orchestration story — verify).
- Aidence / Quibim / Cassiopeia / others positioned as AI platforms or vendor-specific orchestrators — verify.

**Do not** quote a specific product's claim list without confirming against the vendor's current public documentation.

---

## Capability Checklist When Evaluating

Use this to compare orchestrators on what actually matters in a deployment:

### Inbound Routing

- DICOMweb (QIDO/WADO/STOW)? Classic DIMSE (C-STORE, C-FIND, C-MOVE)?
- HL7 v2 ORM / OMI inbound for order context?
- FHIR `ImagingStudy` / `ServiceRequest` inbound?
- Conditional routing (modality, body part, age, study description) with rule editor?

### Model Runtime

- Container standard (Docker, OCI)?
- GPU support (model and version)?
- Co-located inference (low latency) vs. cloud inference (egress, BAA)?
- Model versioning and rollback?

### Outbound Returning

- DICOM SR with TID 1500 measurement report?
- DICOM SEG?
- DICOM Secondary Capture / Encapsulated PDF?
- HL7 v2 ORU^R01 to RIS / EHR?
- FHIR `DiagnosticReport` + `Observation` + `RiskAssessment`?
- Worklist re-prioritization (RIS / PACS-specific integration)?

### Governance and Audit

- Per-model per-study audit log?
- Algorithm identification on every output (AlgorithmName + AlgorithmVersion)?
- Output retention and PHI scope (does the orchestrator store images? results? both?)?
- Access controls aligned with institutional roles?

### Regulatory

- FDA-cleared / CE-marked at the platform level vs. per-model?
- Support for PCCP-driven model updates?
- Documentation suitable for SaMD post-market reporting?

### Monitoring

- Throughput, latency, failure-rate dashboards?
- Per-model performance metrics where ground truth feedback is collected?
- Drift detection (input or output)?
- Alerting integration (PagerDuty, ServiceNow, SIEM)?

### Commercial

- Pricing model (per-study, per-model, platform fee)?
- Marketplace revenue share (relevant if you are deploying your own model)?
- BAA scope across every sub-service used?

---

## Integration Patterns

### Pattern A: Orchestrator-on-Prem with Cloud Catalog

The orchestrator runs inside the institution's network; the catalog and license management talk to a cloud control plane. Studies stay on prem; metadata about model invocations egresses.

- Pro: lowest egress, lowest latency to PACS.
- Con: hardware footprint to manage.

### Pattern B: Cloud Orchestrator with PACS Connector

A small connector inside the institution routes studies to the cloud orchestrator; results come back. Requires a BAA with the orchestrator vendor and an egress path scoped to imaging traffic.

- Pro: minimal on-prem footprint.
- Con: full-study egress; throughput-dependent latency; BAA scoping must include every sub-service.

### Pattern C: Embedded in the PACS / Advanced Visualization

For vendor-aligned orchestrators (GE Edison, Philips), AI runs within the same platform that hosts advanced visualization. Tight integration; comes with the lock-in of that vendor's PACS.

### Pattern D: Direct Modality Integration

Some AI runs at the modality (CT scanner, MR scanner). Bypasses PACS for latency-critical use cases. Requires modality-vendor cooperation. Audit and update mechanisms are non-trivial.

---

## Common Pitfalls

- **Conflating "FDA-cleared orchestrator" with "FDA-cleared model"**. Orchestrators may have a clearance for routing / display; the individual model's clearance is separate and must be verified per model.
- **Underestimating PACS integration work**. Even with a vendor-neutral orchestrator, hanging protocols, DICOM tag preservation, and worklist updates are still site-specific.
- **Not negotiating BAA scope across sub-services**. Cloud orchestrators use storage, compute, monitoring, ML platforms — each must be in scope.
- **Missing the dictation / reporting integration**. A model whose output never reaches the dictation system is half-deployed. Confirm SR pre-population works in PowerScribe / Fluency / structured-reporting tooling.
- **Not testing rollback**. Update the model in staging; can the orchestrator roll back to the prior version in under an hour if the live model behaves badly?
- **Skipping monitoring on day one**. Day-one drift, day-one failures, and day-one equity gaps all happen. The dashboards and feedback loops should exist before go-live, not after.
