# Patient Engagement Channel Matrix

Reference for matching outreach channel to audience, content, and regulatory posture. Verify current FCC, state, and HHS rules with counsel before launch; this is a working orientation, not legal advice.

## Contents

- How to Read This Matrix
- Channel-by-Channel
  - SMS
  - Email
  - Push Notifications (Native App)
  - IVR / Voice
  - Secure Portal Message
  - Mail / Print
  - In-App Chat / Chatbot
  - Human Callback
- Channel Selection Heuristics
- Multi-Channel Orchestration
- Common Pitfalls

---

## How to Read This Matrix

Each channel section covers:

- **Best for** — content types and program stages where the channel earns its place
- **Audience fit** — who responds well; who is excluded
- **Regulatory posture** — HIPAA, TCPA, state overlays, accessibility expectations (always verify current rules)
- **Operational gotchas** — practical traps

Default architecture for most programs: portal as the anchor channel (secure, identity-bound) plus one notify-out channel to drive engagement back into it. Let patients opt down to their preferred mix.

---

## SMS

**Best for**: appointment reminders, two-way confirmations (Reply C / R / N), short nudges, refill reminders, screening-due prompts, post-discharge symptom check-ins.

**Audience fit**: broadest reach across age and income; near-universal in US adults. Older adults adopt SMS more readily than apps or email. Less reliable for visually impaired users without screen-reader-configured devices, and for some rural patients with intermittent coverage.

**Regulatory posture (verify current rules with counsel)**:
- **HIPAA**: A covered entity may use SMS for treatment communication if reasonable safeguards are in place and the patient has been told SMS is not fully secure and offered a more secure channel.
- **HIPAA conduit exception**: narrow — covers entities that merely transmit (ISPs, phone carriers). Most modern SMS platforms touch content (templates, scheduling, analytics) and are not conduits. BAA required.
- **TCPA**: Treatment messages to a wireless number typically need prior express consent (not necessarily written). Marketing requires prior express written consent. The FCC has issued orders specific to healthcare exceptions — verify current text.
- **State overlays**: Florida (FTSA), Washington (My Health My Data Act), and others add restrictions; check the patient's state of residence.
- **42 CFR Part 2**: SUD treatment messages typically require explicit patient consent before SMS that could disclose treatment status.

**Operational gotchas**:
- 160-character SMS limit, 70-char for Unicode (accented characters); MMS/long-SMS concatenation costs vary.
- Short codes vs. 10DLC vs. toll-free: registration, throughput, deliverability differ; verify current carrier rules.
- STOP / UNSUBSCRIBE must be honored within one message cycle and across the organization (no resurrection by a different campaign).
- Quiet hours (TCPA + state rules); time zone the patient, not the sender.
- No PHI on non-BAA carriers or downstream analytics endpoints.
- Carrier filtering can silently drop messages flagged as spam — monitor delivery receipts.

---

## Email

**Best for**: longer education content, attachments / linked PDFs, billing notices, appointment summaries, newsletters, post-visit surveys.

**Audience fit**: skews younger and more digitally engaged; deliverability is highly variable; spam filters are aggressive on healthcare-flavored content.

**Regulatory posture (verify current rules with counsel)**:
- **HIPAA**: PHI in email requires either a secure email service (encrypted in transit, BAA-covered storage) or a portal-link approach where the email only signals that a message awaits the patient inside the secure portal. Sending PHI to ordinary patient inboxes requires explicit informed acknowledgment that the patient prefers unencrypted email.
- **CAN-SPAM**: applies to marketing email; transactional / treatment email has narrower rules but still needs a working unsubscribe.
- **State overlays**: same as SMS — check Washington, California, Florida, Texas, New York at minimum.

**Operational gotchas**:
- Deliverability: warm up sending domains, configure SPF / DKIM / DMARC, monitor bounce and spam rates.
- Open-tracking pixels and click-tracking can be considered PHI-adjacent in some interpretations; review with privacy counsel.
- Inbox UI varies wildly; design for mobile.
- Subject lines that look transactional perform better than promotional for healthcare.

---

## Push Notifications (Native App)

