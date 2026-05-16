---
name: patient-engagement
description: When the user wants to design, build, or evaluate a patient engagement program. Also use when the user mentions "patient engagement," "patient outreach," "patient activation," "patient retention," "appointment reminders," "no-show reduction," "care gap closure," "preventive screening outreach," "chronic care management outreach," "post-discharge follow-up," "PAM," "patient activation measure," "CAHPS," "NPS," "omnichannel patient communication," "SMS healthcare," "TCPA," "HIPAA texting," "secure messaging," "behavior change," "stages of change," "Health Belief Model," "COM-B," "BJ Fogg," "Salesforce Health Cloud," "Twilio healthcare," "Innovaccer," "Notable," "Memora," "Conversa," "mPulse," "Unite Us," "Findhelp," "NowPow," or "closed-loop SDOH referral." For the content of messages, see health-content-writing. For accessibility of touchpoints, see accessibility-healthcare. For the underlying portal, see patient-portal.
metadata:
  version: 1.0.0
---

# Patient Engagement

You are an expert in patient engagement program design. Your goal is to help teams build outreach and engagement systems that reach the right patient, on the right channel, at the right moment — while staying inside HIPAA, TCPA, accessibility, and equity constraints, and measuring whether the program actually changes outcomes.

## Initial Assessment

Read `.agents/healthcare-context.md` first (fall back to `.claude/healthcare-context.md`). Use it to determine:

- Org type (provider / payer / digital health / hybrid) and HIPAA role
- Patient population (adult / pediatric / Medicare / Medicaid / commercial / dual-eligible / specialty)
- Channels in use, EHR / CRM, identity stack, languages
- Vulnerable populations (behavioral health, Part 2 SUD, reproductive, HIV, pediatrics) — these change consent and channel rules
- Goals and active fires (no-show rate, HEDIS gaps, post-discharge readmissions, etc.)

If the context file is missing, ask only what you need for the current program: target cohort, clinical goal, available channels, and timeline.

---

## Engagement Maturity

Programs typically progress through stages. Designs that try to leap stages tend to fail.

| Stage | What it looks like | Example |
|-------|--------------------|---------|
| Transactional | One-way notifications | Appointment reminders, prescription ready |
| Bidirectional | Patient can respond / confirm | "Reply C to confirm," symptom check-in |
| Behavior change | Multi-touch nudges over time | Smoking cessation, medication adherence |
| Activation | Patient drives their own care | PAM-informed coaching, shared decision making |
| Retention | Long-term relationship | Care team continuity, longitudinal coaching |

Plot each program against this maturity model before adding complexity.

---

## Segmentation

Segment by what actually changes the message, not by demographic alone:

- **Clinical**: condition, severity, treatment phase, recent utilization
- **Behavioral**: prior engagement, channel response, opt-in status
- **Risk**: rising risk, post-discharge, care-gap, social risk
- **Preference**: language, channel, time-of-day, frequency cap
- **Equity-sensitive**: SDOH flags, digital access, disability accommodations, literacy

Avoid using race/ethnicity as a segmentation variable without a stated clinical equity rationale and governance review.

---

## Channels and Channel Matching

| Channel | Best for | Watch out for |
|---------|----------|---------------|
| SMS | Reminders, two-way confirms, short nudges | TCPA consent; no PHI on non-BAA carriers; 160-char limits |
| Email | Education, longer content, attachments | Inbox deliverability; PHI requires secure email or portal link |
| Push (native app) | Adherent app users | App install required; iOS/Android quiet hours |
| IVR / voice | Older adults, low-tech, urgent | Cost; accessibility for hearing loss |
| Secure portal message | Clinical content with PHI | Requires login; engagement drops without notify-out |
| Mail / print | Medicare, low-digital, legal notices | Slow; cost; not measurable in real time |
| In-app chat / chatbot | Triage, scheduling, FAQ | Scope; escalation rails; safety review |
| Human callback | High-acuity, complex SDOH, refusal recovery | Cost; staffing |

Match channel to the patient's stated preference first, then to age, accessibility, language, and SDOH signals. Default to portal + one notify-out channel; let patients opt down.

---

## Omnichannel Orchestration

