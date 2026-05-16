# ACMG/AMP Germline Variant Classification

Reference summary of the ACMG/AMP 2015 standards and guidelines for the interpretation of sequence variants, with later ClinGen refinements. **Verify against the current ACMG/AMP recommendations, ClinGen Sequence Variant Interpretation (SVI) Working Group updates, and gene- or disease-specific ClinGen Variant Curation Expert Panel (VCEP) specifications before clinical use.**

This summary is for orientation. Production classification should use the ClinGen Variant Curation Interface (VCI), a validated commercial tool, or an equivalent rule-engine that implements the **current** rule set and the **current** VCEP specifications.

## Contents

- Five-Tier Classification
- Evidence Categories
- Pathogenic Evidence Codes (PVS, PS, PM, PP)
- Benign Evidence Codes (BA, BS, BP)
- Combining Evidence to a Classification
- ClinGen Refinements and VCEPs
- Common Pitfalls
- Reporting Conventions

---

## Five-Tier Classification

| Tier | Abbrev | Meaning (clinical implication) |
|------|--------|-------------------------------|
| Pathogenic | P | Sufficient evidence variant causes disease |
| Likely Pathogenic | LP | Strong but not conclusive evidence |
| Variant of Uncertain Significance | VUS | Insufficient or conflicting evidence |
| Likely Benign | LB | Evidence suggests not disease-causing |
| Benign | B | Sufficient evidence variant is not disease-causing |

LP and LB are explicitly "greater than 90% certainty" tiers; P and B are stronger. VUS is the appropriate label when evidence does not meet the LP / LB thresholds — over-calling VUS as LP is one of the most common errors.

---

## Evidence Categories

Each category combines pieces of evidence about a variant. Categories carry weight (Very Strong, Strong, Moderate, Supporting; analogous on the benign side).

### Pathogenic Side

- **PVS** — Pathogenic Very Strong (one code: PVS1).
- **PS** — Pathogenic Strong (codes PS1–PS4).
- **PM** — Pathogenic Moderate (codes PM1–PM6).
- **PP** — Pathogenic Supporting (codes PP1–PP5).

### Benign Side

- **BA** — Benign Stand-alone (one code: BA1).
- **BS** — Benign Strong (codes BS1–BS4).
- **BP** — Benign Supporting (codes BP1–BP7).

---

## Pathogenic Evidence Codes

The following descriptions are short orientations to the codes in the 2015 framework. Each code has nuanced applicability criteria that ClinGen has refined repeatedly. **Verify the specific applicability against current ACMG/AMP and the relevant VCEP before use.**

### PVS1 — Very Strong

Null variant (nonsense, canonical splice ±1 or ±2, initiation codon loss, single or multi-exon deletion) in a gene where loss-of-function is an established mechanism of disease. ClinGen SVI has published a detailed PVS1 decision tree that adjusts strength based on transcript context, nonsense-mediated decay (NMD), and the position of the truncation. Apply that decision tree, not the headline.

### PS — Strong

- **PS1**: same amino acid change as an established pathogenic variant, regardless of DNA change.
- **PS2**: de novo, with confirmed parentage, in a patient with the disease and a negative family history.
- **PS3**: well-established functional studies show a damaging effect.
- **PS4**: significantly increased prevalence in affected individuals vs. controls.

### PM — Moderate

- **PM1**: located in a mutational hot spot or critical functional domain.
- **PM2**: absent from controls in an appropriately sized population database. ClinGen has downgraded the default strength of PM2 to "Supporting" (PM2_Supporting) for many use cases — verify per VCEP.
- **PM3**: detected in trans with a pathogenic variant for recessive disorders.
- **PM4**: protein length changes (in-frame indels in non-repeat regions, stop-loss).
- **PM5**: novel missense at the same residue where a different missense is established pathogenic.
- **PM6**: assumed de novo without confirmation of paternity/maternity.

### PP — Supporting

- **PP1**: cosegregation with disease in multiple affected family members.
- **PP2**: missense in a gene with low rate of benign missense variation.
- **PP3**: multiple lines of computational evidence support a deleterious effect (current recommendations on computational thresholds and tool choice have evolved — verify).
- **PP4**: patient's phenotype or family history is highly specific for a disease with a single genetic etiology.
- **PP5**: reputable source reports the variant as pathogenic (current recommendations from ClinGen SVI advise against PP5; verify and prefer to cite the primary evidence directly).

---

## Benign Evidence Codes

### BA1 — Stand-alone

Allele frequency above a threshold in a population database (e.g., gnomAD), where the threshold depends on the disease prevalence and inheritance pattern. ClinGen has set gene-specific BA1 thresholds for many genes — verify per VCEP.

### BS — Strong

- **BS1**: allele frequency greater than expected for the disorder.
- **BS2**: observed in a healthy adult with the disease in question (when full penetrance is expected at the age observed).
- **BS3**: well-established functional studies show no damaging effect.
- **BS4**: lack of segregation in affected family members.

### BP — Supporting