**Best for**: in-app notifications for engaged users — appointment alerts, secure message arrivals, RPM alerts, medication reminders, daily check-ins.

**Audience fit**: app installers only — typically a small fraction of any health-plan or provider patient base. Skews younger and more digitally engaged.

**Regulatory posture (verify current rules with counsel)**:
- **HIPAA**: push payloads should not contain PHI by default (lock screens are public surfaces); send a generic notification ("You have a new message in our portal") and load PHI only after authenticated entry.
- **iOS / Android quiet hours and focus modes**: device-level overrides limit delivery; design assuming the user controls when notifications surface.
- **App store rules**: Apple and Google both have rules on health-related notifications; verify current policies.

**Operational gotchas**:
- App install requirement excludes the majority of most patient populations; never make push the sole channel.
- Deep linking from notification to in-app context requires routing logic.
- Token churn (uninstall, OS reset) breaks delivery silently.

---

## IVR / Voice

**Best for**: older adults, low-digital populations, urgent/time-sensitive (callbacks), Medicare populations, multilingual outreach, Medicaid populations with address volatility.

**Audience fit**: older adults, rural populations, and patients with low digital literacy respond better to voice than text. Excludes deaf and many hard-of-hearing patients unless paired with TTY / relay / video relay.

**Regulatory posture (verify current rules with counsel)**:
- **TCPA**: autodialed and prerecorded calls to a wireless number need prior express consent; healthcare exceptions exist for treatment but are narrower than many realize — verify current FCC orders.
- **State overlays**: state do-not-call lists, call-time restrictions (typically 8 AM - 9 PM local).
- **Accessibility**: must support TTY and 711 relay; consider video relay for ASL users (see accessibility-healthcare).

**Operational gotchas**:
- Voicemail vs. live answer: leave a short, generic message; do not disclose PHI.
- Cost per attempt is high; reserve for high-acuity or refusal-recovery touches.
- Detection of answering machine vs. live person is imperfect; design for both.
- Multilingual IVR menus need real translation, not literal — and same-day callback in-language.

---

## Secure Portal Message

**Best for**: clinical content with PHI, results sharing, after-visit summaries, secure dialogue with the care team.

**Audience fit**: only patients who have activated portal accounts and log in. Without a notify-out (email or SMS pointing the patient to the portal), engagement is low.

**Regulatory posture (verify current rules with counsel)**:
- **HIPAA**: portal is the most defensible channel for PHI; identity is bound to the authenticated session.
- **ONC information-blocking rules**: clinical notes and most results must be shared with patients without delay (verify current scope and exceptions).
- **Section 1557**: portal must offer in-language interface and meaningful access for people with disabilities.

**Operational gotchas**:
- Activation rates vary widely; portals with proxy access (parents, caregivers) need explicit consent workflows.
- Inbox fatigue: a portal that pings on every lab result can train patients to ignore it.
- Mobile-portal vs. native-app UX: most portal logins are now on mobile.
- Accessibility (see accessibility-healthcare): forms, dynamic content, focus management, screen-reader compatibility.

---

## Mail / Print

**Best for**: Medicare and Medicaid populations, legal notices (Notice of Privacy Practices, Notice of Non-Discrimination under Section 1557, EOBs), low-digital populations, regulatory mailings.

**Audience fit**: still the most reliable channel for many Medicare patients. Address volatility is high in Medicaid; expect a meaningful return rate.

**Regulatory posture (verify current rules with counsel)**:
- **HIPAA**: mail is acceptable for PHI; window envelopes that expose PHI through the cellophane are a common privacy incident.
- **Section 1557 / CLAS**: vital documents in the top non-English languages of the state (verify current rule and language threshold).
- **State overlays**: state-specific notice requirements for managed care, behavioral health, reproductive care.

**Operational gotchas**:
- Lead time: print and mail takes 5-10 business days; not suitable for time-sensitive outreach.
- Cost per piece is much higher than digital.
- Real-time measurement is not possible; survey-based attribution only.
- Address-quality vendors (NCOA, USPS Address Verification) reduce returns; refresh quarterly.

---

## In-App Chat / Chatbot

**Best for**: triage, FAQ, scheduling, refill requests, navigation help, light symptom check-in.

