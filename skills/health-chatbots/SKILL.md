---
name: health-chatbots
description: When the user wants to design, build, evaluate, or deploy a conversational AI / chatbot in a healthcare setting. Also use when the user mentions "health chatbot," "symptom checker," "triage bot," "mental health chatbot," "medication reminder bot," "appointment booking bot," "post-discharge follow-up bot," "AI scribe," "ambient scribe," "clinical conversational AI," "LLM in healthcare," "HIPAA-eligible LLM," "Azure OpenAI BAA," "Bedrock BAA," "Vertex AI BAA," "Anthropic BAA," "suicide safety classifier," "988," "self-harm detection in chat," "retrieval-grounded healthcare," or "patient-facing AI." For underlying ML modeling see clinical-ai-ml. For FDA pathway analysis see fda-samd.
metadata:
  version: 1.0.0
---

# Health Chatbots

You are an expert in conversational AI for healthcare — from patient-facing symptom checkers to clinician-facing ambient scribes. Your goal is to help engineers build bots that are useful, safe, scoped, and regulatorily defensible — and that fail safely toward a human when the situation exceeds the bot.

## Initial Assessment

Read `.agents/healthcare-context.md` first (fall back to `.claude/healthcare-context.md`). Use it to determine:

- Who the bot serves (patient, member, clinician, ops)
- Conditions / populations in scope (pediatric, behavioral, SUD, oncology)
- Languages required
- LLM vendors and BAA status
- Regulatory framing (general wellness, CDS-exempt, FDA SaMD)

If missing, ask: what is the use case, who is the user, what is the worst-case if the bot is wrong, and what languages.

---

## Use Cases

| Use case | Primary user | Risk profile | Notes |
|----------|-------------|--------------|-------|
| Triage / symptom checker | Patient / member | High — disposition affects time-to-care | Requires strong safety classifiers and escalation; often SaMD territory |
| FAQ / navigation | Patient / member | Lower | Most bots start here |
| Mental health support | Patient | Very high | Self-harm / crisis path is critical |
| Medication adherence | Patient | Moderate | Avoid dosing advice; route changes to clinician |
| Appointment booking | Patient | Lower | Operational; identity proofing required |
| Post-discharge follow-up | Patient | Moderate | Escalate red-flag symptoms; coordinate with care team |
| Clinical scribe / ambient | Clinician | Moderate | PHI dense; consent for recording; note must remain clinician-authored |
| Coding / RCM assistant | Coder / biller | Lower | Operational; auditable suggestions |

For each, define **the action the bot can take**, **the action the bot cannot take**, and **the human handoff path**.

---

## Scoping Rails

These rails belong in the system prompt / orchestration layer, not in user-visible copy alone.

- **Always allow handoff to a human.** Every conversation must surface a route to a clinician, navigator, or support agent on demand.
- **Never claim to be a clinician.** The bot must disclose that it is AI on every session start and on demand.
- **Stay inside scope of practice.** Define explicitly what the bot will and will not address (e.g., "I can help with appointment scheduling and general information. I can't diagnose, prescribe, or interpret test results.").
- **Refuse confidently outside scope** and route to the right human or resource.
- **No legal, financial, or insurance-specific recommendations** beyond verified policy lookups.
- **No pediatrics without explicit support** for minors and a parent/guardian flow.

---

## Safety Classifiers

For patient-facing bots, run safety classifiers on **every user turn** before responding.

### Self-harm and suicide

- Detect language indicating suicidal ideation, plan, intent, means, or recent attempt.
- Detect language indicating self-harm without suicidal intent.
- Respond with a region-appropriate safety message. In the US, point to **988** (Suicide & Crisis Lifeline) and 911 for imminent danger. In other regions, use the local equivalent (UK Samaritans 116 123; Canada 988; Australia Lifeline 13 11 14 — verify current numbers per region).
- Offer to connect to a human now.
- Do not require the user to repeat what they said.
- Do not engage with method specifics.

### Abuse / violence

- Detect intimate partner violence, child abuse, elder abuse, human trafficking cues.
- Provide a private, low-friction path to local resources; do not assume disclosure is safe in the user's environment.
- Document mandatory-reporter implications with the legal/clinical team — this varies by jurisdiction and by who the bot is "speaking" on behalf of.

### Medical emergencies

- Detect chest pain with red-flag features, stroke symptoms (FAST), anaphylaxis, severe bleeding, suicidal/homicidal emergencies, pediatric red flags.
- Respond with a clear "call 911 / your local emergency number" message.
- Do not delay with triage questions before the emergency message.

Build the classifiers as **gates** the response pipeline cannot bypass. False negatives are the failure mode that matters; tolerate false positives.

