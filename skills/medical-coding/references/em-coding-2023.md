# E/M Coding (Post-2021 Office/Outpatient and Post-2023 Inpatient)

High-level reference for engineers building E/M-suggestion or validation tools. Describes the MDM (Medical Decision Making) leveling structure for the office/outpatient revision (effective 2021) and the inpatient/observation revision (effective 2023). Does not enumerate the specific MDM tables — those are AMA-licensed content that must be sourced from the current CPT publication.

> Verify all MDM table content, code numbers, and time thresholds against the **current AMA CPT** publication and the **current CMS guidance** before encoding in software. Definitions and thresholds change.

## Contents

- What changed and when
- Leveling paths (MDM vs. time)
- MDM elements at a high level
- Time-based coding considerations
- Engineering implications
- Adjacent care-management code families

---

## What Changed and When

**Office and outpatient E/M** (the most common E/M codes in ambulatory medicine) were substantially revised effective **2021**. History and exam no longer drive the level — they must be medically appropriate but are not scored. The level is selected by **MDM** *or* by **total time on the date of the encounter**.

**Inpatient, observation, consultation, nursing facility, home/residence, and several other E/M categories** were aligned with the office/outpatient framework effective **2023** — using MDM or time, with refreshed MDM tables and category-specific rules.

> Verify current scope of which E/M code families are MDM-or-time-based vs. legacy against the current AMA CPT publication.

---

## Leveling Paths

The clinician (or the software supporting them) selects the level using **either** of two paths:

1. **MDM-based leveling** — score the encounter's MDM and pick the level whose MDM definition is met.
2. **Time-based leveling** — sum eligible time on the date of the encounter and pick the level whose time threshold is met.

The clinician chooses whichever yields the appropriate level for the work performed. Software that only supports one path will under- or over-level a meaningful share of encounters.

---

## MDM Elements (High Level)

MDM is scored across **three elements**. The level is determined by meeting the definition in **two of three** elements.

1. **Number and Complexity of Problems Addressed**
   - What was addressed during the encounter and the complexity/severity of each problem.
   - Considers acute vs. chronic, stable vs. exacerbated, threat to life or bodily function, undiagnosed new problems, etc.

2. **Amount and/or Complexity of Data Reviewed and Analyzed**
   - Categories include: review of prior external notes; review/order of tests; independent interpretation; discussion with external physician/QHP/appropriate source; independent historian.
   - Tests, documents, and discussions are counted at the category level with category-specific rules.

3. **Risk of Complications and/or Morbidity or Mortality of Patient Management**
   - Considers the risk associated with the encounter's diagnostic and treatment options, including social determinants that limit management options.
   - Examples used in the AMA-published table include things like prescription drug management, decision regarding minor vs. major surgery, decision regarding hospitalization, etc.

> The exact wording, examples, and category thresholds inside each MDM element are defined in the **AMA CPT MDM tables**. Verify against the current publication.

---

## Time-Based Coding Considerations

Time-based E/M for office/outpatient sums **total time on the date of the encounter** by the reporting clinician. Per AMA guidance, eligible activities include both face-to-face and **non-face-to-face** work on that date, such as:

- Reviewing prior records and tests
- Obtaining/reviewing separately obtained history
- Performing a medically appropriate examination/evaluation
- Counseling and educating the patient/family/caregiver
- Ordering medications, tests, or procedures
- Referring and communicating with other healthcare professionals
- Documenting clinical information in the EHR
- Independently interpreting results and communicating them
- Care coordination

Activities by other clinical staff and time on a separate date generally do not count toward the reporting clinician's time. Inpatient and other 2023-revised categories have similar but category-specific time definitions.

> Verify the eligible-activities list and the specific time thresholds per code against current AMA CPT and CMS guidance.

---

## Engineering Implications

### Extraction approach

Free-form summarization is the wrong tool for MDM scoring — MDM is a **structured rubric** with category-level evidence. Most CAC vendors use **targeted NLP plus rules**:

- Concept extractors identify problems addressed, tests ordered/reviewed, external records reviewed, discussions held, and risk-relevant interventions (prescription drug management, hospitalization decision, etc.).
- Rules / tables encode the current MDM grid (sourced from current CPT) and apply the two-of-three logic.
- Time is captured either as structured time-tracking input or extracted from documentation phrases (e.g., "X minutes spent on...").

### Surfacing to the clinician

E/M suggestion tools should:
- Show **both** the MDM-based suggested level and any time-based level when both are computable.
- Surface the **evidence** behind each MDM element score (problems, data, risk) so the clinician can verify.
- Allow the clinician to **override** with documented reason.
- Maintain an audit trail of suggestion vs. final selection.

### Version pinning

MDM table content changes. Pin the MDM rule set to the **date of service** and re-validate when AMA publishes updates. Treat a CPT release as a regression-testable event.

### Setting-specific rules

Office/outpatient and the 2023-revised categories (inpatient, observation, consult, nursing facility, home/residence) each have their own code ranges and some category-specific definitions. Engineers must select the correct code family by **setting + encounter type**, not just MDM/time.

> Verify code family and category rules against current AMA CPT.

---

## Adjacent Care-Management Code Families

Engineers building E/M suggestion or compliance tooling often touch adjacent care-management families that have their own time, staffing, consent, and documentation rules. Verify current CPT/HCPCS values and rules from AMA/CMS:

- **Annual Wellness Visit (AWV)** — Medicare HCPCS G-codes
- **Transitional Care Management (TCM)**
- **Chronic Care Management (CCM)**
- **Principal Care Management (PCM)**
- **Remote Physiologic Monitoring (RPM)** and **Remote Therapeutic Monitoring (RTM)**
- **Behavioral Health Integration (BHI)** and **Collaborative Care (CoCM)**
- **Advance Care Planning (ACP)**

These often interact with E/M on the same date and have specific overlap and modifier rules.

> Verify current code values, time/staffing requirements, and same-date overlap rules against AMA CPT and CMS guidance.

---

## Hard-Coded Values to Avoid

Do not hard-code from memory:
- MDM table thresholds and category definitions
- Time thresholds per code
- Same-date overlap rules with care-management families
- Specific code numbers for any of the revised E/M categories

Build them as **configurable reference data** sourced from the current CPT publication and updated each release. Treat the update as a release event requiring regression testing against a gold-set of expert-leveled encounters.

> Verify against current AMA CPT and CMS guidance.
