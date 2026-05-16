# Health Chatbot Safety Rails Checklist

This is the safety-rail checklist for patient-facing health chatbots. Each rail is implemented as a **non-bypassable gate** in the response pipeline — not as a content guideline in the system prompt. False negatives are the failure mode that matters; tolerate false positives.

Most rails apply to any patient-facing bot. The clinical-scribe (clinician-facing) sub-case has its own considerations noted at the end.

## 1. Self-harm and suicide

### Detect

Run a self-harm / suicide classifier on **every user turn before responding**. The classifier should detect:

- Suicidal ideation (passive or active)
- Plan or method
- Intent or imminence
- Access to means
- Recent attempt
- Self-harm without suicidal intent (cutting, eating-disorder behaviors)
- Indirect language ("I can't do this anymore," "What's the point")

### Respond

When detected, respond with a clinician-curated safety template (not an ad hoc LLM output):

- A short, warm acknowledgment that does not require the user to repeat what they said
- A clear pointer to the appropriate crisis line — **US: 988 Suicide & Crisis Lifeline**, **911 for imminent danger** (verify current numbers per region)
- A one-tap call-the-number affordance
- An immediate option to connect to a human (live clinician, nurse line, crisis counselor)
- Do **not** ask triage questions before the safety message
- Do **not** engage with method specifics

### Region-appropriate crisis lines (verify current)

| Region | Crisis line | Number (verify) |
|--------|-------------|------------------|
| US | 988 Suicide & Crisis Lifeline | 988 |
| US — Veterans | Veterans Crisis Line | Dial 988 + press 1 |
| Canada | 988 | 988 |
| UK / Ireland | Samaritans | 116 123 |
| Australia | Lifeline | 13 11 14 |
| Other | Local equivalent | Verify per country |

Maintain a region-keyed crisis-line table that operations can update. Bake the table into the safety-template rendering, not into the LLM prompt.

### Audit

Capture every safety classifier hit as a logged event with timestamp, user identifier, classifier verdict, response served, and any handoff outcome.

## 2. Abuse and violence

### Detect

- Intimate partner violence
- Child abuse / neglect
- Elder abuse
- Human trafficking cues
- Sexual assault
- Threats of harm to others (homicidal ideation)

### Respond

- Provide a private, low-friction path to local resources.
- **Do not assume disclosure is safe in the user's environment** — a victim using a shared device may be monitored. Offer a "quick exit" affordance that clears the chat / closes the app.
- Provide region-appropriate hotlines (US: National Domestic Violence Hotline, Childhelp National Child Abuse Hotline, etc. — verify current).
- For homicidal ideation, the response and handoff path differ from suicide — engage clinical and legal leads on the appropriate copy and escalation.

### Mandatory reporter implications

- Some bot operators may have mandatory-reporter obligations depending on jurisdiction and the bot's principal (e.g., a bot operating on behalf of a health system whose staff are mandatory reporters).
- Work with legal and clinical leadership to determine reporting obligations.
- Document the policy in the safety runbook.

## 3. Medical emergencies

### Detect

- Chest pain with red-flag features (radiating, shortness of breath, sweating, nausea)
- Stroke symptoms (FAST: Face drooping, Arm weakness, Speech difficulty, Time to call 911)
- Anaphylaxis (swelling, difficulty breathing, hives after exposure)
- Severe bleeding
- Severe head injury with concerning features
- Pregnancy emergencies (severe pain, bleeding, decreased fetal movement)
- Pediatric red flags (lethargy in an infant, fever in newborn, breathing difficulty)
- Difficulty breathing
- Loss of consciousness

### Respond

- Clear "call 911 (or your local emergency number) now" message
- Do **not** delay with triage questions before the emergency message
- One-tap call-911 affordance
- After the emergency message, offer to connect to a human

The emergency rail and the suicide rail can both fire for the same turn. When both fire, route to both — do not let one suppress the other.

## 4. Out-of-scope refusal

### Detect

- Diagnosis ("Do I have ___?")
- Dosing recommendations
- Contraindication assessments
- Prognosis ("How long do I have?")
- Legal / financial / insurance-specific advice beyond verified policy lookups
- Recommendations outside the bot's defined scope of practice

### Respond

