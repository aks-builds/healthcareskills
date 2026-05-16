# Healthcare Context Discovery Interview Script

Reference script for running a guided discovery interview when the user wants a structured session to build their `.agents/healthcare-context.md` from scratch. Use this as a working outline — adapt the depth and order to the organization type and what the user has already shared. Capture answers conversationally, summarize after each section, and write to the context file at natural stopping points (after every 2-3 sections, or at the end).

## How to Run This

- Walk through one section at a time. Do not paste the whole question list at the user.
- After each section, summarize what you heard and confirm before moving on.
- Branch based on org type: payer answers differ from provider answers, which differ from digital health vendor answers.
- Use `TBD` for sections the user wants to defer; never invent values.
- Bump the `Last updated` line at the end.
- If the user wants to stop partway through, save what you have to the file and note which sections are still TBD.

---

## Section 1 — Organization

**Goal**: Establish org type, size, geography, and HIPAA / regulated role. These shape every other recommendation.

Questions:

1. How would you describe your organization? (Hospital / health system / clinic / FQHC / specialty practice / payer / pharma / med device / digital health startup / EHR vendor / health tech vendor / academic medical center / research org / public-health agency / other)
2. What's your scale? (Number of providers, beds, members, patients, or active monthly users; rough orders of magnitude are fine.)
3. Which countries, states, or regions do you operate in?
4. What's your HIPAA role? (Covered Entity / Business Associate / Subcontractor BA / not subject to HIPAA)
5. Are there other regulated roles to flag? (HITECH, GDPR controller/processor, EU MDR manufacturer, FDA SaMD manufacturer, CLIA-regulated lab, 42 CFR Part 2 SUD program, FERPA-touching school health, VA/DoD / CMS contractor, etc.)

Branch hints:
- Provider → focus next sections on patient population, EHR, clinical workflows.
- Payer → focus on members, claims, EDI, plan-side regulation.
- Digital health vendor → focus on BA role, BAAs, EHR-integration target, FDA posture.
- Pharma / med device → focus on regulated product line, FDA / EU MDR, real-world evidence.

---

## Section 2 — Patient / Member Population

**Goal**: Capture who the org serves; this changes consent rules, channel choices, language/literacy posture, and audit requirements.

Questions:

1. What is the demographic mix? (Adult / pediatric / mixed / geriatric / specialty — e.g., oncology, behavioral, maternal)
2. Are there vulnerable populations you serve at scale? (Pediatric, behavioral health, SUD / 42 CFR Part 2, HIV, reproductive health, undocumented, incarcerated, refugee, LGBTQ+, etc.) These change consent, disclosure, and audit requirements significantly.
3. What languages and health-literacy considerations matter for your population?
4. What is the rough PHI volume? (Records, transactions per day, storage size — order of magnitude is fine.)

---

## Section 3 — Regulatory Jurisdiction

**Goal**: Pin down the jurisdictions and sectoral overlays that will drive compliance work.

Questions:

1. What is your primary jurisdiction? (US federal + state, EU / EEA — which member states, UK, Canada — federal + provincial, India, Australia, etc.)
2. Are there US-state specifics that apply heavily? (CA — CMIA, CCPA/CPRA; TX — HB 300; NY — SHIELD; WA — My Health My Data; IL — BIPA, Genetic Information Privacy Act; MA — 201 CMR 17.00; others)
3. Any sectoral overlays? (Title X family planning, 42 CFR Part 2 SUD, FERPA, Title V MCH block grant, 340B)
4. What standards are you required to follow by contract or law? (HITRUST, SOC 2, HITECH, NIST 800-66, NIST 800-53 / FedRAMP, ISO 27001 / 27799)

---

## Section 4 — Clinical & Operational Systems

**Goal**: List the systems other skills will need to know about — EHR, PACS, lab, pharmacy, billing, HIE, identity.

Questions:

1. Which EHR(s) do you use, and which modules? (Epic, Oracle Health / Cerner, Meditech, Athenahealth, eClinicalWorks, NextGen, Allscripts / Veradigm, Greenway, Practice Fusion, OpenEMR, custom — Inpatient, Ambulatory, ED, OR, Anesthesia, Lab, Radiology, Pharmacy)
2. What PACS / imaging system, and is DICOMweb available?
3. What lab / LIS, pharmacy / PIS, billing / RCM systems?
4. Are you on any HIE / network? (eHealth Exchange, Carequality, CommonWell, regional HIE, Epic Care Everywhere, TEFCA QHIN)
5. Identity stack — enterprise IDP (Okta, Microsoft Entra, PingID), patient IDP (Auth0, Cognito, custom), EMPI vendor?
6. Anything else relevant — scheduling, secure messaging, telehealth platform, RPM platforms, wearable platforms, mobile apps?

---

## Section 5 — Standards & Terminology

**Goal**: Capture the technical standards in use so FHIR / HL7 / C-CDA / terminology / EDI work lands on the right version.

Questions:

1. FHIR — which version (R4, R5), which IGs (US Core, IPS, CARIN BB, Da Vinci, mCODE, others)?
2. HL7 v2 — which version (2.3.1, 2.5.1, 2.6, 2.8), which message types (ADT, ORU, ORM, SIU, MDM, BAR, etc.)?
3. CDA / C-CDA — which document types?
4. DICOM — in use?
5. Terminologies in use — SNOMED CT, ICD-10-CM, ICD-10-PCS, ICD-11, CPT, HCPCS, LOINC, RxNorm, NDC, UCUM, CVX?
6. EDI X12 — 837P / I / D, 835, 270/271, 276/277, 278, 834, 999 / TA1?

---

## Section 6 — Data & Analytics

**Goal**: Surface the data platform so analytics, ML, and population-health work fits the stack.

Questions:

1. Data warehouse / lake / lakehouse — Snowflake, BigQuery, Databricks, Redshift, Synapse, custom?
2. Clinical CDM — OMOP, PCORnet CDM, Sentinel CDM, i2b2, custom?
3. BI / reporting tools, ML platforms?
4. Bulk FHIR export consumers, if any?

---

## Section 7 — Security Posture

**Goal**: Capture current controls and certifications. Many later recommendations depend on these.

Questions:

1. Frameworks / certifications — HITRUST CSF (which level), SOC 2 (Type I / II), ISO 27001, FedRAMP, StateRAMP, Cyber Essentials?
2. Encryption posture — at-rest, in-transit, key management (KMS, HSM)?
3. Identity controls — MFA enforcement, SSO, privileged access management?
4. Audit / logging stack — SIEM, ATNA, FHIR AuditEvent collection?
5. Any known compensating controls or accepted risks worth flagging?

---

## Section 8 — AI / ML

**Goal**: Surface AI / ML in production and its regulatory posture. This area moves fast and intersects FDA SaMD, HHS CDS, and state AI laws.

Questions:

1. Is there AI / ML in production today? (Predicting what — readmission, sepsis, no-show, fall risk, decompensation; clinical decision support; patient-facing chat; ambient scribe; coding assistance; revenue-cycle automation?)
2. Regulatory status of each model — enterprise tool, HHS CDS-exempt, FDA-cleared SaMD, in research, third-party embedded?
3. Governance — model inventory, validation pipeline, drift monitoring, bias audits?

Branch hints:
- Patient-facing chat → flag the chatbot-disclosure laws and Section 1557 / accessibility implications.
- Ambient scribe → flag the PHI / cloud BAA and clinical documentation chain.
- Risk prediction → flag the bias / equity audit need.
- FDA SaMD → flag the fda-samd skill.

---

## Section 9 — Engineering Constraints

**Goal**: Capture the engineering reality so recommendations fit the team and the stack.

Questions:

1. Cloud(s) and regions — are HIPAA-eligible services / regions enforced?
2. On-prem or hybrid components?
3. Build vs. buy preferences?
4. Existing API gateway, service mesh, identity, observability?
5. Languages and stacks the team is fluent in?
6. BAA status with key vendors (cloud, AI providers, third-party libraries, analytics)?

---

## Section 10 — Goals & Priorities

