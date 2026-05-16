# Healthcare Skills for AI Agents

A collection of AI agent skills focused on building healthcare software. Built for engineers, informaticists, security teams, and product builders who want AI coding agents to help with EHR integration, FHIR/HL7 interoperability, HIPAA compliance, clinical AI/ML, telehealth, revenue cycle, and more. Works with Claude Code, OpenAI Codex, Cursor, Windsurf, and any agent that supports the [Agent Skills spec](https://agentskills.io).

> **Important**: These skills are tools for *software engineering* in healthcare. They are not a substitute for clinical, legal, or regulatory advice, and they do not generate medical advice for patients. See [AGENTS.md](AGENTS.md) for safety rules.

## What are Skills?

Skills are markdown files that give AI agents specialized knowledge and workflows for specific tasks. When you add these to your project, your agent can recognize when you're working on a healthcare task and apply the right standards, frameworks, and safety practices.

## How Skills Work Together

Skills reference each other and build on shared context. The `healthcare-context` skill is the foundation — every other skill checks it first to understand your organization, patient population, regulatory jurisdictions, EHR systems, and security posture before doing anything.

```
                              ┌──────────────────────────────────────┐
                              │         healthcare-context           │
                              │    (read by all other skills first)  │
                              └──────────────────┬───────────────────┘
                                                 │
   ┌──────────────┬───────────────┬──────────────┼──────────────┬───────────────┬──────────────┐
   ▼              ▼               ▼              ▼              ▼               ▼              ▼
┌──────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐ ┌────────────┐
│ Clinical │ │  Interop   │ │ Compliance │ │  Digital   │ │ Revenue    │ │  Public &  │ │  Patient   │
│Informatics│ │            │ │ & Security │ │   Health   │ │   Cycle    │ │    Pop.    │ │ Experience │
├──────────┤ ├────────────┤ ├────────────┤ ├────────────┤ ├────────────┤ ├────────────┤ ├────────────┤
│ehr-integ │ │fhir-integ  │ │hipaa-comp  │ │telehealth  │ │medical-cod │ │public-hlth │ │health-cont │
│clin-dec  │ │hl7-v2      │ │phi-handle  │ │patient-prt │ │billing-clm │ │pop-health  │ │patient-eng │
│clin-doc  │ │dicom       │ │hc-cybersec │ │rpm         │ │prior-auth  │ │clinical-rsr│ │accessibty  │
│cpoe      │ │cda-ccda    │ │21-cfr-11   │ │wearables   │ │value-based │ │            │ │            │
│smart-fhir│ │ihe-profile │ │gdpr-health │ │clinical-ml │ │            │ │            │ │            │
│med-recon │ │terminology │ │hitrust-csf │ │fda-samd    │ │            │ │            │ │            │
│          │ │tefca-hie   │ │audit-log   │ │chatbots    │ │            │ │            │ │            │
└────┬─────┘ └─────┬──────┘ └─────┬──────┘ └─────┬──────┘ └─────┬──────┘ └─────┬──────┘ └─────┬──────┘
     │             │              │              │              │              │              │
     └─────────────┴──────┬───────┴──────────────┴──────────────┴──────────────┴──────────────┘
                          │
       Specialized skills cross-reference broadly:
         medical-imaging-ai ↔ dicom-imaging ↔ fda-samd ↔ clinical-ai-ml
         genomics-precision-medicine ↔ fhir-integration ↔ terminology-services
         health-data-lake ↔ fhir-integration ↔ population-health-analytics
```

See each skill's **Related Skills** section for the full dependency map.

## Available Skills

<!-- SKILLS:START -->
| Skill | Description |
|-------|-------------|
| [healthcare-context](skills/healthcare-context/) | Create or update their healthcare context document, or whenever any other healthcare skill needs to understand the organization… |
| [21-cfr-part-11](skills/21-cfr-part-11/) | Building, validating, or auditing a computer system that creates, modifies, maintains, archives, retrieves, or transmits… |
| [accessibility-healthcare](skills/accessibility-healthcare/) | Design, build, audit, or remediate accessibility for healthcare apps and patient-facing systems. |
| [audit-logging](skills/audit-logging/) | Designing, implementing, or reviewing audit logging for PHI access — including the HIPAA §164.312(b) audit controls requirement… |
| [billing-claims](skills/billing-claims/) | Design or build software that handles US healthcare claims — submission, adjudication, remittance, denials, or appeals. |
| [cda-ccda](skills/cda-ccda/) | Author, parse, validate, or transform Clinical Document Architecture (CDA) or Consolidated CDA (C-CDA) documents. |
| [clinical-ai-ml](skills/clinical-ai-ml/) | Build, evaluate, deploy, or monitor machine learning models for clinical or operational healthcare use cases. |
| [clinical-decision-support](skills/clinical-decision-support/) | Design, build, deploy, or evaluate a clinical decision support (CDS) tool. |
| [clinical-documentation](skills/clinical-documentation/) | Design clinical documentation workflows, templates, or note ingestion. |
| [clinical-research](skills/clinical-research/) | Design, build, or operate clinical research informatics tools and workflows. |
| [cpoe-orders](skills/cpoe-orders/) | Design, build, or integrate with computerized provider order entry (CPOE). |
| [dicom-imaging](skills/dicom-imaging/) | Design, integrate, or troubleshoot DICOM medical imaging systems. |
| [ehr-integration](skills/ehr-integration/) | Integrate with a specific EHR vendor's APIs, app program, or sandbox. |
| [fda-samd](skills/fda-samd/) | Determine if their software is FDA-regulated, plan a submission pathway, design under quality systems, or maintain a regulated… |
| [fhir-integration](skills/fhir-integration/) | Design, implement, debug, or review FHIR integrations. |
| [gdpr-health-data](skills/gdpr-health-data/) | Processing health, genetic, or biometric data of people in the EU/EEA, UK, or other GDPR-aligned jurisdictions, or designing… |
| [genomics-precision-medicine](skills/genomics-precision-medicine/) | Design, build, or integrate genomics and precision-medicine informatics. |
| [health-chatbots](skills/health-chatbots/) | Design, build, evaluate, or deploy a conversational AI / chatbot in a healthcare setting. |
| [health-content-writing](skills/health-content-writing/) | Write, edit, or audit patient-facing health content. |
| [health-data-lake](skills/health-data-lake/) | Design, build, or operate a clinical / healthcare data lake, lakehouse, or warehouse. |
| [healthcare-cybersecurity](skills/healthcare-cybersecurity/) | Design, assess, or harden a healthcare cybersecurity program — covering threat-aligned safeguards, medical device security… |
| [hipaa-compliance](skills/hipaa-compliance/) | Help applying the HIPAA Privacy, Security, or Breach Notification Rules to a healthcare product, workflow, vendor relationship… |
| [hitrust-csf](skills/hitrust-csf/) | Preparing for, scoping, scoring, or maintaining a HITRUST CSF assessment or certification. |
| [hl7-v2](skills/hl7-v2/) | Design, parse, generate, or troubleshoot HL7 v2.x pipe-delimited messages. |
| [ihe-profiles](skills/ihe-profiles/) | Design, deploy, or troubleshoot IHE (Integrating the Healthcare Enterprise) integration profiles. |
| [medical-coding](skills/medical-coding/) | Design or build software that touches medical coding — computer-assisted coding (CAC), autocoders, NLP for clinical coding… |
| [medical-imaging-ai](skills/medical-imaging-ai/) | Design, build, validate, deploy, or monitor AI/ML on medical images. |
| [medication-reconciliation](skills/medication-reconciliation/) | Design, build, or improve medication reconciliation workflows at transitions of care. |
| [patient-engagement](skills/patient-engagement/) | Design, build, or evaluate a patient engagement program. |
| [patient-portal](skills/patient-portal/) | Design or build a patient portal, patient mobile app, or patient-facing API. |
| [phi-handling](skills/phi-handling/) | Designing or reviewing the operational controls around PHI — how to de-identify, anonymize, pseudonymize, encrypt, mask… |
| [population-health-analytics](skills/population-health-analytics/) | Design, build, or evaluate a population health analytics platform. |
| [prior-authorization](skills/prior-authorization/) | Design, build, integrate, or automate prior authorization workflows. |
| [public-health-reporting](skills/public-health-reporting/) | Design, build, or troubleshoot reporting from clinical systems to public health authorities. |
| [remote-patient-monitoring](skills/remote-patient-monitoring/) | Design, build, or bill a remote patient monitoring (RPM) program. |
| [smart-on-fhir](skills/smart-on-fhir/) | Build a SMART on FHIR app, configure OAuth against an EHR's FHIR API, or implement Backend Services. |
| [tefca-hie](skills/tefca-hie/) | Plan, join, or build against TEFCA or other US health information exchange networks. |
| [telehealth-platform](skills/telehealth-platform/) | Design, build, or evaluate a telehealth platform. |
| [terminology-services](skills/terminology-services/) | Choose, map, validate, or serve healthcare code systems and value sets. |
| [value-based-care](skills/value-based-care/) | Design or build software for value-based care (VBC) — risk adjustment, quality measurement, ACO/REACH analytics, capitation… |
| [wearables-integration](skills/wearables-integration/) | Integrate consumer wearables or fitness devices into a clinical or health-app workflow. |
<!-- SKILLS:END -->

## Installation

### Option 1: Claude Code Plugin

```bash
/plugin marketplace add aks-builds/healthcareskills
/plugin install healthcare-skills
```

### Option 2: Clone and Copy

```bash
git clone https://github.com/aks-builds/healthcareskills.git
cp -r healthcareskills/skills/* .agents/skills/
```

### Option 3: Git Submodule

```bash
git submodule add https://github.com/aks-builds/healthcareskills.git .agents/healthcareskills
```

Then reference skills from `.agents/healthcareskills/skills/`.

## Usage

Once installed, just ask your agent to help with healthcare engineering tasks:

```
"Help me design a FHIR R4 integration for our patient portal"
→ Uses fhir-integration, patient-portal, smart-on-fhir

"Do a HIPAA risk analysis on this architecture"
→ Uses hipaa-compliance, healthcare-cybersecurity, audit-logging

"Set up an EDI 837 claims submission flow"
→ Uses billing-claims, medical-coding

"Build a CDS Hook that warns on drug-drug interactions"
→ Uses clinical-decision-support, medication-reconciliation, terminology-services
```

You can also invoke skills directly:

```
/fhir-integration
/hipaa-compliance
/ehr-integration
```

## Skill Categories

### Foundation
- `healthcare-context` — Org/patient/jurisdiction/EHR context file every other skill reads first

### Clinical Informatics & EHR
- `ehr-integration` — Epic, Cerner/Oracle, Athenahealth, Meditech
- `clinical-decision-support` — CDS Hooks, alerts, guidelines
- `clinical-documentation` — Structured notes, SOAP, ambient scribes
- `cpoe-orders` — Order entry, order sets, signatures
- `smart-on-fhir` — App launch, OAuth, OIDC, scopes
- `medication-reconciliation` — Med rec workflows, drug interactions

### Interoperability
- `fhir-integration` — FHIR R4/R5
- `hl7-v2` — ADT/ORM/ORU messaging
- `dicom-imaging` — PACS, modality worklist
- `cda-ccda` — Clinical documents
- `ihe-profiles` — XDS, PIX/PDQ, ATNA
- `terminology-services` — SNOMED CT, ICD-10, LOINC, RxNorm
- `tefca-hie` — TEFCA, QHINs, HIEs

### Compliance & Security
- `hipaa-compliance`
- `phi-handling` — De-identification, encryption
- `healthcare-cybersecurity`
- `21-cfr-part-11`
- `gdpr-health-data`
- `hitrust-csf`
- `audit-logging`

### Digital Health & AI/ML
- `telehealth-platform`
- `patient-portal`
- `remote-patient-monitoring`
- `wearables-integration`
- `clinical-ai-ml`
- `fda-samd`
- `health-chatbots`

### Revenue Cycle & Operations
- `medical-coding`
- `billing-claims` — EDI X12
- `prior-authorization`
- `value-based-care`

### Public & Population Health
- `public-health-reporting`
- `population-health-analytics`
- `clinical-research`

### Patient Experience
- `health-content-writing`
- `patient-engagement`
- `accessibility-healthcare`

### Specialized / Cross-cutting
- `medical-imaging-ai`
- `genomics-precision-medicine`
- `health-data-lake` — OMOP, FHIR bulk export

## Contributing

PRs and issues welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) — note the elevated bar for safety/accuracy in healthcare content.

## License

[MIT](LICENSE) — Use these however you want, but verify before relying on them in safety-critical systems.