- Refuse cleanly and confidently.
- Route to the appropriate human or resource (clinician, nurse line, plan customer service, pharmacist).
- Do not let the LLM substitute its own recommendation when the scope rail says no.

## 5. AI disclosure (transparency)

### Always

- Disclose at session start that the user is interacting with AI.
- Allow the user to **request a human at any time**.
- For clinical / policy content, show the source.
- Provide a clear feedback path (thumbs / flag / report).

### Never

- Claim to be a clinician.
- Imply a medical license.
- Use first-person clinical language ("I, your doctor, recommend").

## 6. Consent

### Capture at session start

- AI disclosure consent (acknowledged).
- Recording / transcription consent if the modality includes voice.
- Data-use disclosure aligned with privacy policy.

### Capture at sensitive points

- Before connecting to a human, confirm the user understands the chat may be reviewed.
- Before sending any PHI to an integrated tool (scheduling, e-Rx), the consent context is captured.

## 7. Age verification and parental consent

### Detect

- Self-reported age at session start (with re-prompt on suspicious answers).
- If integrated with the patient portal, use the verified account age.

### Respond

- **Under 13 (US)** — COPPA applies. Verifiable parental consent, limited data collection, no behavioral advertising.
- **Under age of majority** — parental consent flow; intersect with adolescent confidentiality rules (see `patient-portal`).
- For pediatric mental-health bots, an additional layer: a parent who set up the account may not have access to mental-health content per state direct-consent rules.

## 8. PHI handling

- **Default-deny** outbound calls to any LLM endpoint not on the BAA allowlist.
- **PHI redaction** when calling models without a BAA — strip names, MRN, DOB, full address, phone, email, SSN. Re-inject after response only if needed.
- **Egress logging** of every LLM call (timestamp, template, retrieved documents, response, latency). Treat the log as PHI.
- **Encryption** in transit (TLS 1.2+), at rest, customer-managed keys where supported.
- **Region pinning** — keep inference in regions the BAA covers.

See `llm-vendor-baa.md` for BAA-eligible vendor paths.

## 9. Hallucination mitigation

- **Retrieval-grounded answers** — answer only from the trusted corpus.
- **Citations** — every clinical / policy claim shows the source with click-through.
- **Refuse on low-confidence** — if retrieval returns nothing relevant, say "I don't have an answer for that" and route to a human.
- **Strict content filters** for diagnosis, dosing, contraindications, prognosis claims.
- **Tool use over free text** for any operational action (scheduling, refill, message to provider).

## 10. Multilingual coverage

- **Per-language gold sets** for Q&A accuracy, safety classifiers, and tone — not just translation quality.
- **Professional translators** validate clinical safety messages — do not rely on the LLM to self-translate the 988 message.
- **Language picker** before the first message.
- For low-resource languages, surface a "translation available; quality may vary" disclosure plus a human handoff.

## 11. Accessibility

- WCAG 2.1 AA minimum (2.2 AA for new builds).
- Screen-reader-friendly chat-bubble markup with aria labels.
- Captions / transcripts for voice modalities.
- Avoid time-limited responses that disadvantage users with motor or cognitive disabilities.
- Plain-language reading level (often 6th-8th grade) for patient content.

## 12. Evaluation discipline

- Red-team set for every rail; re-run on every model / prompt / retrieval-index change.
- Target miss rate ~0% for self-harm and emergency rails. Tolerate false positives.
- Track per-language and per-subgroup performance.

## Clinical scribe (special case)

For ambient / clinical-scribe bots, the rails differ:

- **Recording consent** from both clinician and patient at the start of every encounter.
- **PHI is dense** — only call HIPAA-eligible endpoints with a BAA.
- The generated note is a **clinician-authored** record — the clinician must review and sign. The bot's draft is not the chart.
- Keep raw audio retention short; redact transcripts before long-term storage where possible.
- Audit every draft, edit, and sign-off.

## Verify-current sources

- 988 Suicide and Crisis Lifeline / SAMHSA — current number and routing
- COPPA — current rule and verifiable-parental-consent methods
- Per-region crisis-line tables
- FDA / Health Canada / MHRA transparency principles for ML-enabled devices
- ONC Information Blocking exceptions for content released to patients via portal or chat
- WCAG 2.1 / 2.2 AA criteria
