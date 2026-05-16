# Accessibility Regulations Cheatsheet for Healthcare

Reference for the accessibility statutes and standards that healthcare products commonly need to satisfy. **This is orientation, not legal advice.** Accessibility law has moved quickly since 2022 — verify the current text and effective dates of every regulation cited here with counsel before relying on any specific obligation.

## Contents

- How to Use This Cheatsheet
- US Federal
  - ADA Title II
  - ADA Title III
  - Section 504 of the Rehabilitation Act
  - Section 1557 of the Affordable Care Act
  - Section 508 of the Rehabilitation Act
- US State Examples
- International
  - EN 301 549 and the European Accessibility Act (EAA)
  - Web Accessibility Directive (EU)
  - AODA (Ontario)
  - Accessible Canada Act
  - UK Equality Act
  - Australian DDA
- Sectoral Overlays in Healthcare
- Standards Referenced by Regulation
- Quick Decision Table
- Common Pitfalls

---

## How to Use This Cheatsheet

1. Identify the **product type** (consumer web, mobile app, telehealth, kiosk, clinical / B2B, hardware).
2. Identify the **jurisdiction(s)** the product operates in.
3. Identify the **funding / contracting posture** — federal funds, federal procurement, state procurement, private commercial.
4. Identify any **sectoral overlays** — Medicare, Medicaid, VA / DoD, 340B, behavioral health, reproductive, pediatrics, Title X.
5. Map to the regulations below to scope the obligation.
6. Verify the current text and effective dates with counsel before quoting specific obligations to a customer or regulator.

---

## US Federal

### ADA Title II

**Scope**: State and local government services, programs, and activities.

**Healthcare relevance**: State Medicaid websites and member portals; county / city hospital systems; state health departments; public university health systems.

**Accessibility specifics**: A 2024 DOJ rulemaking ties Title II web and mobile-app accessibility to **WCAG 2.1 AA** for public entities, with phased effective dates based on entity population (verify current effective dates and phase schedule with counsel). Public entities may have a slightly different conformance window depending on their size.

**Practical implication**: If you sell to a state, county, or city health entity, expect Title II conformance language in the procurement.

### ADA Title III

**Scope**: Places of public accommodation.

**Healthcare relevance**: Doctors' offices, hospitals, clinics, pharmacies, urgent care, and other healthcare facilities have long been Title III "public accommodations." DOJ guidance and case law (e.g., *Robles v. Domino's*) extend Title III to websites and apps connected to those physical places of public accommodation.

**Accessibility specifics**: DOJ has historically declined to specify a single technical standard for Title III web content. Courts and DOJ settlements commonly use **WCAG 2.1 AA** as the de facto baseline. Verify the current DOJ guidance and any binding case law in your circuit.

**Practical implication**: Patient-facing websites and apps of any provider organization carry Title III risk. Plaintiffs' firms file high volumes of Title III web-accessibility suits annually.

### Section 504 of the Rehabilitation Act

**Scope**: Recipients of federal financial assistance.

**Healthcare relevance**: Essentially every Medicare- and Medicaid-participating provider and plan; HHS-grantee organizations; federally qualified health centers (FQHCs); academic medical centers receiving NIH funding.

**Accessibility specifics**: HHS has updated its Section 504 rule to incorporate **WCAG 2.1 AA** for web and mobile content, with phased effective dates. Verify the current rule and effective dates by entity type.

**Practical implication**: Section 504 conformance is now a real, specific technical obligation — not just a non-discrimination principle.

### Section 1557 of the Affordable Care Act

**Scope**: Health programs and activities receiving federal financial assistance, health programs administered by an executive agency, or health programs established under Title I of the ACA.

**Healthcare relevance**: Most US healthcare entities are touched by 1557 because of Medicare and Medicaid funds.

**Accessibility specifics**: 1557 prohibits discrimination based on race, color, national origin, sex, age, or disability. The disability prong requires effective communication and accessibility consistent with Section 504. The national-origin prong requires meaningful access for individuals with **limited English proficiency** — translated taglines and notices in the top languages of the state, qualified interpreters, restrictions on using minor children / family as interpreters.

