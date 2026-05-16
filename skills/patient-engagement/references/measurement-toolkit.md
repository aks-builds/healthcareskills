# Patient Engagement Measurement Toolkit

Reference for picking the right metric for the right program stage, and avoiding the common measurement traps that make engagement programs look better than they are. Verify all licensed instruments (PAM, CAHPS, condition-specific PROs) for current versions and license terms before deploying.

## Contents

- Measurement Hierarchy
- Reach Metrics
- Response Metrics
- Action Metrics
- Activation Metrics (PAM)
- Experience Metrics (NPS, CAHPS)
- Clinical Outcome Metrics
- Equity Metrics
- Financial Metrics
- Study Design: Avoiding the Pre/Post Trap
- What to Avoid Measuring (or Reporting Alone)
- Dashboard Anti-Patterns
- Common Pitfalls

---

## Measurement Hierarchy

Patient engagement programs should be measured at multiple levels. A single metric at any level is misleading without the others.

| Level | What it answers | Example metrics |
|-------|-----------------|-----------------|
| Reach | Did the message arrive? | Deliverability, bounce rate, listen-through, render rate |
| Response | Did the patient engage with the message? | Reply rate, click-through, callback rate, app open rate |
| Action | Did the patient do the target behavior? | Appointment booked, refill picked up, screening completed |
| Activation | Is the patient more capable of self-management? | PAM (Patient Activation Measure) — verify license terms |
| Experience | How does the program feel to patients? | CAHPS (multiple instruments), NPS, qualitative interviews |
| Clinical | Did the patient's health change? | HbA1c, BP control, readmission rate, ED utilization |
| Equity | Did the program work for everyone, or only some? | Every metric above, stratified by language, age, SDOH, race/ethnicity, payer, disability |
| Financial | Did the program return value? | PMPM cost avoidance, shared-savings impact, no-show recovery value |

A program that improves reach without improving clinical outcomes is sending messages, not changing health. A program that improves the average clinical outcome while widening disparities is failing CLAS and Section 1557.

---

## Reach Metrics

- **Deliverability** — SMS delivered, email rendered (not bounced or filtered to spam), voice call answered or completed, mail not returned.
- **Render rate** — for email, the share that actually loaded (images, HTML, tracking pixel — note PHI implications of pixels).
- **Carrier filtering rate** (SMS) — silent drops are common; monitor 10DLC and short-code performance separately.
- **Address-quality returns** (mail) — particularly high in Medicaid; refresh via NCOA quarterly.

**What good looks like**: Deliverability should be >95% for SMS, >90% for email (transactional), >85% for mail. Below those, you have an infrastructure problem before you have a content problem.

---

## Response Metrics

- **Reply rate** (SMS two-way) — confirm / decline / reschedule replies.
- **Click-through rate** (email, SMS with links).
- **IVR completion / pass-through** — share of calls that reach the end of the menu vs. drop off.
- **App open rate from push** — share of push notifications that drove an app open within 24h.
- **Portal login rate from notify-out** — share of "you have a message" notifications that drove a portal login.

Response rates vary wildly by audience and channel. Use prior-period baselines, not industry benchmarks, to set targets.

---

## Action Metrics

- **Appointment booked** — first-touch attribution within a defined window (e.g., 7 days after reminder).
- **Appointment kept** — actual show vs. no-show.
- **Refill picked up** — pharmacy fill data, often via EHR / SureScripts.
- **Screening completed** — colorectal, mammography, cervical, AAA, etc., often via HEDIS measure logic.
- **Care-gap closed** — provider-defined gap measures.
- **Lab completed** — A1C, lipid panel, INR, etc.

Action metrics are the most defensible "what did the program do." Pair with reach and response to diagnose where the funnel breaks.

---

## Activation Metrics

### PAM (Patient Activation Measure)

A 10- or 13-item validated instrument that scores patient knowledge, skill, and confidence in self-management on a 0-100 scale, with four activation levels.

- **What it measures**: self-management readiness, not satisfaction or behavior.
- **License**: PAM is proprietary (Insignia Health, now part of Phreesia). Verify current license terms and pricing before deploying.
- **When to use**: chronic-condition programs, coaching programs, populations with mixed activation levels. Useful for tailoring intensity of intervention.
- **What to avoid**: using PAM as an outcome on its own — activation gain without clinical or utilization change is incomplete evidence.

### Alternative activation-flavored measures

- **PEPPI** (Perceived Efficacy in Patient-Physician Interactions) — narrower scope.
- **Self-efficacy scales** condition-specific (e.g., Stanford Self-Efficacy for Managing Chronic Disease).
- Verify the validation evidence and license terms for any instrument you adopt.

---

## Experience Metrics

### CAHPS

A family of survey instruments developed by AHRQ for patient experience. Different instruments for different settings:

- **CG-CAHPS** (Clinician & Group)
- **HCAHPS** (Hospital)
- **MA & PDP CAHPS** (Medicare Advantage / Part D)
- **ECHO** (Behavioral Health)
- **Home Health CAHPS**, **Hospice CAHPS**, **Surgical CAHPS**, **Dental CAHPS**, etc.

Standards-based and used in CMS Star Ratings and value-based contracts. Required for many programs. Verify current instrument version, mode (mail, phone, mixed, web), and field period rules from the AHRQ CAHPS site before fielding.

**What CAHPS measures well**: comparable experience metrics across providers / plans.
**What CAHPS does not measure well**: program-specific experience, real-time feedback, qualitative themes.

### NPS

Net Promoter Score — "How likely are you to recommend...?" on 0-10. Promoters (9-10) minus Detractors (0-6).

