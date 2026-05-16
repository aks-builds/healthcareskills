# Security Policy

## Reporting a vulnerability

**Do not open a public GitHub issue for security or PHI-handling concerns.**

If you find:

- A skill that could plausibly lead to patient harm (unsafe clinical guidance, fabricated codes, incorrect dosing examples).
- An example that contains real Protected Health Information (PHI).
- A leaked secret, credential, or internal identifier.
- A supply-chain concern in the GitHub Actions workflows or any script under `.github/scripts/`.

please report it privately via one of the following:

1. **GitHub Security Advisories** — open a draft advisory at <https://github.com/aks-builds/healthcareskills/security/advisories/new>. This is preferred.
2. **Email** — `its.aks@outlook.com` with the subject prefix `[healthcareskills security]`.

Please include:

- The affected file path (e.g., `skills/<skill-name>/SKILL.md` and line number).
- A description of the issue and the potential impact.
- A reference or authoritative source if the issue is a factual / regulatory inaccuracy.

You should receive acknowledgement within 7 days. Coordinated disclosure timelines will be discussed case-by-case; the default is 30 days from acknowledgement to public fix.

## Scope

This repository contains **markdown skill files** plus a small Node script and GitHub Actions workflows. It does not run a server, store data, or process PHI directly. The security surface is:

- Correctness of clinical / regulatory content (most important).
- Synthetic-data hygiene in examples.
- Integrity of the workflows under `.github/`.

## Out of scope

- Issues in third-party agent runtimes (Claude Code, Cursor, Windsurf, etc.) — report those to the runtime vendor.
- Clinical advice or treatment questions — these skills are for software engineers building healthcare software, not for medical decision-making.
