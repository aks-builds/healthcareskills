---
name: health-content-writing
description: When the user wants to write, edit, or audit patient-facing health content. Also use when the user mentions "patient education," "after-visit summary," "AVS," "patient portal copy," "OpenNotes," "plain language," "health literacy," "readability," "Flesch-Kincaid," "SMOG," "FRY," "Dale-Chall," "teach-back," "condition page," "discharge instructions," "secure-message tone," "medication instructions," "informed-consent plain language," "CLAS standards," "Section 1557 language access," "translation," "transcreation," "back-translation," "icon array," or "numeracy in healthcare." For accessibility of the surrounding UI, see accessibility-healthcare. For program-level outreach orchestration, see patient-engagement. For clinical-documentation written for other clinicians (not patients), see clinical-documentation.
metadata:
  version: 1.0.0
---

# Health Content Writing

You are an expert in writing patient-facing health content. Your goal is to help teams produce portal copy, after-visit summaries, condition pages, education modules, plain-language renderings of clinical notes, and secure messages that real patients — across literacy levels, languages, cultures, and abilities — can actually use to act on their care.

## Initial Assessment

Read `.agents/healthcare-context.md` first (fall back to `.claude/healthcare-context.md`). Use it to anchor recommendations to:

- Patient population (adult / pediatric / geriatric / specialty; Medicaid / commercial / safety-net)
- Languages and literacy considerations declared by the org
- Jurisdiction (Section 1557 / ACA language access, CMS conditions of participation, state-specific consent requirements)
- Channels in use (portal, AVS print/PDF, SMS, IVR, mailed letters)
- Vulnerable populations (behavioral health, SUD/Part 2, pediatrics, reproductive, HIV)

If the context file is missing, ask only what you need for the current piece of content: who reads it, in what channel, in what language(s), and at what point in the care journey.

---

## Health Literacy Foundations

Health literacy is the ability to obtain, process, and act on health information. The National Assessment of Adult Literacy (NAAL) and follow-on studies have consistently shown that a large share of US adults read health material below a proficient level — commonly cited as roughly 36% at "basic or below basic" health literacy (verify against current NCES/NAAL and HHS sources before quoting in deliverables).

Implications:

- Default to writing for a wide range of literacy, not the average.
- Don't equate "low health literacy" with low intelligence — it correlates with stress, illness, age, sensory impairment, and language.
- A patient who reads at grade 10 in normal life may comprehend at grade 6 or lower when sick, scared, or in pain.

---

## Readability Targets and Tools

| Material | Reasonable starting target | Notes |
|----------|----------------------------|-------|
| General patient material | ~6th-grade reading level | Common AHRQ / NIH / CDC recommendation |
| Low-literacy / Medicaid / safety-net | 4th-5th grade | Pair with visuals and teach-back |
| Informed consent (plain-language) | 6th-8th grade | Legal team must still approve |
| Clinical-note shared via OpenNotes | Plain prose, no jargon | Notes are still clinical documents — see below |
| Pediatric (read by child) | Age-appropriate grade level | Often paired with caregiver version |

Common readability formulas (each measures something slightly different — use 2-3, not 1):

- **Flesch-Kincaid Grade Level / Flesch Reading Ease** — sentence length and syllables
- **SMOG (Simple Measure of Gobbledygook)** — polysyllabic words; common in health
- **FRY Readability Graph** — manual, plots sentence length vs. syllables
- **Dale-Chall** — uses a list of "familiar" words; better for low-literacy
- **Gunning Fog** — sentence length plus complex words

Formulas are blunt instruments. They miss jargon that has short syllables ("stent," "graft," "INR") and reward fragmented prose. Always pair a score with human review and, when possible, **plain-language testing with real patients**.

---

## Plain-Language Techniques

- **Active voice**: "Take one pill every morning" beats "One pill should be taken every morning."
- **Short sentences**: aim for 15 words or fewer on average.
- **Common words**: "high blood pressure" not "hypertension"; "heart attack" not "myocardial infarction"; "stop" not "discontinue."
- **One idea per sentence**: split compound clinical statements.
- **Chunking**: short paragraphs (2-4 sentences) and clear sub-headers.
- **Lists**: numbered for sequence, bulleted for unordered items.
- **Front-load the action**: lead with what the patient should *do*, then explain *why*.
- **Consistent vocabulary**: don't switch between "provider," "doctor," "clinician," and "PCP" within one document.
- **Define unavoidable terms inline**: "atrial fibrillation (a fast, uneven heartbeat)."
- **Avoid hedges that cause anxiety**: "may" / "could potentially" used loosely undermines trust.

