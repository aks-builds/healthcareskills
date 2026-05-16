# Pharmacogenomics — CPIC Overview

Reference for the CPIC (Clinical Pharmacogenetics Implementation Consortium) framework and adjacent pharmacogenomic-implementation resources. **Verify every gene-drug pair, CPIC level, and FDA labeling claim against current CPIC guidelines (https://cpicpgx.org), the FDA Table of Pharmacogenomic Biomarkers in Drug Labeling, PharmGKB, and DPWG (Dutch Pharmacogenetics Working Group) before any clinical implementation.** The pharmacogenomic landscape changes regularly as new evidence and FDA labeling decisions accrue.

## Contents

- What CPIC Is
- CPIC Levels
- CPIC Guideline Structure
- Allele and Phenotype Translation
- Frequently Discussed Gene-Drug Pairs (verify)
- Implementation Patterns
- DPWG vs. CPIC
- FDA Labeling and Regulatory Context
- Common Pitfalls

---

## What CPIC Is

CPIC is an international consortium that publishes peer-reviewed clinical practice guidelines that translate genetic test results into prescribing recommendations. Each guideline:

- Reviews the evidence linking a gene (or gene set) to a specific drug.
- Assigns an actionability level.
- Specifies how to translate raw genotypes to **star alleles** and then to **phenotypes** (e.g., Poor Metabolizer, Intermediate Metabolizer, Normal Metabolizer, Rapid Metabolizer, Ultra-rapid Metabolizer, or HLA carrier status).
- Provides phenotype-to-prescribing-recommendation tables, including dose adjustments, alternative therapies, and contraindications.
- Is freely available online and indexed on PharmGKB.

**Important boundary**: CPIC guidelines tell you how to act *if* you have the result. CPIC does not, in general, recommend when to test — that decision is regulatory (FDA labeling), payer-driven, and institutional.

---

## CPIC Levels

CPIC assigns a level to each gene-drug pair based on the evidence supporting clinical actionability of the gene's effect on the drug.

| Level | Meaning |
|-------|---------|
| A | Genetic information should be used to change prescribing of the affected drug. CPIC publishes prescribing recommendations. |
| B | Genetic information could be used to change prescribing because alternative therapies are available, but evidence is moderate. |
| C | There is published evidence supporting the association but no strong evidence that prescribing should change, or no alternative therapy. |
| D | Evidence is weak or limited; no prescribing recommendation. |

A and B pairs are the typical focus for clinical implementation. Verify the **current** level for each pair you care about — levels can change as evidence accumulates.

---

## CPIC Guideline Structure

A typical CPIC guideline contains:

1. **Background**: gene biology, drug mechanism, pharmacokinetics affected.
2. **Evidence review**: studies linking genotype to drug response.
3. **Allele frequency tables**: by major ancestry group.
4. **Genotype-to-phenotype translation table**: which star-allele combinations map to which phenotype.
5. **Phenotype-to-recommendation table**: the actionable output.
6. **Other considerations**: pediatric, pregnancy, special populations.
7. **Limitations and uncertainty**.
8. **Implementation resources**: example EHR alert text, test-ordering language, patient-education templates.

PharmGKB hosts the machine-readable versions of allele definitions, genotype-to-phenotype tables, and dosing tables — useful for building an automated PGx CDS engine.

---

## Allele and Phenotype Translation

Pharmacogenes are usually called as **star alleles** (e.g., CYP2C19 *1, *2, *3, *17), not as individual SNVs. Star alleles are haplotypes — combinations of variants that define functional units.

### Genotype → Star Alleles

Star-allele calling is not the same as variant calling. Use a validated star-allele caller:

- **PharmCAT** (Pharmacogenomics Clinical Annotation Tool) — open-source; consumes a VCF and emits per-gene star-allele calls and CPIC-aligned phenotype + recommendation text.
- **Stargazer** — academic / open-source; supports more genes including CNV-driven calls.
- **Aldy** — academic; star-allele calling with structural variation support.
- Commercial alternatives from clinical labs and EHR-aligned PGx vendors — verify.

Important: CYP2D6 in particular has CNV (duplications, deletions, hybrid alleles) that simple SNV-based callers miss. Confirm the caller handles CYP2D6 CNV before deploying.

### Star Alleles → Phenotype

Each gene has a defined translation. Examples (illustrative — verify against current CPIC tables):

- **CYP2C19**: *1/*1 → Normal Metabolizer; *1/*2 or *1/*3 → Intermediate; *2/*2, *2/*3, *3/*3 → Poor; *1/*17 → Rapid; *17/*17 → Ultra-rapid.
- **CYP2D6**: activity-score system where each star allele has a numeric activity score; sum determines phenotype tier.
- **TPMT** and **NUDT15**: activity tiers with deficient / intermediate / normal phenotypes.

The translation table is gene-specific and is published with each CPIC guideline.

### Phenotype → Recommendation

Phenotype maps to a recommendation in the guideline, e.g., for a Poor Metabolizer of an enzyme that activates a prodrug: avoid the prodrug, use an alternative agent. Recommendations differ by drug, dose, and clinical context.

---

## Frequently Discussed Gene-Drug Pairs

The following pairs appear frequently in implementation discussions. **All entries below are illustrative and must be verified against the current CPIC guideline level and FDA labeling before clinical use.** The table is not exhaustive.

| Gene | Drug or drug class | Therapeutic area | Notes (verify) |
|------|--------------------|------------------|----------------|
| CYP2C19 | Clopidogrel | Cardiology | Reduced active metabolite in poor metabolizers; alternative antiplatelet may be considered. |
| CYP2C19 | Voriconazole | Infectious disease | Dose adjustment for ultra-rapid and poor metabolizers. |
| CYP2C19 | Certain SSRIs (citalopram, escitalopram, sertraline) | Psychiatry | Dose considerations for poor / ultra-rapid metabolizers. |
| CYP2D6 | Codeine, tramadol | Pain | Ultra-rapid metabolizers convert prodrug to opioid too efficiently; poor metabolizers get inadequate analgesia. |
| CYP2D6 | Tamoxifen | Oncology (breast) | Active-metabolite formation impaired in poor metabolizers; clinical relevance debated. |
| CYP2D6 | Certain antidepressants (tricyclics, some SSRIs) | Psychiatry | Dose considerations across phenotypes. |
| HLA-B*57:01 | Abacavir | HIV | Hypersensitivity risk in carriers; testing standard of care before initiation. |
| HLA-B*15:02 | Carbamazepine, oxcarbazepine | Neurology / psychiatry | Stevens-Johnson syndrome / TEN risk; particularly relevant in patients of Han Chinese, Thai, and certain other Asian ancestries. |
| HLA-B*58:01 | Allopurinol | Rheumatology | Severe cutaneous adverse reactions in carriers. |
| TPMT, NUDT15 | Thiopurines (azathioprine, mercaptopurine, thioguanine) | Oncology, GI, rheumatology | Myelotoxicity risk in deficient / intermediate metabolizers. |
| DPYD | 5-fluorouracil, capecitabine | Oncology | Severe / fatal toxicity in deficient patients. EMA and FDA have updated labeling — verify current scope. |
| UGT1A1 | Irinotecan | Oncology | Neutropenia / diarrhea risk in patients with reduced-function alleles. |
| CYP2C9, VKORC1 | Warfarin | Cardiology / hematology | Dose-initiation algorithms incorporate genotype; INR monitoring still required. |
| CYP3A5 | Tacrolimus | Transplant | Expressers require higher doses to achieve target trough. |
| SLCO1B1 | Simvastatin (and other statins to varying degrees) | Cardiology | Myopathy risk in reduced-function carriers. |
| CFTR | Ivacaftor (and combination CFTR modulators) | Cystic fibrosis | Genotype-restricted indication. |
| G6PD | Rasburicase, dapsone, certain antimalarials | Oncology, infectious disease | Hemolysis risk in deficient patients. |

**Do not** quote a CPIC level or a dosing recommendation for any of these pairs without verifying against the current CPIC guideline. Likewise, do not quote an FDA label without checking the current product labeling.

---

## Implementation Patterns

### Reactive PGx

PGx test is ordered when a drug is being considered. Slow path: result arrives after prescribing decision is being made. Common but suboptimal.

### Pre-emptive PGx

PGx panel run on patients before any of the included drugs is prescribed. Results stored discretely in the EHR. CDS fires at the moment a relevant drug is ordered. Drives more value because the result is available exactly when needed and is amortized across many possible drug decisions.

### CDS Hooks

Surface PGx alerts using the HL7 CDS Hooks specification — typically `order-sign` and `medication-prescribe` (and sometimes `order-select`) hooks fire when a relevant drug is being ordered. The hook service queries the patient's stored PGx phenotype and returns a card with the recommendation, alternative drug suggestions, and a link to the source CPIC guideline.

Tune to avoid alert fatigue:

- Suppress alerts for routes / doses where the genotype impact is minimal.
- Suppress alerts when the alternative therapy is contraindicated for this patient.
- Allow "acknowledge and proceed" with reason capture for retrospective audit.

See `clinical-decision-support`.

### Storage in the EHR

Store at the **phenotype** level (e.g., "CYP2C19 Intermediate Metabolizer") as a discrete observation, not just as a free-text note. Many EHRs have a dedicated PGx panel area; if not, use a `DiagnosticReport` with `Observation` per gene per the FHIR Genomics Reporting IG.

The raw genotype / star-allele call should be retained for re-interpretation, but **clinicians act on the phenotype**, not the raw alleles.

### Patient Education

PGx results have implications for future prescribing across the patient's lifetime. Patient-portal disclosure language, primary-care follow-up, and patient-facing summary documents all matter. Coordinate with the EHR portal team.

---

## DPWG vs. CPIC

The **Dutch Pharmacogenetics Working Group (DPWG)** publishes prescribing guidance in parallel to CPIC, more commonly used in Europe. Some gene-drug pairs are covered by both bodies and the recommendations are mostly aligned; in a few cases they differ on dose adjustment or strength of recommendation.

For institutions operating in both US and EU contexts, pick a primary guideline body and document where you defer to the other.

---

## FDA Labeling and Regulatory Context

The **FDA Table of Pharmacogenomic Biomarkers in Drug Labeling** lists drugs with pharmacogenomic information in their labeling. Categories within the table indicate the strength of the labeling claim (testing recommended, required, actionable, informative).

- A CPIC Level A recommendation does not necessarily correspond to an FDA "testing required" label.
- Some drugs have FDA-required testing language with limited CPIC actionability content, and vice versa.
- Verify both the CPIC guideline and the **current** FDA labeling before making clinical or system-design decisions tied to either source.

International regulators (EMA, MHRA, Health Canada, PMDA) have their own labels — for global products, the union of regulatory positions matters.

---

## Common Pitfalls

- **Surfacing raw genotype to clinicians.** "CYP2C19 *1/*2" is not actionable without phenotype translation. Always translate before display.
- **Using SNV-only callers for CYP2D6.** CYP2D6 CNV is common and clinically relevant; SNV-only calling miscalls phenotype frequently.
- **Ignoring ancestry-specific allele frequencies.** Some alleles are common in one population and rare in another; allele coverage of the assay must match the patient population.
- **Hard-coding recommendations into the EHR.** CPIC tables change. Reference the table; do not bake it into application code.
- **Treating PGx as one-time.** A new drug ordered tomorrow may trigger a new alert from the same stored phenotype. Pre-emptive PGx amortizes across all future relevant orders.
- **Alert fatigue.** Overly broad firing trains clinicians to dismiss alerts. Tune by drug, dose, route, indication.
- **Inconsistent storage.** Phenotype stored in one place, star alleles in another, free-text note in a third — pick one canonical location with discrete codes (LOINC / FHIR Genomics Reporting profiles) and a documented sync to any secondary location.
- **Not addressing insurance implications.** PGx results, like other genetic information, are covered by GINA for health insurance and employment but not life / disability / long-term-care insurance — patients should be informed before testing.

For implementation, pair this overview with the current CPIC guideline for each gene-drug pair you intend to act on, the FDA label for each drug, and the PharmGKB machine-readable tables for codifying the translation engine.
