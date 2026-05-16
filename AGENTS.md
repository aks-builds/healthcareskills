# AGENTS.md

Guidelines for AI agents working in this repository.

## Repository Overview

This repository contains **Agent Skills** for AI agents working on healthcare software, following the [Agent Skills specification](https://agentskills.io/specification.md). Skills install to `.agents/skills/` (the cross-agent standard). This repo also serves as a **Claude Code plugin marketplace** via `.claude-plugin/marketplace.json`.

- **Name**: Healthcare Skills
- **Focus**: Health informatics, EHR/EMR integration, interoperability (FHIR/HL7/DICOM), HIPAA & healthcare compliance, clinical AI/ML, telehealth, RCM, public health
- **License**: MIT

## Repository Structure

```
healthcareskills/
├── .claude-plugin/
│   └── marketplace.json    # Claude Code plugin marketplace manifest
├── skills/                 # Agent Skills
│   └── skill-name/
│       ├── SKILL.md        # Required skill file (<500 lines)
│       └── references/     # Optional deep-dive docs loaded on demand
├── AGENTS.md
├── CLAUDE.md
├── CONTRIBUTING.md
├── LICENSE
├── README.md
└── VERSIONS.md
```

## Agent Skills Specification

Skills follow the [Agent Skills spec](https://agentskills.io/specification.md).

### Required Frontmatter

```yaml
---
name: skill-name
description: What this skill does and when to use it. Include trigger phrases.
metadata:
  version: 1.0.0
---
```

### Frontmatter Field Constraints

| Field         | Required | Constraints                                                      |
|---------------|----------|------------------------------------------------------------------|
| `name`        | Yes      | 1-64 chars, lowercase `a-z`, numbers, hyphens. Must match dir.   |
| `description` | Yes      | 1-1024 chars. Describe what it does and when to use it.          |
| `license`     | No       | License name (default: MIT)                                      |
| `metadata`    | No       | Key-value pairs (author, version, etc.)                          |

### Name Field Rules

- Lowercase letters, numbers, and hyphens only
- Cannot start or end with hyphen
- No consecutive hyphens (`--`)
- Must match parent directory name exactly

**Valid**: `fhir-integration`, `hipaa-compliance`, `clinical-decision-support`
**Invalid**: `FHIR-Integration`, `-hipaa`, `clinical--decision`

### Optional Skill Directories

```
skills/skill-name/
├── SKILL.md        # Required - main instructions (<500 lines)
├── references/     # Optional - detailed docs loaded on demand
├── scripts/        # Optional - executable code (kept minimal)
└── assets/         # Optional - templates, data files
```

## Writing Style Guidelines

### Structure

- Keep `SKILL.md` under 500 lines (move details to `references/`)
- Use H2 (`##`) for main sections, H3 (`###`) for subsections
- Use bullet points and tables liberally — clinical/regulatory content benefits from structured reference data
- Short paragraphs (2-4 sentences max)

### Tone

- Direct and instructional
- Second person ("You are an expert in FHIR R4 integration")
- Professional, precise, and conservative — readers may be building safety-critical software

### Clarity Principles

- Clarity over cleverness
- Specific over vague
- Active voice over passive
- One idea per section

### Description Field Best Practices

The `description` is critical for skill discovery. Include:
1. What the skill does
2. When to use it (trigger phrases users actually say)
3. Related skills for scope boundaries

```yaml
description: When the user wants to design or implement FHIR integrations. Also use when the user mentions "FHIR," "R4," "R5," "SMART on FHIR," "FHIR resource," "FHIR bundle," "FHIR search," or "USCDI." For lower-level HL7 v2 messaging, see hl7-v2. For SMART OAuth app development, see smart-on-fhir.
```

## Healthcare-Specific Authoring Rules

These rules are mandatory because misinformation in a healthcare context can directly cause patient harm or legal liability.

### Safety

- **Never invent codes**: do not fabricate ICD-10, CPT, SNOMED CT, LOINC, RxNorm, or NDC codes. If you reference a code, it must be one you've verified — otherwise say "look up the current code from the source of truth."
- **Never invent dosages, diagnoses, or contraindications**: skills are for *software engineering* tasks, not medical advice generation.
- **Never invent regulatory citations**: HIPAA, GDPR, 21 CFR Part 11, EU MDR, and similar regulations have specific section numbers. Cite only what you know; otherwise refer the reader to the official text.
- **Cite the source of truth**: every regulatory or terminology claim should link to (or name) the authoritative source.

### PHI handling in examples

- Use synthetic identifiers in every example (e.g., `MRN: 123456`, `Patient/example-1`).
- Never paste a real name, DOB, address, phone, email, MRN, or full-face image into any skill or reference doc.
- When showing audit log examples or request/response payloads, redact or synthesize.

### Scope

- These skills target engineers, informaticists, security teams, and product builders.
- They are **not** for generating medical advice to patients.
- When the user appears to be asking for a clinical recommendation (dosing, diagnosis, treatment choice), the skill should redirect them to clinical resources or a licensed clinician.

## Skill Categories

| Category | Skills (representative) |
|----------|------------------------|
| Foundation | `healthcare-context` |
| Clinical informatics & EHR | `ehr-integration`, `clinical-decision-support`, `clinical-documentation`, `cpoe-orders`, `smart-on-fhir`, `medication-reconciliation` |
| Interoperability | `fhir-integration`, `hl7-v2`, `dicom-imaging`, `cda-ccda`, `ihe-profiles`, `terminology-services`, `tefca-hie` |
| Compliance & security | `hipaa-compliance`, `phi-handling`, `healthcare-cybersecurity`, `21-cfr-part-11`, `gdpr-health-data`, `hitrust-csf`, `audit-logging` |
| Digital health & AI/ML | `telehealth-platform`, `patient-portal`, `remote-patient-monitoring`, `wearables-integration`, `clinical-ai-ml`, `fda-samd`, `health-chatbots` |
| Revenue cycle & ops | `medical-coding`, `billing-claims`, `prior-authorization`, `value-based-care` |
| Public & population health | `public-health-reporting`, `population-health-analytics`, `clinical-research` |
| Patient experience | `health-content-writing`, `patient-engagement`, `accessibility-healthcare` |
| Specialized | `medical-imaging-ai`, `genomics-precision-medicine`, `health-data-lake` |

See `README.md` for the canonical list.

## Skill Cross-References

Every skill references `healthcare-context` as its foundation. Beyond that, skills reference each other for scope clarity (e.g., `fhir-integration` points at `smart-on-fhir` for OAuth, at `terminology-services` for code systems, at `hipaa-compliance` for transmission requirements).

When adding a new skill, include a **Related Skills** section at the bottom listing 3-6 closely related skills.

## Git Workflow

### Branch Naming

- New skills: `feature/skill-name`
- Improvements: `fix/skill-name-description`
- Documentation: `docs/description`

### Commit Messages

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

- `feat: add fhir-integration skill`
- `fix: correct CDS Hooks card structure in clinical-decision-support`
- `docs: update README`

### Pull Request Checklist

- [ ] `name` matches directory name exactly
- [ ] `name` follows naming rules (lowercase, hyphens, no `--`)
- [ ] `description` is 1-1024 chars with trigger phrases
- [ ] `SKILL.md` is under 500 lines
- [ ] No real PHI, codes invented, or unverified regulatory citations
- [ ] References section at the bottom links related skills