- Useful as a **directional** experience metric and easy to deploy.
- **Not validated** for healthcare patient experience the way CAHPS is.
- Should not replace CAHPS for regulatory or comparative reporting.
- Vulnerable to selection bias (engaged patients respond; angry patients respond; the silent middle skews the score).
- Pair with qualitative comments for any insight beyond the number.

### Qualitative interviews

Often higher signal than any survey. 8-12 interviews with patients across the engagement funnel — engaged, disengaged, opted-out, complained — typically surface issues no dashboard catches. Compensate participants.

---

## Clinical Outcome Metrics

Match the metric to the condition and the program goal. Examples:

| Condition | Common clinical metrics |
|-----------|--------------------------|
| Diabetes | HbA1c, time-in-range, hypoglycemia events, eye exam, foot exam |
| Hypertension | Average BP, % at goal (e.g., <140/90), home BP adherence |
| CHF | Readmission rate (7, 30, 90 day), ED utilization, BNP trend |
| COPD | Exacerbation rate, rescue inhaler refills, FEV1 |
| Behavioral health | PHQ-9, GAD-7, AUDIT-C, follow-up after ED visit |
| Oncology | PRO-CTCAE symptom burden, oral chemo adherence, ED visits, hospice timing |
| Maternity | Prenatal visit completion, postpartum depression screening, NICU rate |
| Preventive | Screening completion rates per HEDIS measures (verify current versions) |

For clinical claims, **measure uplift over a control or comparison cohort** (see Study Design below). Pre/post is rarely enough.

---

## Equity Metrics

Stratify every primary metric by:

- Language
- Age band
- Sex / gender
- Race / ethnicity (with governance review)
- Insurance / payer
- ZIP code (proxy for SDOH)
- Disability status
- SDOH flags (food, housing, transportation, IPV — from PRAPARE / AHC-HRSN)

A program with an average reach of 85% and a Spanish-speaking reach of 40% is not an 85% program. Build equity stratification into the dashboard from day one — adding it later is much harder.

Audit prioritization models for bias: who gets contacted, who doesn't, and does the distribution match clinical need across groups?

---

## Financial Metrics

- **PMPM cost avoidance** — per-member-per-month delta in total cost of care for engaged vs. comparison cohort.
- **Shared-savings impact** — for value-based contracts, the program's contribution to attributed savings.
- **No-show recovery value** — value of recovered appointment slots × marginal revenue per slot.
- **Readmission penalty avoidance** — CMS HRRP exposure reduction.
- **Star Rating impact** — for MA plans, the program's contribution to Star measure performance.

Financial metrics should be a **consequence** of clinical and experience improvement, not the only metric reported.

---

## Study Design: Avoiding the Pre/Post Trap

Pre/post designs systematically overstate program impact because:

- **Regression to the mean** — patients enrolled because they were at a high point (high A1c, recent ED visit) drift toward the average regardless of the program.
- **Selection bias** — patients who enroll differ systematically from those who don't.
- **Secular trends** — the world changes around the program (COVID, payer policy, new medications).
- **Care-management contamination** — other programs are running concurrently.

Better designs, in order of rigor:

1. **Randomized controlled trial** — the gold standard; requires equipoise and IRB. Possible at the message / cadence level even when the overall program is not blinded.
2. **Cluster randomization** — randomize at the clinic, panel, or geography level. Useful for orchestration-level interventions.
3. **Stepped-wedge** — staggered rollout where each unit acts as its own control during the pre-rollout phase.
4. **Propensity-matched comparison cohort** — match treated patients to similar untreated patients on observable confounders.
5. **Difference-in-differences** — compare changes over time in treated vs. comparison groups.
6. **Pre/post with concurrent controls** — better than pre/post alone, weaker than the above.
7. **Pre/post only** — only acceptable for early-stage hypothesis generation; never for claimed clinical impact.

Document the design before launch. Pre-register the primary metric and analysis plan to discipline against post-hoc cherry-picking.

---

## What to Avoid Measuring (or Reporting Alone)

- **Vanity metrics**: messages sent, total touchpoints, "engagement minutes." These move without correlation to outcomes.
- **Open rate as a clinical metric**: opens don't equal reads, reads don't equal action.
- **Self-reported adherence without verification**: "Yes, I took my meds" overstates pharmacy data by 20-40% across studies.
- **NPS as a sole experience metric** for regulatory reporting.
- **PAM gain without clinical or utilization context**.
- **Cost savings without a control cohort**.
- **Aggregate metrics that hide equity gaps**.

---

## Dashboard Anti-Patterns

- **Aggregated everything**: averages that hide stratified disparities.
- **Reach metrics only**: deliverability dashboards without action or clinical metrics.
- **No baseline**: percentage changes against unstated baselines.
- **Significance theater**: p-values on uncontrolled comparisons.
- **Selective dates**: comparing the best month against the worst month.
- **Vendor-supplied attribution** with no methodology disclosed.
- **No data quality view**: missing the rate of nulls, duplicates, and definition changes over time.

---

## Common Pitfalls

1. **Choosing the metric after seeing the data**.
2. **No primary metric**. Pre-register one.
3. **Pre/post without controls**.
4. **No equity stratification until quarter three**, when leadership asks for it.
5. **Conflating engaged-patient outcomes with program outcomes**. Engaged patients differ; that's selection bias, not program impact.
6. **Using PAM in isolation**. Activation is necessary, not sufficient.
7. **Treating NPS as CAHPS**.
8. **Vanity dashboards** with no decision attached. If no one would change behavior based on a metric moving, drop the metric.
