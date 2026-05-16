# Ryan Haight Act and DEA Telemedicine Flexibilities for Controlled-Substance Prescribing

The **Ryan Haight Online Pharmacy Consumer Protection Act of 2008** is the federal statute governing controlled-substance prescribing over the internet. Engineering teams building telehealth platforms that include any controlled-substance workflow must design against the **current** DEA rule — which has been a moving target since 2020.

This reference summarizes the framework. **Verify the current DEA rule and any pending rulemaking before designing or deploying a controlled-substance prescribing path.** DEA has issued multiple temporary extensions and proposed rules since the PHE; the operative rule today may differ from what is written here.

## The statutory baseline

Under Ryan Haight, a practitioner generally must conduct **at least one in-person medical evaluation** of the patient before prescribing a controlled substance via the internet. The statute then carves out a set of statutory exceptions, collectively called the **"practice of telemedicine"** exception, in 21 U.S.C. § 802(54).

The statute defines seven enumerated categories under "practice of telemedicine" that allow controlled-substance prescribing without a prior in-person evaluation:

1. **Hospital or clinic registration** — the patient is being treated by, and physically located in, a DEA-registered hospital or clinic, and the practitioner is acting in the usual course of professional practice.
2. **In-person evaluation by another practitioner** — the patient is in the physical presence of another DEA-registered practitioner who is acting in the usual course of professional practice.
3. **Indian Health Service / Tribal health** — the practitioner is an IHS or Tribal employee or contractor.
4. **Public health emergency** — the Secretary of HHS has declared a PHE and the Attorney General (DEA) has authorized telemedicine prescribing under that PHE.
5. **VA emergency** — telemedicine within the VA system in a covered medical emergency.
6. **DEA special registration** — the practitioner has obtained a special registration to engage in the practice of telemedicine. The statute authorizes this category; DEA has proposed regulations to implement it.
7. **Other circumstances specified by regulation** — a catch-all the Attorney General may use by rulemaking.

The seventh category is the one DEA has used to issue temporary flexibilities. The sixth category (special registration) has been the subject of long-running DEA rulemaking.

## What has changed since 2020

- **March 2020** — DEA invoked the PHE exception (#4) to permit controlled-substance prescribing via telemedicine without a prior in-person evaluation, subject to certain conditions.
- **May 2023** — DEA issued **temporary extensions** of the PHE flexibilities to give time for rulemaking; the extensions have been re-issued multiple times since.
- **DEA proposed rules** in 2023 on **telemedicine prescribing of controlled medications** (including a separate proposed rule for Schedule III-V buprenorphine for opioid use disorder); the agency received extensive public comment and the rules have not finalized in the form initially proposed.
- **Special-registration rulemaking** (the long-promised #6 framework) has been proposed and revised.

The net effect: as of any given month, the operative authority for telemedicine controlled-substance prescribing may be a combination of the statutory exceptions, an active PHE-era temporary extension, and any final or proposed DEA rule. **Verify the current DEA rule before designing your workflow.**

## Engineering implications

Regardless of which authority applies at any given moment, the following controls should always be present in a controlled-substance telemedicine prescribing path.

### Practitioner controls

- **Active DEA registration** for the practitioner, in the schedule(s) being prescribed, covering the location of the prescribing event and the location of the patient where required.
- **State controlled-substance registration** where the state requires one (e.g., NY, NJ, others — verify per state).
- **EPCS** (Electronic Prescriptions for Controlled Substances) certified flow with **two-factor authentication** at the time of signing. EPCS requirements apply regardless of telehealth context.
- **Practitioner identity verification** that meets DEA EPCS identity-proofing requirements.

### Encounter controls

- Capture and persist **patient location** (state) at the start of the encounter — drives both licensure and any state-specific controlled-substance rules.
- Validate the encounter against the **current DEA telemedicine authority** the platform is relying on (statutory exception category, active temporary rule, future special registration). Block the controlled-substance order path if no authority applies.
- For Schedule II opioids, additional state-level rules and PDMP (Prescription Drug Monitoring Program) checks typically apply. Integrate with the state PDMP at the point of prescribing where required.
- For buprenorphine for opioid use disorder, separate DEA / SAMHSA rules apply. The DATA-Waiver requirement (X-waiver) was eliminated by the **Consolidated Appropriations Act, 2023** for general buprenorphine prescribing, but specific telemedicine rules for buprenorphine remain a separate analysis. Verify current.

### Audit trail

- Capture: practitioner DEA number, practitioner identity proofing, two-factor signing event, the prescription content, transmission to pharmacy, patient location, the authority basis used (e.g., "statutory exception #1 — DEA-registered clinic" or "temporary rule X dated Y"), and any PDMP query result.
- Generate a FHIR `AuditEvent` per prescribing event.

### Prescribing-path block

Do not allow a controlled-substance prescription path to fire until:

1. The platform's current-authority configuration explicitly permits it for the encounter context.
2. The practitioner's DEA registration is verified and active.
3. The two-factor EPCS signing has succeeded.
4. The prescription has been transmitted to a DEA-registered pharmacy.

A common engineering failure mode is allowing the UI to expose controlled-substance prescribing while the underlying authority configuration is stale or absent. Default to deny; require an explicit, time-bounded configuration to enable.

## Non-controlled substances

For non-controlled prescriptions, Ryan Haight does not apply. Standard state e-prescribing rules govern. Use a certified e-prescribing flow, capture the prescription as a FHIR `MedicationRequest`, and follow your state's surescripts integration / EHR workflow.

## Verify-current sources

- 21 U.S.C. § 802 and 21 U.S.C. § 829 (statutory text)
- 21 CFR Part 1300 et seq. (DEA regulations)
- DEA Diversion Control Division — current rules, temporary extensions, proposed rulemakings
- 21 CFR Part 1311 (EPCS technical requirements)
- SAMHSA — buprenorphine prescribing rules
- State Board of Pharmacy / State Board of Medicine — state-level controlled-substance and PDMP rules

When in doubt, do not ship the controlled-substance path — engage your regulatory and clinical leadership to confirm the current authority basis.