**Audience fit**: digitally engaged users; bias toward younger, English-fluent, smartphone-enabled audiences.

**Regulatory posture (verify current rules with counsel)**:
- **HIPAA**: chatbot platform handling PHI needs BAA; transcripts may be PHI.
- **State AI / chatbot rules**: emerging state laws on disclosure of bot-vs-human, especially in healthcare; verify current state law.
- **Section 1557 + ADA**: chatbot must be accessible (keyboard, screen reader, language).
- **FDA**: if the chatbot makes individualized clinical recommendations, it may meet SaMD definitions (see fda-samd).

**Operational gotchas**:
- Escalation rail to a human is non-negotiable, especially for clinical or behavioral-health content.
- Safety review for crisis content (suicide, IPV, child abuse, overdose) is required before launch.
- Scope creep: bots tend to drift into clinical advice; document and enforce scope boundaries.
- Logging and audit retention need to match the platform's BAA terms.

---

## Human Callback

**Best for**: high-acuity follow-up, complex SDOH, refusal-recovery, post-discharge for high-risk patients, behavioral-health continuity, multilingual outreach where qualified interpreters are needed in real time.

**Audience fit**: universal; the most trust-building channel.

**Regulatory posture (verify current rules with counsel)**:
- **HIPAA**: verify identity before disclosing PHI; document verification approach.
- **TCPA**: same call-time and consent rules as IVR.
- **Section 1557**: qualified interpreters, not family / minor children, except in narrow emergencies.

**Operational gotchas**:
- Cost is high; reserve for where it changes outcomes.
- Staffing model (in-house vs. vendor) affects training, BAA, and continuity of relationship.
- After-hours coverage and warm-handoff to clinical triage need explicit playbooks.

---

## Channel Selection Heuristics

1. **Start from the patient's stated preference**, not from what is cheapest to send.
2. **Match acuity to channel**. High-acuity content (new diagnoses, crisis, complex behavioral health) deserves a human, not an SMS template.
3. **Account for the digital divide**. SMS-default plus IVR / mail fallback covers the realistic audience for most US Medicaid and Medicare populations.
4. **Account for accessibility**. Pair every voice channel with a TTY / text alternative; pair every text channel with a screen-reader-tested rendering.
5. **Account for language**. Channel-by-channel, including IVR menus, in-language SMS, and translated portal interface.
6. **Default to portal + one notify-out channel**, let patients opt down to their preferred mix.
7. **Cap frequency at the patient level**, not just the campaign level. 2-3 touches per cycle typically outperforms 1; 5+ approaches harassment for most cohorts.

---

## Multi-Channel Orchestration

Common patterns:

- **Journey-based** — fixed sequence with branches (appointment journey: book to confirm to remind to follow-up).
- **Rule-based** — triggers from EHR / CRM events (admit, discharge, lab result, gap close).
- **ML / uplift-based** — model picks who to contact and on what channel; audit who gets contacted and who doesn't for bias.
- **Patient-pull** — patient initiates from portal, chatbot, or kiosk; the system responds.

For most chronic-care programs, expect a mix: rule-based triggers for clinical events, ML or rule-based prioritization for outreach lists, and patient-pull for everything in between. ML alone over-indexes on patients already engaged.

---

## Common Pitfalls

1. **Treating "engagement" as "send more messages."** Frequency caps and quiet hours protect retention.
2. **Defaulting to SMS-only.** Excludes the highest-need patients with the highest unmet need.
3. **Ignoring BAA scope.** "We use Twilio" is not the same as "we have a BAA covering the products we are using"; verify per product.
4. **Conflating treatment and marketing.** TCPA, HIPAA marketing rules, and Section 164.508 authorization requirements differ; mis-classification creates real liability.
5. **Not measuring with a control / comparison cohort.** Pre/post designs systematically over-estimate program impact.
6. **No equity stratification.** A program that improves the average while widening disparities is failing the populations CLAS and Section 1557 are about.
7. **Skipping the audit trail.** Consent state changes, STOP / UNSUBSCRIBE, language preference, channel preference all need an audit log with timestamps and source.