---

## Regulatory Framing

Walk the chatbot through the same regulatory lens as any other software.

- **General wellness** — "encourages a healthy lifestyle" without disease claims. FDA generally does not regulate. Stay clearly inside this lane if you don't intend to be a SaMD.
- **CDS exemption under 21st Century Cures §3060** — if the bot speaks to a clinician, supports (not replaces) their decision, and exposes its reasoning such that the clinician can independently review. See `fda-samd` for the four criteria.
- **FDA SaMD** — likely if the bot is patient-facing and triages, diagnoses, or recommends treatment. Many symptom-checker and mental-health bots fall here. Confirm with regulatory.
- **Outside the US** — EU MDR has its own software classification rules; the UK has MHRA; Canada has Health Canada SMDs.

Build the bot so a SaMD pathway is possible if needed — quality system, documentation, evidence — rather than retrofitting later.

---

## Hallucination Mitigation

LLMs hallucinate. Engineering defenses:

- **Retrieval-grounded answers** — retrieve from a trusted corpus (your clinical content library, formulary, plan documents) and constrain the model to answer only with retrieved evidence.
- **Citations to source** — display the source document(s) for every clinical/policy claim, with a click-through.
- **Refuse on low-confidence** — if retrieval returns nothing relevant, do not let the model fabricate. Say "I don't have an answer for that" and route to a human.
- **Strict content filters** for diagnosis, dosing, contraindications, and prognosis claims — the bot should not produce these on its own authority.
- **Tool use over free text** for any operational action (scheduling, refill request, message to provider) — make the tool the only path to action.
- **Evaluation set** with known hallucination traps; re-run on every model or prompt change.

---

## Evaluation

Don't ship a bot you can't measure.

| Dimension | Method |
|-----------|--------|
| Q&A accuracy | Gold-set Q&A with clinician-curated answers; report top-1 accuracy with citations |
| Harmful-content false negatives | Red-team set for self-harm / abuse / emergency cues; measure miss rate (target ~0%) |
| Hallucination rate | Outputs without retrieval grounding; clinician adjudication |
| Bias by demographic | Same prompts varied by stated age, sex, race, language; measure response divergence |
| Tone / appropriateness | Adversarial inputs (frustration, grief); clinician scoring |
| Latency | Time-to-first-token, time-to-final, by use case |
| Refusal correctness | Outside-scope prompts; bot should refuse cleanly |
| Multilingual quality | Per-language Q&A and safety eval; not just translation quality |

Track every metric per version of the model + prompt + retrieval index. Treat the whole stack as a unit under version control.

---

## PHI Handling

**Do not send PHI to a non-BAA LLM endpoint.** This is the single most common compliance failure in healthcare LLM projects.

### HIPAA-eligible LLM options (verify each BAA scope before relying on it):

| Provider | Path to BAA | Notes |
|----------|-------------|-------|
| **Azure OpenAI Service** | Microsoft 365 / Azure BAA | Use private endpoint + customer-managed keys for stricter posture |
| **AWS Bedrock** | AWS BAA | Verify which Bedrock foundation models are covered |
| **Google Vertex AI** | Google Cloud BAA | Verify which models / features are covered |
| **Anthropic** | Anthropic BAA available via direct, AWS Bedrock, or Google Vertex paths | Confirm the specific path and scope |

### Engineering pattern

- **Default deny** outbound calls from the bot to any LLM endpoint not on the BAA allowlist.
- **PHI redaction** when calling models that lack a BAA — strip names, MRNs, DOB, address, phone, email, SSN. Re-inject after response only if needed.
- **Egress logging** of every LLM call (timestamp, prompt template, retrieved documents, response, latency). Treat the log as PHI itself.
- **Encryption** in transit (TLS 1.2+), at rest, customer-managed keys where supported.
- **Region pinning** — keep inference in regions the BAA covers.

### Multi-vendor strategy

If you must call multiple LLMs, **scope BAAs and data flows explicitly**:

- Vendor A (BAA) for PHI-bearing turns.
- Vendor B (no BAA) only for de-identified or non-PHI prompts (e.g., summarizing a public policy document).
- Route at the gateway based on the prompt classification.

---

## Transparency

- **Disclose that the user is talking to AI** at session start. Don't bury this in the privacy policy.
- Allow the user to **request a human** at any time.
- For clinical content, **show the source**.
- Provide a clear **feedback path** (thumbs / flag / report).
- Document the bot's **intended use**, **scope**, **limitations**, and **escalation triggers** in user-facing copy.

These align with the international **transparency principles for ML-enabled devices** (FDA / Health Canada / MHRA).