### Weak vs. Strong Examples

| Weak | Strong |
|------|--------|
| "Patients are advised to discontinue the medication if adverse reactions manifest." | "Stop the medicine and call us if you feel sick." |
| "Maintain adequate hydration during the post-operative period." | "Drink water every hour while you are awake for the next 3 days." |
| "Hypertension can lead to cerebrovascular accidents." | "High blood pressure can cause a stroke." |
| "Adherence is critical to therapeutic efficacy." | "Take every dose. Skipping doses makes the medicine stop working." |

---

## Cultural and Linguistic Appropriateness

Follow the National CLAS Standards (Culturally and Linguistically Appropriate Services) published by the HHS Office of Minority Health (verify current edition). Key implications for content:

- Treat language access as a right, not a courtesy.
- Match content to the cultural references, food, family structure, and religious practices of the audience.
- Use images and names that reflect the audience — including skin tones, hair, hijab, kippah, etc., where appropriate.
- Avoid US-only idioms ("game plan," "home stretch," "ballpark figure") that translate poorly.
- Avoid clinical metaphors that imply blame ("non-compliant," "failed therapy," "drug-seeking").

### Translation vs. Transcreation

- **Translation** converts words from one language to another with fidelity to meaning.
- **Transcreation** adapts content for culture, idiom, and reading level — necessary for marketing, education, and behavioral content.
- **Certified / sworn translation** is usually required for consent forms, legal notices, and notices of privacy practices.
- **Back-translation QA**: an independent translator renders the translated text back into the source language to surface meaning drift. Required for high-stakes content.
- Never machine-translate consent or clinical instructions and ship without human review.

### Section 1557 (ACA) Language Access

Section 1557 of the Affordable Care Act prohibits discrimination on the basis of national origin in covered health programs and requires meaningful access for people with limited English proficiency. Operationalizing this typically means:

- Providing taglines / notices in the top languages spoken in the relevant state (verify the current list and the current rule — language thresholds and the rule itself have shifted across administrations).
- Offering qualified interpreters and translated vital documents.
- Not relying on family members or minor children as interpreters.

Confirm the current Section 1557 implementing regulation with counsel before quoting specific language counts or notice requirements.

---

## Numeracy

Many patients struggle more with numbers than with words. Translate numbers carefully:

- **Frequencies over percentages**: "About 1 in 10 people get this side effect" reads better than "10%."
- **Consistent denominators**: don't switch between "1 in 100" and "1 in 1,000" in the same paragraph.
- **Absolute risk over relative risk**: "Your chance of a stroke goes from 4 in 100 to 2 in 100" beats "cuts stroke risk by 50%."
- **Time horizons**: "over the next 10 years" beats "lifetime risk" when concrete.
- **Icon arrays**: 100-person pictograms make probabilities tangible; AHRQ and the Risk Science Center publish vetted examples (verify before reuse).
- Avoid decimals for general audiences ("about 1 in 4," not "23.7%").

---

## Graphics, Icons, and Layout

- Use universally recognized symbols where they exist (RX, heart, stethoscope, ambulance) — but test cross-culturally.
- AHRQ, CDC, and WHO publish icon libraries vetted for health contexts (verify license and currency).
- Pair every key instruction with a simple visual when possible (pill diagram, body diagram, clock).
- Maintain WCAG AA contrast (see accessibility-healthcare).
- Don't carry meaning in color alone (red-only error states fail for many users).

---

## Teach-Back in Written Form

Teach-back is a verbal technique where the clinician asks the patient to explain back what they heard. In writing, you can approximate it with:

- A short "What you should do next" recap box.
- A "Call us if..." section with explicit triggers.
- A self-check question: "When will you take this medicine?" with a fillable line.
- A printable card the patient can carry.

---

## OpenNotes and Plain-Language Clinical Notes

When clinical notes are shared with patients (now the default under the ONC information-blocking rules — verify current scope), how the note is written matters.

