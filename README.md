<div align="center">

# 🏥 healthcareskills

**Give your AI agent the domain knowledge to build healthcare software correctly.**

91 production-grade skills covering the full healthcare engineering stack —
FHIR/HL7 interoperability, HIPAA compliance, clinical AI/ML, EHR integration,
telehealth, revenue cycle, patient engagement, and more.
Real-world clinical constraints built in. No hallucinated dosing.
Works with Claude Code, Claude Desktop, Cursor, Zed, Windsurf, and any
agent that speaks [MCP](https://modelcontextprotocol.io) or the
[Agent Skills spec](https://agentskills.io).

[![Validate Skill](https://github.com/aks-builds/healthcareskills/actions/workflows/validate-skill.yml/badge.svg)](https://github.com/aks-builds/healthcareskills/actions/workflows/validate-skill.yml)
[![Evals](https://github.com/aks-builds/healthcareskills/actions/workflows/eval-skills.yml/badge.svg)](https://github.com/aks-builds/healthcareskills/actions/workflows/eval-skills.yml)
[![Skills: 91](https://img.shields.io/badge/skills-91-brightgreen)](#available-skills)
[![MCP](https://img.shields.io/badge/MCP-ready-8A63D2.svg)](./mcp/)
[![Agent Skills spec](https://img.shields.io/badge/spec-agentskills.io-blue)](https://agentskills.io/specification.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

<br/>

<!-- hero image placeholder: terminal showing /fhir-integration scaffolding a FHIR R4 patient query -->
<img src="./.github/media/hero.svg" width="720" alt="Claude Code invoking the fhir-integration skill to scaffold a FHIR R4 patient query" />

<sub>☝️ Claude Code invoking <code>fhir_validate</code> — one of 220+ typed MCP tools across healthcare domains.</sub>

</div>

---

## Why healthcareskills

Healthcare software has a higher bar than most domains — HIPAA violations carry
real fines, HL7 message errors delay patient care, and hallucinated drug dosing
is dangerous. General-purpose AI agents don't know the difference between a
FHIR R4 `Patient.name.use` value set and a free-text field. These skills do.

healthcareskills gives your agent accurate, up-to-date knowledge of FHIR R4/R5,
HL7 v2, DICOM, HIPAA/HITECH, CDS Hooks, SMART on FHIR, ICD-10/SNOMED/LOINC,
and 80+ more healthcare standards and regulations. The `healthcare-context` skill
anchors every other skill to your specific org, patient population, EHR vendor,
and regulatory jurisdiction — so advice is never generic.

> **Important:** These skills are tools for *software engineering* in healthcare.
> They are not a substitute for clinical, legal, or regulatory advice, and they
> do not generate medical advice for patients. See [AGENTS.md](AGENTS.md) for
> safety rules.

## How skills work together

`healthcare-context` is the foundation — every skill reads it first to understand
your organization before offering guidance.

```
                        ┌──────────────────────────────────────┐
                        │          healthcare-context          │
                        │    (read by all skills first)        │
                        └──────────────────┬───────────────────┘
                                           │
   ┌──────────┬───────────┬────────────────┼────────────┬──────────┬──────────┐
   ▼          ▼           ▼                ▼            ▼          ▼          ▼
Clinical   Interop    Compliance       Digital      Revenue    Public &   Patient
Informatics           & Security       Health       Cycle      Pop.       Experience
```

See each skill's **Related Skills** section for the full dependency map.

## Install

### Option 1 — Claude Code Plugin (recommended)

```bash
/plugin marketplace add aks-builds/healthcareskills
/plugin install healthcare-skills
```

### Option 2 — MCP Server (Claude Desktop, Cursor, Zed, Windsurf, custom agents)

Add to your `mcp_servers` config (e.g. `.mcp.json` or `claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "healthcare-skills": {
      "command": "npx",
      "args": ["github:aks-builds/healthcareskills/mcp"]
    }
  }
}
```

Or if you have it cloned locally:

```json
{
  "mcpServers": {
    "healthcare-skills": {
      "command": "node",
      "args": ["/path/to/healthcareskills/mcp/index.js"]
    }
  }
}
```

### Option 3 — Clone and Copy

```bash
git clone https://github.com/aks-builds/healthcareskills.git
cp -r healthcareskills/skills/* .agents/skills/
```

### Option 4 — Git Submodule

```bash
git submodule add https://github.com/aks-builds/healthcareskills.git .agents/healthcareskills
```

## Usage

Once installed, ask your agent naturally:

```
"Help me design a FHIR R4 integration for our patient portal"
→ fhir_validate + patient-portal + smart-on-fhir

"Do a HIPAA risk analysis on this architecture diagram"
→ hipaa_checklist + healthcare-cybersecurity + audit-logging

"Set up an EDI 837 claims submission flow"
→ billing-claims + medical-coding

"Build a CDS Hook that warns on drug-drug interactions"
→ clinical-decision-support + medication-reconciliation + terminology-services

"Is our ambient scribe compliant with HIPAA and FDA SaMD rules?"
→ ambient-clinical-intelligence + fda-samd + hipaa_checklist
```

Or invoke skills directly:

```
/fhir-integration
/hipaa-compliance
/ehr-integration
/clinical-ai-ml
```

MCP tool calls (any MCP host):

```
fhir_validate({ resource: "Patient", payload: { ... } })
hipaa_checklist({ component: "audit-logging", context: "patient portal" })
hl7_troubleshoot({ message_type: "ADT^A01", error: "MSH segment missing" })
```

## Available Skills

<!-- SKILLS:START -->
<!-- auto-generated — do not edit -->
<!-- SKILLS:END -->

## Skill Categories

### Foundation
- `healthcare-context` — Org/patient/jurisdiction/EHR context — every other skill reads this first

### Clinical Informatics & EHR
- `ehr-integration` · `clinical-decision-support` · `clinical-documentation` · `cpoe-orders` · `smart-on-fhir` · `medication-reconciliation`

### Interoperability
- `fhir-integration` · `hl7-v2` · `dicom-imaging` · `cda-ccda` · `ihe-profiles` · `terminology-services` · `tefca-hie`
- **New:** `fhir-shorthand` · `fhir-subscriptions` · `fhir-bulk-data` · `ncpdp-pharmacy` · `smart-health-cards` · `omop-cdm` · `direct-trust-messaging` · `fhirpath-cql`

### Compliance & Security
- `hipaa-compliance` · `phi-handling` · `healthcare-cybersecurity` · `21-cfr-part-11` · `gdpr-health-data` · `hitrust-csf` · `audit-logging`
- **New:** `cms-interoperability-rules` · `information-blocking` · `ehr-certification-uscdi` · `42-cfr-part-2` · `nist-csf-healthcare` · `soc2-healthcare` · `fedramp-healthcare`

### Digital Health & AI/ML
- `telehealth-platform` · `patient-portal` · `remote-patient-monitoring` · `wearables-integration` · `clinical-ai-ml` · `fda-samd` · `health-chatbots`
- **New:** `digital-therapeutics` · `mental-health-tech` · `care-coordination` · `social-determinants` · `care-plan-management` · `chronic-disease-management` · `ai-governance-healthcare` · `ambient-clinical-intelligence` · `nlp-clinical-text` · `digital-pathology` · `predictive-analytics-clinical` · `clinical-llm-safety`

### Revenue Cycle & Operations
- `medical-coding` · `billing-claims` · `prior-authorization` · `value-based-care`
- **New:** `revenue-cycle-analytics` · `credentialing` · `remittance-processing` · `population-health-programs`

### Public & Population Health
- `public-health-reporting` · `population-health-analytics` · `clinical-research`

### Patient Experience
- `health-content-writing` · `patient-engagement` · `accessibility-healthcare`
- **New:** `patient-matching-mpi` · `consent-management` · `health-equity-analytics`

### Cloud & Engineering
- **New:** `healthcare-cloud-aws` · `healthcare-cloud-azure` · `healthcare-cloud-gcp` · `healthcare-devops` · `healthcare-observability` · `healthcare-microservices`

### Data & Analytics
- `health-data-lake`
- **New:** `clinical-data-warehouse` · `real-world-evidence` · `synthetic-patient-data` · `fhir-analytics` · `healthcare-data-quality`

### Specialized / Cross-cutting
- `medical-imaging-ai` · `genomics-precision-medicine`
- **New:** `oncology-informatics` · `pharmacy-informatics` · `laboratory-informatics` · `behavioral-health` · `genomics-data-standards` · `decentralized-clinical-trials` · `healthcare-iot` · `voice-healthcare` · `ar-vr-healthcare`

## Repository layout

```
healthcareskills/
├── .claude-plugin/
│   ├── plugin.json            Claude Code plugin manifest
│   └── marketplace.json       Claude plugin marketplace entry
├── mcp/
│   ├── index.js               MCP server entrypoint
│   ├── loader.js              SKILL.md parser + skill-graph resolver
│   └── manifest.js            Generates tool manifest from all skills
├── skills/
│   └── skill-name/
│       ├── SKILL.md           Skill instructions (v2 frontmatter)
│       ├── evals/
│       │   └── evals.json     5–6 eval scenarios (CI-benchmarked)
│       └── references/        Optional deep-dive docs loaded on demand
├── AGENTS.md                  Guidelines for AI agents working in this repo
├── CLAUDE.md                  Claude Code reminders
├── CONTRIBUTING.md
├── LICENSE
├── README.md
├── SECURITY.md
└── VERSIONS.md
```

## Contributing

PRs and issues welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) — note the elevated
bar for safety and accuracy in healthcare content. Clinical claims, regulatory
references, and standard field values must be verified before merging.

## License

[MIT](LICENSE) — use these however you want, but verify before relying on them
in safety-critical clinical systems.