**Goal**: Pin down what the user is shipping and what is on fire so downstream skills can prioritize.

Questions:

1. What are you trying to ship in the next 1-2 quarters?
2. What is on fire right now? (Audit prep, breach response, EHR cutover, ONC certification, prior auth automation, Star Ratings push, value-based care contract, regulatory deadline, etc.)
3. Who must approve work? (CMIO, CMO, CISO, Privacy Officer, Legal, Compliance, IRB, Patient Advisory Council, Board)

---

## Closing the Interview

Once all sections are captured (or marked TBD by the user):

1. Read back a short summary — org type, jurisdiction, EHR, primary standards, security frameworks, AI/ML posture, current fires.
2. Confirm with the user before writing.
3. Write the file to `.agents/healthcare-context.md` using the exact template from the skill's Output Format section. Fall back to `.claude/healthcare-context.md` if `.claude/` exists but `.agents/` does not.
4. Bump the `Last updated` line.
5. Remind the user that other healthcare skills will check this file first, and to re-run the skill when the organization changes (new EHR, new jurisdiction, new BAA, new framework certification, new patient population).
6. Treat the context file as **non-sensitive enough to commit to source control** by default — but never include patient data, named individuals, real credentials, or internal IP addresses.

---

## Quick Branch Cheatsheet

If org type is...

- **Hospital / health system** — emphasize EHR modules, HIE participation, CMS Conditions of Participation, Joint Commission, 1557, ACO contracts.
- **FQHC / community health center** — emphasize HRSA / Section 330, sliding fee, UDS reporting, vulnerable populations, 340B.
- **Payer** — emphasize members vs. patients, claims systems, EDI, NCQA, CMS Star Ratings, provider directory, prior auth.
- **Digital health vendor (B2B)** — emphasize BA role, BAAs, EHR integration target, SOC 2 / HITRUST trajectory, customer compliance commitments.
- **Digital health vendor (DTC)** — emphasize patient-facing app status (not necessarily a covered entity, may still be regulated under FTC Health Breach Notification Rule, state My Health My Data Act, GDPR if EU users).
- **Pharma / med device** — emphasize regulated product line, FDA / EU MDR, real-world evidence platforms, post-market surveillance, MDUFA, MAUDE.
- **Academic medical center** — emphasize research role, IRB, HIPAA + FERPA + Common Rule overlap, NIH / FDA grants.
- **Research org** — emphasize Common Rule, IRB, data use agreements, OMOP / PCORnet / Sentinel CDM, GDPR if EU subjects.
- **Public-health agency** — emphasize Title V, immunization registries, syndromic surveillance, ELR, public health authority exception under HIPAA.

If jurisdiction is...

- **US only** — focus on HIPAA, state overlays, 1557, Section 504, ADA, FDA where relevant.
- **EU / EEA** — focus on GDPR (Article 9 health data, lawful basis), EU MDR / IVDR if devices, EHDS (verify status), national overlays per member state.
- **UK** — focus on UK GDPR, Data Protection Act 2018, NHS digital standards, MHRA for devices.
- **Canada** — focus on PIPEDA, provincial laws (Ontario PHIPA, Alberta HIA, BC PIPA), federally regulated work under ACA.
- **Australia** — focus on Privacy Act 1988, My Health Records Act, TGA for devices.
- **Cross-border** — flag the matrix and ask which jurisdiction takes precedence for which data flow.

If patient population includes...

- **Behavioral health / SUD** — flag 42 CFR Part 2 and state behavioral-health laws.
- **Pediatrics** — flag parental access / proxy consent, adolescent confidentiality (state-specific).
- **Reproductive health** — flag the post-Dobbs cross-state-disclosure question, 1557, state-specific restrictions.
- **HIV** — flag state-specific disclosure laws.
- **Genetic data** — flag GINA, state genetic-privacy laws (IL, CA, others).
- **Limited English proficiency populations** — flag 1557 language access.

Use this script as a starting point, not a checklist to read verbatim. The goal is a context file that downstream skills can rely on to give correct, applicable advice.