- **BP1**: missense in a gene where only loss-of-function causes disease.
- **BP2**: observed in trans with a pathogenic variant for dominant disease, or in cis with a pathogenic variant in any inheritance.
- **BP3**: in-frame indel in a repetitive region without known function.
- **BP4**: multiple lines of computational evidence support a benign effect (same nuance on tool selection as PP3).
- **BP5**: variant found in a case with an alternate explanation.
- **BP6**: reputable source reports as benign (current SVI guidance is the same as for PP5 — verify; prefer primary evidence).
- **BP7**: synonymous (silent) variant with no predicted splicing impact.

---

## Combining Evidence to a Classification

The 2015 paper specifies combinations that yield each tier. Examples (orient only; verify against current rules):

### Pathogenic — any one of:

- 1 Very Strong + 1 Strong, or 1 Very Strong + 2 Moderate, or 1 Very Strong + 1 Moderate + 1 Supporting, or 1 Very Strong + 2 Supporting.
- 2 Strong.
- 1 Strong + 3 Moderate, or 1 Strong + 2 Moderate + 2 Supporting, or 1 Strong + 1 Moderate + 4 Supporting.
- (Various other combinations.)

### Likely Pathogenic — any one of:

- 1 Very Strong + 1 Moderate.
- 1 Strong + 1–2 Moderate.
- 1 Strong + 2 Supporting.
- 3 Moderate.
- 2 Moderate + 2 Supporting.
- 1 Moderate + 4 Supporting.

### Likely Benign:

- 1 Strong + 1 Supporting (benign).
- 2 Supporting (benign).

### Benign:

- 1 Stand-alone.
- 2 Strong (benign).

### VUS:

- Anything not meeting LP/P/LB/B criteria, or conflicting evidence across categories.

ClinGen has since published a Bayesian formalization of these combinations (Tavtigian et al., 2018+) that gives a more principled way to combine evidence; many modern tools use that backbone.

---

## ClinGen Refinements and VCEPs

The 2015 framework is a generic backbone. ClinGen's **Variant Curation Expert Panels** publish gene- or disease-specific specifications that refine how each code applies. Examples (verify currency):

- **Hearing-loss panel** has gene-specific BA1 thresholds and PM2 strength adjustments.
- **Hereditary cancer panels** (BRCA1, BRCA2, ATM, CDH1, MLH1/MSH2/MSH6/PMS2, etc.) have refined PVS1 application, PS1/PM5 use, and functional-evidence calibration.
- **Cardiomyopathy panels** have specifications for cardiac-gene panels.
- **RASopathy panel**, **inherited cardiac arrhythmia panels**, **inherited retinal disorder panels**, and many others.

**Apply the VCEP specification when one exists** for the gene-disease pair you are classifying. The generic 2015 backbone is the fallback when no VCEP exists.

ClinGen also runs the **Sequence Variant Interpretation (SVI) Working Group** which publishes cross-cutting refinements (PVS1 decision tree, PP3/BP4 computational-evidence calibration, PM2 strength, PP5/BP6 discouragement, in-trans rules for recessive). Track the SVI publication list and update tooling accordingly.

---

## Common Pitfalls

- **Over-calling LP from a single moderate plus several weak supporters.** Verify the combination meets the rule.
- **Using PP5 / BP6 from ClinVar without checking the underlying evidence**. Cite primary evidence directly.
- **Applying PVS1 to nonsense in a gene where loss-of-function is not the mechanism.** PVS1 requires LoF as established mechanism.
- **Ignoring NMD escape.** Truncations in the last exon or near the 3' end may escape nonsense-mediated decay and not lead to LoF. The PVS1 decision tree addresses this.
- **Counting computational tools as independent evidence.** PP3/BP4 require multiple independent lines; in practice many tools share inputs and are not independent. Current ClinGen guidance calibrates specific tool combinations.
- **Mixing germline and somatic frameworks.** ACMG/AMP germline does **not** apply to somatic tumor variants — use AMP/CAP/ASCO 2017 tiers for somatic.
- **Not capturing the evidence trail.** A classification without per-code citations is unreviewable. Tools should record each code's evidence and the version of the rule applied.
- **Failing to re-classify on schedule.** ClinVar updates weekly; functional studies appear; VCEPs publish refinements. Plan a documented re-analysis cadence for VUS and any P/LP variants where the evidence may shift.
- **Treating a VUS as actionable.** A VUS is, by definition, of uncertain significance. Clinical management on a VUS alone is not supported by ACMG/AMP and exposes the patient to risk.

---

## Reporting Conventions

A clinical report should carry, per variant:

- Variant name in HGVS with versioned transcript and genome build (see hgvs-nomenclature.md).
- Classification tier (P / LP / VUS / LB / B).
- The evidence codes applied with their strength (e.g., PVS1, PM2_Supporting, PP3) and a short text citation of the evidence underlying each code.
- The rule set version (ACMG/AMP 2015 + applicable SVI refinements + VCEP specification name and version, where used).
- The date of classification and the curator / tool of record.
- Disposition for re-analysis (next review date or trigger conditions).

For ClinVar submission: aim for 3-star expert-panel-reviewed where possible; submit your interpretations with the same evidence detail. Where your interpretation differs from an existing ClinVar entry, engage the relevant VCEP or follow the SVI discrepancy-resolution process.