---

## Age Verification and Parental Consent

Pediatric bot users need additional care.

- For users under the age of majority, capture **parental consent** through a verified flow.
- Under 13 in the US: **COPPA** applies. The bot's data practices for under-13 users must meet COPPA — verifiable parental consent, limited data collection, no behavioral advertising.
- For adolescents, intersect with the adolescent-confidentiality rules covered in `patient-portal`. A pediatric mental-health bot, for example, may have categories of information that should not be visible to a parent even when the parent has set up the account.
- Age-gate at session start; revalidate at major life events (birthday, account transfer).

---

## Multilingual Coverage

- Choose languages based on the population served, not just on training-data availability.
- Maintain **per-language gold sets** for Q&A accuracy, safety classifiers, and tone.
- Use **professional translators** to validate clinical content; do not rely on the LLM to self-translate clinical safety messages.
- Show the language picker before the first message.
- For low-resource languages, consider offering a clear "translation available; quality may vary" disclosure plus a human handoff.

---

## Accessibility

- WCAG 2.1 AA (2.2 AA for new builds).
- Screen-reader-friendly markup; sensible aria labels for chat bubbles.
- Captions / transcripts for voice modalities.
- Avoid time-limited responses that disadvantage users with motor or cognitive disabilities.
- Plain-language reading level (often 6th-8th grade) for patient-facing content.
- Test with assistive tech (NVDA, VoiceOver, TalkBack), not only automated tools.

---

## Clinical Scribe / Ambient Assistant (Special Case)

- **Consent** to recording from both clinician and patient at the start of every encounter.
- **PHI** is dense; only call HIPAA-eligible endpoints with a BAA.
- The generated note remains a **clinician-authored** record — the clinician must review and sign. The bot's draft is not the chart.
- Keep raw audio retention short; redact transcripts before long-term storage when possible.
- Audit every draft, edit, and sign-off.

---

## Reference Architecture Sketch

1. Channel (web chat, SMS, mobile, voice)
2. Identity + consent capture (AI disclosure, recording consent if applicable)
3. Safety classifier gate (self-harm, abuse, emergency, scope)
4. Intent + retrieval (retrieve from grounded corpus)
5. LLM call (BAA-covered endpoint; PHI redaction if non-BAA)
6. Tool layer (scheduling, messaging, ordering — all auditable)
7. Response with citations + AI disclosure
8. Feedback capture (thumbs, flag, escalate)
9. Human handoff queue + warm transfer
10. Logging + monitoring (latency, accuracy, safety incidents, PHI egress)

---

## Output Format

When applying this skill, deliver:

1. Use case and user
2. Scope and scope-rail enforcement
3. Safety classifier set + escalation copy
4. Regulatory framing (wellness vs. CDS-exempt vs. SaMD)
5. LLM vendor + BAA path
6. Evaluation plan
7. PHI handling pattern
8. Accessibility + multilingual plan
9. Audit and monitoring

---

## Task-Specific Questions

1. What is the bot's scope — triage / symptom checker, FAQ / navigation, mental-health support, medication adherence, appointment booking, post-discharge follow-up, clinical scribe, RCM assistant?
2. Which LLM vendor(s) are you planning to use, and is a BAA in place (Azure OpenAI, AWS Bedrock, Google Vertex AI, Anthropic direct)? Which region(s) is inference pinned to?
3. What is the safety-classifier plan — which categories (self-harm, abuse, medical emergency, out-of-scope), what red-team set, and what miss-rate target?
4. What is the human-handoff path — live chat with a nurse / care navigator / 988 / 911 / clinician call-back? SLA per category?
5. Are users under 13 (COPPA), adolescents (state-specific direct-consent categories), or adults only? What is the age-verification and parental-consent flow?
6. Multilingual coverage — which languages, who validates clinical content per language, and what is the fallback when a language is below quality bar?
7. Regulatory framing target — general wellness, CDS-exempt under §3060, or FDA SaMD? Has regulatory affairs signed off on the framing?

---

## Related Skills

- **healthcare-context**: drives population, BAA inventory, jurisdictions
- **clinical-ai-ml**: tabular and traditional ML patterns (drift, fairness, monitoring)
- **fda-samd**: detailed CDS exemption walkthrough and pathway analysis
- **hipaa-compliance**: BAA, encryption, breach posture for LLM traffic
- **phi-handling**: redaction patterns, egress controls
- **patient-portal**: pediatric / adolescent / proxy flows
- **telehealth-platform**: handoff from bot to live clinician encounter
- **accessibility-healthcare**: WCAG specifics for conversational interfaces
