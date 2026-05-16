# GAMP 5 vs. FDA Computer Software Assurance (CSA)

A working reference contrasting two complementary approaches to validating computer systems in FDA-regulated environments: ISPE's GAMP 5 (Second Edition / v2) and FDA's Computer Software Assurance (CSA) approach for the case-for-quality. Verify the current text of GAMP 5 v2 and the current FDA CSA guidance (the CSA guidance was issued in draft form and updated; check the latest FDA.gov version) before relying on specifics. Consult QA / Validation / Regulatory Affairs for binding decisions.

## At a Glance

| Topic | GAMP 5 (v2) | FDA CSA |
|---|---|---|
| Publisher | ISPE (International Society for Pharmaceutical Engineering) | FDA |
| Scope | All GxP computerized systems | Production / quality-system software under 21 CFR Part 820 (now QMSR after the harmonization rule — verify current FDA terminology) |
| Approach | Risk-based, scaled, supplier-leveraged | Risk-based, "least burdensome" assurance, intended-use focused |
| Mindset | "Critical thinking" through the SDLC | "Critical thinking" before testing; minimize wasted documentation |
| Software categories | Cat 1 (infrastructure), Cat 3 (non-configured products), Cat 4 (configured products), Cat 5 (custom applications) | Not categorized — approach is per intended use and per process risk |
| Predicate rule applicability | Any GxP context (GCP, GMP, GLP, PV) | Primarily quality-system / production software for medical devices |

## When Each Applies

- **GAMP 5** is the de facto standard for validating GxP computerized systems in pharma, biotech, and device manufacturing globally. Used for EDC, LIMS, MES, ERP-as-GxP-record, eTMF, pharmacovigilance systems.
- **CSA** is FDA's specific guidance for software used in production or in the quality system under 21 CFR Part 820 / QMSR. It is **complementary** to GAMP 5 — many programs use GAMP 5 as the lifecycle framework with a CSA mindset for test strategy.

The two are not in conflict. CSA was issued in part to push back against over-documented IQ/OQ/PQ scripts that consumed validation budgets without proportionate quality return. GAMP 5 v2 (2022 update) substantially incorporated CSA-aligned concepts (critical thinking, leverage of supplier activities, scaled rigor).

## GAMP 5 v2 Software Categories

### Category 1 — Infrastructure Software
Operating systems, databases, middleware, IT infrastructure platforms.
- Validation: qualified through IT infrastructure qualification (network, server, OS)
- Focus: configuration management, change control, supplier qualification
- Typical effort: low for the application; infrastructure team owns

### Category 3 — Non-configured Products
COTS used out of the box without configuration of business rules. (Office productivity used in regulated context, certain calibrated instruments running fixed firmware.)
- Validation: leverage supplier evidence + verify intended use
- Focus: requirements + supplier qualification + minimal user testing
- Typical effort: low to moderate

### Category 4 — Configured Products
COTS / SaaS configured to the customer's business rules (EDC, LIMS, eTMF, ERP, MES, pharmacovigilance SaaS).
- Validation: requirements + configuration spec + risk-based testing + traceability matrix
- Focus: configuration is the validation surface; what was configured, why, and that it works
- Typical effort: moderate to high — most clinical / GMP systems land here

### Category 5 — Custom Applications
Bespoke software developed for the user.
- Validation: full SDLC — URS / FS / DS / code review / unit / integration / system / UAT / traceability
- Focus: design, code quality, testing, defect management
- Typical effort: high

## CSA Core Concepts (verify against current FDA draft/guidance)

### Critical Thinking First
Before writing a single test script, ask:
- What is the intended use of the software?
- What is the process risk if the software fails?
- What features support the intended use?
- What testing actually evidences confidence?

### Risk-Based Approach
- **High risk** — software directly impacting product quality or patient safety → unscripted exploratory + scripted + objective evidence
- **Medium risk** — software supporting quality decisions → scripted testing, scaled
- **Low risk** — software with no direct quality impact → ad-hoc / unscripted with minimal evidence

### Least Burdensome Records
- Capture objective evidence (screenshots, log files, system records) rather than retyping every step
- Use the system's own records as evidence where they answer the assurance question
- Leverage vendor / supplier evidence with documented qualification — don't re-test the supplier's COTS unit tests

### Unscripted Testing Is Acceptable
For lower-risk uses, ad-hoc, exploratory, or error-handling testing without rigid scripts is acceptable when documented. Move away from the assumption that more pages = more compliance.

## GAMP 5 + CSA Combined Pattern

A common modern pattern:

1. Define the system's **intended use** and the **regulated process** it supports
2. Determine the **GAMP 5 software category** (1 / 3 / 4 / 5)
3. Apply **CSA-aligned risk thinking** to scale validation activities
4. Generate the **validation deliverables** appropriate to the category and risk:
   - Validation plan
   - URS / FS (user requirements / functional specification)
   - Configuration spec (for Cat 4)
   - Design spec (for Cat 5)
   - Risk assessment (FMEA or equivalent — tied to patient safety / product quality / data integrity)
   - Traceability matrix
   - Test summary (scripted + unscripted, with objective evidence)
   - Validation summary report
5. Maintain through **change control, periodic review, supplier audits, deviation management**

## Common Mistakes

- Treating CSA as "less validation" — CSA is risk-based, not lighter. High-risk software still requires rigorous evidence.
- Treating GAMP 5 v2 as "the same as v1" — v2 incorporates significant CSA-aligned changes.
- Assuming a vendor's "Part 11 / GAMP / CSA compliant" label removes the user's validation obligation — it does not.
- Validating to a category lower than the actual use — a Cat 5 (custom) system documented as Cat 4 (configured) under-validates the custom code.
- Carrying forward IQ/OQ/PQ scripts from a 15-year-old SOP without re-thinking the test strategy.
- Over-documenting Cat 1 infrastructure with application-level scripts.

## Predicate Rule and GxP Mapping (Verify)

| Predicate rule context | Likely framework |
|---|---|
| GCP (clinical trials, EDC, eTMF, IRT/RTSM, eCOA) | GAMP 5 v2 |
| GMP (manufacturing, LIMS, MES, ERP-as-batch-record) | GAMP 5 v2 + CSA mindset |
| GLP (preclinical / lab systems) | GAMP 5 v2 |
| PV (pharmacovigilance / safety databases) | GAMP 5 v2 |
| Device QSR / QMSR (production and process control software) | FDA CSA specifically |
| Combination / borderline | Engage QA — both may apply |

## Final Disclaimers

- Verify the current text of GAMP 5 (v2 or later) from ISPE
- Verify the current FDA CSA guidance (draft / final / superseded) on FDA.gov
- Verify the harmonization of 21 CFR Part 820 with ISO 13485 (the QMSR rule) and its effective date
- Consult QA / Validation / Regulatory Affairs for binding determinations