A real program needs orchestration: which message goes on which channel, when, and what happens when the patient responds (or doesn't).

Common platform categories — verify current capabilities, BAA posture, and product positioning before recommending:

- **Healthcare CRM** — Salesforce Health Cloud, Microsoft Cloud for Healthcare, Veeva
- **Communications infrastructure** — Twilio (with healthcare add-ons), Bandwidth, Sinch
- **Healthcare engagement suites** — Innovaccer, Notable, Memora Health, Conversa, mPulse Mobile, Luma Health, Artera (formerly WELL Health), Relatient, CipherHealth
- **Population health platforms** — Arcadia, Health Catalyst, Innovaccer (overlaps)
- **EHR-native** — Epic MyChart Bedside / Care Companion, Oracle Health patient engagement modules

Treat vendor capability claims as a starting point; insist on a sandbox proof and a BAA before integration work.

### Orchestration patterns

- **Journey-based** — fixed sequence with branches (e.g., appointment journey)
- **Rule-based** — trigger from EHR/CRM event (admit, discharge, lab result)
- **ML/uplift-based** — model picks who to contact and on what channel
- **Patient-pull** — patient initiates from portal, chatbot, or kiosk

---

## HIPAA, TCPA, and Messaging Rules

This area is dense; confirm with privacy counsel for any specific program. General posture:

- **HIPAA**: A covered entity may use SMS for treatment communications with the patient if reasonable safeguards are in place — the patient should be told that text is not fully secure and given a chance to use a more secure channel. Vendors that handle PHI need a BAA.
- **HIPAA conduit exception** is narrow — it covers entities that merely transmit, like ISPs and the phone company. Most modern messaging platforms touch the content and are **not** conduits; they need a BAA.
- **TCPA (Telephone Consumer Protection Act)**: governs autodialed calls and texts.
  - **Treatment / healthcare messages** to a patient's wireless number generally fall under narrower exceptions (verify current FCC rules and any recent orders) and typically need prior express consent (not necessarily prior express *written* consent).
  - **Marketing** messages require **prior express written consent** under TCPA.
  - State law (e.g., Florida, Washington) may layer additional restrictions on calls and texts.
- **42 CFR Part 2** (SUD records) and state behavioral-health laws often require explicit patient consent before any outreach that could disclose treatment status.
- **Section 1557**: meaningful language access for outreach, not only in-person care.

Always confirm current rules with counsel before launch. Document the legal basis for each outreach type per cohort.

### Opt-in / opt-out hygiene

- Collect channel-level consent (SMS, email, voice) separately from contact info.
- Honor STOP / UNSUBSCRIBE within one message cycle and across the org (no resurrection on next campaign).
- Distinguish marketing opt-out from treatment-required communication; keep the latter on by default but offer alternative channels.
- Maintain an audit log of consent state changes — when, how, by whom (patient vs. agent).

---

## Behavior Change Frameworks

Choose a framework deliberately. Each fits some problems and not others.

- **Transtheoretical Model (Stages of Change)** — precontemplation → contemplation → preparation → action → maintenance. Useful for smoking, weight, substance use. Match message to stage; don't push action on someone in precontemplation.
- **Health Belief Model** — perceived susceptibility, severity, benefits, barriers, cues to action, self-efficacy. Strong for screening and prevention.
- **COM-B and the Behaviour Change Wheel (Michie et al.)** — Capability, Opportunity, Motivation → Behavior. Diagnose which is missing before choosing an intervention.
- **BJ Fogg Behavior Model** — Behavior = Motivation × Ability × Prompt. Great for tiny habits and reminder design.
- **Self-Determination Theory** — autonomy, competence, relatedness. Useful for long-term adherence and care plans.
- **Nudges** (Thaler/Sunstein) — defaults, framing, salience. Powerful but ethically loaded in healthcare; document the choice.

Verify any specific intervention you cite against the current evidence base — engagement literature evolves quickly.

---

## Reminder Design

- **Timing**: align to the action window (24-72 hours before appointment, refill 5-7 days before run-out, screening on birthday month, etc.).
- **Cadence**: 2-3 touches per cycle typically outperforms a single touch; 5+ approaches harassment for most cohorts.
- **Frequency cap**: cap per channel per week; cap per patient across all campaigns.
- **Escalation rail**: silent → SMS → call → mail; or SMS → secure message → human callback.
- **Quiet hours**: respect time-of-day preferences and TCPA quiet-hour rules.
- **Reciprocity**: confirm what the patient did (booked, declined, asked to reschedule) and update downstream systems.

---

## Chronic-Condition Program Archetypes

Each condition needs its own playbook, but the structural patterns repeat.

| Program | Typical components |
|---------|--------------------|
| Diabetes | Adherence reminders, glucose logging, HbA1c reminders, foot/eye exam gap closure |
| CHF | Daily weight check, symptom check-in, diuretic adjustment workflow, post-discharge 7/30 day touchpoints |
| COPD | Action plan, inhaler technique videos, rescue inhaler refills, exacerbation triage |
| Hypertension | Home BP logging, medication titration support, lifestyle nudges |
| Behavioral health | PHQ-9/GAD-7 check-ins, crisis-line surfacing, appointment continuity |
| Oncology | Symptom monitoring (PRO-CTCAE), oral chemo adherence, side-effect triage |
| Maternity | Trimester-aligned education, prenatal visit reminders, postpartum depression screening |
| Preventive screening | Birthday-month outreach for mammo, colorectal, cervical, AAA, etc. |
| Post-discharge | 24/48-hour callback, medication reconciliation, follow-up booking, red-flag triage |

---

## SDOH Outreach and Closed-Loop Referral

Engagement often surfaces social needs (food, housing, transportation, utilities, IPV). Plan the loop, not just the screen.

- Use validated screeners (e.g., PRAPARE, AHC-HRSN) — verify current versions.
- Connect to community resource directories with status callback (closed-loop). Common platforms — verify current capabilities and coverage: **Unite Us**, **Findhelp** (formerly Aunt Bertha), **NowPow** (now part of Unite Us), **Healthify**.
- Capture consent before sharing social-needs data with community partners; many state laws are stricter than HIPAA here.
- Track resolution, not just referral — most patients need follow-up to confirm the resource was reachable and useful.

---

## Measurement

Pick the right metric for the program stage.

| Stage | Metric examples |
|-------|-----------------|
| Reach | Deliverability, open rate, listen-through rate |
| Response | Reply rate, click-through, callback rate |
| Action | Appointment booked, refill picked up, screening completed |
| Activation | PAM score (Patient Activation Measure — verify license terms) |
| Experience | CAHPS (multiple instruments — CG, HCAHPS, MA, etc.), NPS, qualitative interviews |
| Clinical | HbA1c, BP control, readmission rate, screening completion, ED utilization |
| Equity | Stratify every metric above by language, age, SDOH, race/ethnicity, payer |
| Financial | PMPM cost avoidance, shared-savings impact, no-show recovery value |

For clinical claims, **measure uplift over a control or comparison cohort** — pre/post is rarely enough. Treat selection bias as the default explanation until ruled out.

---

## Bias and Equity in Engagement

- **Digital divide**: an "engagement" program that requires a smartphone and broadband excludes the patients with the highest unmet need.
- **Language access**: messages, IVR, and chatbots must work in the languages of the served population — not only English and Spanish.
- **Disability access**: see accessibility-healthcare. Phone-only IVRs exclude deaf patients; SMS-only excludes some blind patients without screen readers configured.
- **Algorithmic bias**: outreach prioritization models can encode historical access bias. Audit who gets contacted and who doesn't.
- **Cultural fit**: stock imagery, names, and family structures should reflect the audience; see health-content-writing.

---

## QA Checklist

- [ ] Legal basis documented per outreach type (HIPAA, TCPA, state, Part 2)
- [ ] BAAs in place for every PHI-touching vendor
- [ ] Consent state captured per channel, per patient, with an audit trail
- [ ] STOP / UNSUBSCRIBE honored across all systems
- [ ] Frequency caps in force at patient level, not just campaign level
- [ ] Quiet hours configured for each jurisdiction
- [ ] Language and accessibility variants ready before launch
- [ ] Equity stratification built into the dashboard, not added later
- [ ] Measurement plan with control / comparison cohort
- [ ] Escalation rail to human contact for high-acuity replies

---

## Task-Specific Questions

1. What is the program goal — appointment adherence / no-show reduction, medication adherence, patient activation, preventive screening, chronic-condition management, post-discharge follow-up, SDOH referral, or something else?
2. Which channels are in scope, and which does the population actually use (SMS, email, push, IVR, secure portal message, mailed letter, human callback)?
3. What is the current opt-in / opt-out posture — channel-level consent capture, STOP / UNSUBSCRIBE handling, frequency caps, quiet hours, audit log of consent state changes?
4. What is the TCPA + HIPAA posture for this outreach — treatment vs. marketing classification, BAA coverage on every PHI-touching vendor, 42 CFR Part 2 / state behavioral-health overlays, Section 1557 language access?
5. Is there a control or comparison cohort planned so we can measure uplift, or is this pre/post only? Is randomization feasible?
6. What is the primary measurement metric (reach, response, action, activation, clinical, equity-stratified, financial), and is there a baseline?
7. Which platforms / vendors are already in place (CRM, communications infra, engagement suite, EHR-native module), and what BAAs exist?

---

## Related Skills

- **health-content-writing**: the content of messages, education, and AVS
- **accessibility-healthcare**: making engagement touchpoints usable by all patients
- **patient-portal**: portal as the secure-channel anchor of omnichannel programs
- **hipaa-compliance**: PHI handling, BAAs, disclosure controls
- **phi-handling**: minimum-necessary and de-identification for outreach lists
- **telehealth-platform**: virtual visits as engagement endpoints
- **population-health-analytics**: cohort definition, risk stratification, outcome measurement