The implementing regulation has shifted across administrations. Verify the **current** 1557 implementing rule with counsel for: notice posting language, the language threshold count for taglines, the prohibition on family interpreters, telehealth applicability, and any health-plan specific provisions.

**Practical implication**: 1557 is the single most-cited federal authority for healthcare accessibility *and* language access. Address both together.

### Section 508 of the Rehabilitation Act

**Scope**: Federal procurement — technology purchased, developed, maintained, or used by federal agencies.

**Healthcare relevance**: Products sold to VA, DoD, HHS, CMS, IHS, ONC, FDA, NIH, BARDA, and similar federal health entities must meet Section 508.

**Accessibility specifics**: The **Revised 508 Standards** (effective 2018) map technical requirements to **WCAG 2.0 Level A and AA** for web and electronic content. Verify the current revision (the standard updates periodically).

**Practical implication**: Federal procurement requires a **VPAT/ACR** documenting conformance. Bogus VPATs have led to lawsuits. Have a real auditor sign.

---

## US State Examples

Several states have their own accessibility statutes that layer on top of federal law:

- **California**: Unruh Civil Rights Act, Disabled Persons Act, and case law applying ADA to commercial sites operating in California (regardless of where the company is based).
- **New York**: Human Rights Law and state-level Section 504 / 508-style standards for state agencies.
- **Illinois**: Illinois Information Technology Accessibility Act (IITAA) — state-procurement standard.
- **Massachusetts**: Enterprise IT Accessibility Standards for state agencies.
- **Texas**: Texas Government Code 2054 / 1 TAC 206 / 213 for state agencies (Texas "mini-508").
- **Washington**: Policies 188 and accompanying standards for state agencies.

For any state-government healthcare buyer (state Medicaid, state hospital authority, state public-health agency), expect state-specific accessibility procurement language.

---

## International

### EN 301 549 and the European Accessibility Act (EAA)

**Scope**: EN 301 549 is the European Telecommunications Standards Institute's harmonized accessibility standard for ICT. It is the technical basis for both:

- The **Web Accessibility Directive** — applies to public sector bodies in EU member states.
- The **European Accessibility Act (EAA)** — applies to private-sector products and services including e-commerce, banking, and many digital services, with compliance dates from June 2025 onward (verify the current applicable categories and dates).

**Healthcare relevance**: EU member-state public healthcare bodies fall under the Web Accessibility Directive. EU-facing patient apps and digital-health services need to consider EAA scope.

**Accessibility specifics**: EN 301 549 incorporates **WCAG 2.1 AA** (verify the current version of EN 301 549; periodic updates align with WCAG releases).

### Web Accessibility Directive (EU)

**Scope**: Public-sector bodies in EU member states — including public healthcare entities.

**Accessibility specifics**: WCAG 2.1 AA via EN 301 549. Each member state has implementing legislation; specifics vary.

### AODA (Accessibility for Ontarians with Disabilities Act)

**Scope**: Public-sector organizations, large private organizations (50+ employees), and any organization providing services in Ontario.

**Healthcare relevance**: Ontario hospitals, clinics, and digital health services.

**Accessibility specifics**: WCAG 2.0 AA (verify whether Ontario has moved to a newer version).

### Accessible Canada Act (ACA)

**Scope**: Federally regulated organizations across Canada (banking, telecom, transportation, federal public service, and some federally regulated healthcare).

**Accessibility specifics**: Builds toward WCAG 2.1 AA (verify current text and regulations).

### UK Equality Act 2010

**Scope**: Private and public sector in the UK. Public sector also has the Public Sector Bodies (Websites and Mobile Applications) Accessibility Regulations 2018.

**Healthcare relevance**: NHS and private UK healthcare.

**Accessibility specifics**: WCAG 2.1 AA for public-sector bodies under the 2018 regulations; private-sector obligation is the broader "reasonable adjustments" principle.

### Australian DDA (Disability Discrimination Act)

