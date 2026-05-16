## New Skill

**Skill name:** `skills/SKILL-NAME`

## Summary

<!-- What does this skill do and when should it be used? -->

## Category

<!-- Clinical Informatics & EHR / Interoperability / Compliance & Security / Digital Health & AI/ML / Revenue Cycle / Public Health / Patient Experience / Specialized -->

## Checklist

- [ ] `name` matches directory name exactly
- [ ] `name` follows naming rules (lowercase, hyphens, no `--`)
- [ ] `description` is 1-1024 chars with trigger phrases
- [ ] `SKILL.md` is under 500 lines
- [ ] **Initial Assessment** section references `.agents/healthcare-context.md`
- [ ] **Task-Specific Questions** section present (5-7 numbered questions)
- [ ] **Related Skills** section present with cross-references
- [ ] `evals/evals.json` provided (5-6 evals covering mainline, sub-task, casual phrasing, multi-step, off-scope handoff)
- [ ] `references/*.md` provided where deep-dive content is helpful (plain markdown, no frontmatter)

## Healthcare safety checklist

- [ ] No real PHI in any example — synthetic identifiers only
- [ ] No invented codes (ICD / CPT / SNOMED / LOINC / RxNorm / NDC)
- [ ] No invented regulatory citations — every citation is verifiable
- [ ] No clinical advice generated — content is engineering guidance only
- [ ] AMA / SNOMED / HL7 / other third-party trademarks are mentioned but not redistributed
- [ ] Tested locally with at least one AI agent

## Linked issue (optional)

Closes #