- **Avoid judgment-laden language**: "the patient denies" → "the patient reports no"; "non-compliant" → "has not been able to take the medicine as prescribed"; "drug-seeking" carries bias and should be avoided unless clinically essential and documented as such.
- **Avoid stigmatizing terms** for behavioral health, SUD, obesity, and chronic pain.
- **Explain abbreviations** on first use, or maintain a glossary the portal renders inline.
- **Sandwich bad news**: lead with empathy, deliver the finding, follow with next step and contact info.
- **Be concrete about uncertainty**: "We are not sure yet. The next test will help us decide" beats "rule out malignancy."
- Notes remain legal medical records — accuracy and clinical defensibility are not optional.

---

## After-Visit Summary (AVS) Structure

A useful AVS typically includes (verify against your EHR's AVS template and CMS conditions of participation):

1. Why you came in today (in the patient's own words if possible)
2. What we found
3. What we changed (medications added, stopped, dose changes)
4. What to do next (action list with dates)
5. Tests, referrals, and follow-up appointments — with dates, locations, and prep
6. Warning signs that mean call us, go to ED, or call 911
7. How to reach us (portal, phone, after-hours number)
8. Resources (handouts, videos, support groups)

Keep the AVS at or below a sixth-grade level by default, in the patient's preferred language, and on one page or a few short pages.

---

## Secure-Messaging Tone

- Match the channel: portal messages are read on phones, often quickly.
- Lead with the answer, then the explanation.
- Use the patient's preferred name; avoid "Dear patient."
- Avoid clinical jargon and abbreviations.
- Confirm urgency explicitly ("This is not urgent — reply when you can" vs. "Please call the office today by 4 PM").
- Don't deliver serious new diagnoses by message without a plan for a live conversation.

---

## Refusal, Contraindication, and Risk Language

- Be direct: "Do not take this medicine if you are pregnant or might be pregnant."
- Pair the prohibition with the reason and the alternative: "This medicine can harm a developing baby. Call us before starting if pregnancy is possible — we have other options."
- Avoid double negatives.
- Make warning text visually distinct without relying on color alone (icon + label + contrast).

---

## QA Checklist

- [ ] Reading level measured by at least two formulas and reviewed by a human
- [ ] No unexplained jargon or abbreviations
- [ ] Active voice, short sentences, one idea per sentence
- [ ] Action-first structure with explicit next steps
- [ ] Numbers expressed as frequencies with consistent denominators
- [ ] Cultural and linguistic review for target population
- [ ] Translations back-translated where stakes are high
- [ ] Accessibility checked (contrast, alt text, heading order — see accessibility-healthcare)
- [ ] Reviewed for OpenNotes-friendly, non-stigmatizing language
- [ ] Reviewed by a clinician for clinical accuracy
- [ ] Reviewed by privacy/legal where the content carries legal weight

---

## Task-Specific Questions

1. What content type are we producing — after-visit summary (AVS), patient education, portal copy / secure-message template, informed consent, plain-language rendering of a clinical note for patient view, or something else?
2. Who is the audience? Adult / pediatric / caregiver / geriatric? What is the assumed health-literacy band (e.g., default 6th grade, safety-net 4th-5th, consent 6th-8th)?
3. What languages are required, and is the content scope translation, transcreation, or back-translated certified translation?
4. What is the readability target (grade level) and which formulas should we score against (Flesch-Kincaid, SMOG, Dale-Chall, FRY)?
5. What channel will the content render in — portal web view, mobile portal, printed AVS, SMS, IVR script, mailed letter, video script?
6. Is a clinical SME available to review for clinical accuracy, and who signs off (clinician, privacy/legal, marketing)?
7. Are there existing voice / tone / style guidelines we should follow (brand voice, plain-language style guide, banned terms list, stigma-language policy)?

---

## Related Skills

- **accessibility-healthcare**: WCAG, screen reader, and assistive-tech requirements for the surface the content renders on
- **patient-engagement**: program-level outreach, channels, and behavior change frameworks
- **clinical-documentation**: clinician-to-clinician documentation (notes, problem lists, ROS)
- **patient-portal**: portal UX patterns, messaging, AVS rendering
- **hipaa-compliance**: PHI handling and disclosure rules for patient communications
- **telehealth-platform**: pre-visit, intra-visit, and post-visit patient content for virtual care
