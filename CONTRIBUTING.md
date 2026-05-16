# Contributing

Thanks for your interest in improving these skills. Healthcare software is high-stakes, so we hold contributions to a high standard for accuracy and safety.

## Before You Submit

1. **Read [AGENTS.md](AGENTS.md)** — covers the spec, naming rules, and healthcare-specific authoring rules.
2. **Verify your claims** — codes, regulatory citations, and clinical workflows must be sourced. Cite the authoritative source where relevant.
3. **No real PHI** — every example must use synthetic data.

## Adding a New Skill

1. Create `skills/your-skill-name/SKILL.md` with valid YAML frontmatter (`name`, `description`, `metadata.version`).
2. Keep `SKILL.md` under 500 lines. Push detailed reference material to `skills/your-skill-name/references/`.
3. Cross-reference at least 3 related skills at the bottom of the file under **Related Skills**.
4. Add the skill to:
   - The skills table in `README.md`
   - `VERSIONS.md` with the initial version
5. Open a PR with the conventional commit format: `feat: add your-skill-name skill`.

## Improving an Existing Skill

- Bump `metadata.version` in the frontmatter (semver: patch for typo, minor for new section, major for breaking restructure).
- Update `VERSIONS.md`.
- Use the commit prefix `fix:` (correctness) or `docs:` (clarity).

## Reporting Errors

If you find a wrong code, an outdated regulatory reference, or a clinically unsafe example: **open an issue immediately**, even if you don't have a fix. Errors in this kind of content can propagate into real systems.