**Scope**: Private and public sector in Australia. Australian Human Rights Commission guidance references WCAG 2.0 AA (verify whether they've moved to 2.1 or 2.2).

**Healthcare relevance**: Australian healthcare providers and digital-health services.

---

## Sectoral Overlays in Healthcare

These layer on top of the general accessibility regulations and are easy to miss:

- **42 CFR Part 2 (SUD records)**: not strictly an accessibility statute, but its disclosure restrictions interact with telehealth captioning, transcripts, and recordings.
- **HIPAA**: accessibility-friendly authentication that allows password managers and avoids paste-blocking still has to meet HIPAA's reasonable safeguards.
- **CMS Conditions of Participation** for hospitals reference effective communication (and tie into Section 504 / 1557).
- **CMS Star Ratings and Medicare Advantage** include patient experience and access measures.
- **The Joint Commission** standards (R3 reports periodically) on effective communication, language access, and cultural competence.
- **Title VI of the Civil Rights Act of 1964** — national-origin / language access for federal-fund recipients (the 1557 language-access framework derives from Title VI).
- **VPAT/ACR procurement language** in private-sector hospital RFPs (large health systems increasingly require ACRs for vendor purchases).

---

## Standards Referenced by Regulation

| Standard | What it is | Where it's used |
|----------|------------|-----------------|
| WCAG 2.0 AA | W3C 2008 baseline | Section 508 (Revised), older state laws |
| WCAG 2.1 AA | W3C 2018, adds mobile and low-vision criteria | ADA Title II (HHS Section 504, current DOJ rules), EN 301 549, most current procurement |
| WCAG 2.2 AA | W3C 2023, adds 9 new criteria including 3.3.8 accessible authentication, 2.5.8 target size, 2.4.11 focus not obscured | Recommended design target; not yet broadly named in regulation but coming |
| WCAG 3.0 / "Silver" | W3C draft, outcomes-based model | Not yet a conformance target; track and prepare |
| EN 301 549 | ETSI standard | EU Web Accessibility Directive, EAA |
| Revised Section 508 Standards | US Access Board, 2018 | Federal procurement |
| VPAT (current edition: ITI / INT) | Voluntary template that produces an ACR | Procurement across jurisdictions |
| HHS Office for Civil Rights guidance | HHS-specific | Section 504, 1557 enforcement |

---

## Quick Decision Table

| Situation | Likely regulations |
|-----------|---------------------|
| Patient portal of a hospital that takes Medicare | ADA Title III, Section 504, Section 1557 |
| Member portal of a state Medicaid managed-care plan | ADA Title II, Section 504, Section 1557, state procurement |
| Selling a digital health product to the VA | Section 508, Section 504, VPAT/ACR required |
| Telehealth app sold to private medical groups | ADA Title III, Section 1557 if those groups take Medicare |
| EU-facing patient app | EN 301 549, Web Accessibility Directive (if public-sector buyer), EAA (private sector) |
| Selling to Ontario hospital | AODA, federal Accessibility Act if federally regulated |
| State-public-university medical center | ADA Title II, Section 504, state procurement |
| Telehealth platform used by deaf patients | All of the above + Section 1557 effective-communication and sign-language interpretation routing |

---

## Common Pitfalls

1. **"We're only WCAG 2.0 AA because that's what 508 says."** Section 504 (post-update) and many states now reference 2.1 AA; design to 2.2 AA today.
2. **Treating 1557 as only language access.** It is disability access too.
3. **Treating language access as only Spanish.** Section 1557 generally requires the top non-English languages in the relevant state; verify the current threshold.
4. **Bogus VPATs.** Procurement is increasingly sophisticated; have a real auditor sign.
5. **Ignoring sectoral overlays.** 42 CFR Part 2 interacts with telehealth captioning and transcripts in ways most accessibility teams miss.
6. **Quoting specific paragraph numbers without verifying current text.** Accessibility law has moved fast; verify every citation with counsel before putting it in a customer-facing document.
7. **Treating regulation as the goal.** Conformance is the floor. The population — older, low-vision, low-literacy, motor-impaired, deaf, hard-of-hearing, cognitively diverse — is the actual standard.

Verify all regulatory citations with counsel before relying on any specific obligation.
