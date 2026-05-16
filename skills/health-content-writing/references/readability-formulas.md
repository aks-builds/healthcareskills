# Readability Formulas for Patient-Facing Health Content

Reference for choosing, applying, and interpreting readability formulas. Formulas are blunt instruments — pair every score with human review and, where stakes are high, plain-language testing with real patients.

## Contents

- Why Use Multiple Formulas
- Flesch-Kincaid (Grade Level + Reading Ease)
- SMOG (Simple Measure of Gobbledygook)
- FRY Readability Graph
- Dale-Chall
- Gunning Fog
- Target Grade Levels by Content Type
- Tooling
- Common Pitfalls

---

## Why Use Multiple Formulas

Each formula measures something different. Running 2-3 against the same piece gives a more honest picture:

- Sentence-length-driven formulas (Flesch-Kincaid, Gunning Fog) can be fooled by short fragmented prose.
- Syllable-driven formulas (SMOG, Flesch) miss short jargon ("stent," "graft," "INR," "DVT").
- Word-list formulas (Dale-Chall) catch words that are short but unfamiliar.

If three formulas roughly agree, you have a defensible reading-level claim. If they diverge, the content has structural or vocabulary issues worth investigating.

---

## Flesch-Kincaid

Two related scores published together by most tooling.

**Flesch Reading Ease** — 0-100, higher is easier. Rough buckets:
- 90-100 — very easy, ~5th grade
- 80-90 — easy, ~6th grade
- 70-80 — fairly easy, ~7th grade
- 60-70 — standard, ~8th-9th grade
- 50-60 — fairly difficult, ~10th-12th grade
- 30-50 — difficult, college
- 0-30 — very difficult, college graduate

**Flesch-Kincaid Grade Level** — translates to US grade level directly.

**Strengths**: Universally available, fast, well understood by reviewers.
**Weaknesses**: Treats short clinical words ("CHF," "Rx," "mg") as easy. Penalizes long but familiar words.

---

## SMOG

Counts polysyllabic words (3+ syllables) in 30 sentences. Widely used in health communication and well-validated for medical content.

**Use when**: You want a single defensible number for patient education materials. Many HHS and CDC plain-language assessments call out SMOG explicitly (verify current guidance).

**Strengths**: Built for adult health-literacy work. Easy to calculate by hand if needed.
**Weaknesses**: Needs at least 30 sentences to be reliable; shorter pieces give noisy scores.

---

## FRY Readability Graph

Plots average sentence length vs. average syllables-per-100-words on a graph; the intersection gives a grade level.

**Use when**: Doing manual review on a short piece (a single AVS section, a consent paragraph) or training clinical writers; the visual is more intuitive than a single number.

**Strengths**: Visual, no software required, well-respected in literacy circles.
**Weaknesses**: Manual; same blind spots as other syllable-based formulas.

---

## Dale-Chall

Uses a list of approximately 3,000 "familiar" words. Words outside the list count as hard, regardless of syllables.

**Use when**: Writing for low-literacy / safety-net / Medicaid populations, or when the content is short and jargon-dense. Catches "stent," "lesion," "edema," "INR" that Flesch lets through.

**Strengths**: Best validated formula for low-literacy audiences.
**Weaknesses**: The word list is updated periodically; verify you're using a current edition. Some implementations use an outdated list.

---

## Gunning Fog

Sentence length plus percentage of complex words (3+ syllables, excluding proper nouns, compound words, common suffixes). Gives a US grade level.

**Use when**: A second opinion alongside Flesch-Kincaid. Often used in business writing; reasonable but not health-specific.

**Strengths**: Familiar to non-clinical reviewers.
**Weaknesses**: Same syllable-based blind spots; not health-specific.

---

## Target Grade Levels by Content Type

| Content Type | Target | Notes |
|--------------|--------|-------|
| General patient education | 6th grade | Common AHRQ / NIH / CDC recommendation; verify current guidance |
| Safety-net / Medicaid / low-literacy | 4th-5th grade | Pair with visuals and teach-back |
| Plain-language consent (companion to legal form) | 6th-8th grade | Legal team must still approve the legal form |
| After-visit summary | 6th grade | One page when possible |
| Secure messages / portal copy | 6th-8th grade | Phone-read; short |
| Discharge instructions | 5th-6th grade | Patient is stressed, often in pain |
| Pediatric (read by child) | Age-appropriate | Often paired with caregiver version |
| Behavioral-health / sensitive | 6th grade | Trauma-informed phrasing matters more than score |

A sick, scared, or in-pain reader processes 2-3 grade levels below their everyday reading. Target accordingly.

---

## Tooling

Common options — verify current product positioning and offline-vs-cloud posture (some send PHI to a third-party endpoint; never paste real PHI into a public tool):

- **Microsoft Word / Outlook readability statistics** — Flesch Reading Ease + Flesch-Kincaid; built in.
- **Hemingway Editor** — grade level plus structural feedback (passive voice, adverbs, complex sentences).
- **Readable.com**, **WebFX Readability Test Tool**, **Datayze Readability Analyzer** — multiple formulas at once.
- **textstat** (Python) — open-source library implementing Flesch, SMOG, Dale-Chall, Gunning Fog, ARI, Coleman-Liau, etc. Run locally on PHI-safe infrastructure.
- **Health-literacy assessment tools** — PEMAT (Patient Education Materials Assessment Tool, AHRQ) is not a readability formula but a structured rubric that complements them; verify current edition.

---

## Common Pitfalls

1. **Reporting one score**. Run 2-3 formulas; if they disagree, dig in.
2. **Scoring fragments**. Most formulas need a few hundred words and at least ~10 sentences to be reliable; SMOG needs 30.
3. **Treating the score as the goal**. A 5th-grade-level piece can still be wrong, condescending, or culturally tone-deaf.
4. **Ignoring jargon that scores easy**. "Stent" and "graft" pass Flesch and Gunning Fog but lose patients.
5. **Pasting PHI into public web tools**. Use offline tools (Word, textstat) or a vetted, BAA-covered service for any text containing patient details.
6. **Skipping human review**. Run a clinician for accuracy and at least one plain-language reviewer (ideally a patient or community-health worker from the audience) for clarity.
7. **Forgetting translation**. Reading level in the source language does not transfer to translated text; rescore in-language after transcreation.
