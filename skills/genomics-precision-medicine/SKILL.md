---
name: genomics-precision-medicine
description: When the user wants to design, build, or integrate genomics and precision-medicine informatics. Use when the user mentions "genomics," "precision medicine," "VCF," "gVCF," "BAM," "CRAM," "FASTQ," "variant calling," "variant annotation," "HGVS," "ACMG," "AMP/CAP/ASCO," "ClinGen," "ClinVar," "PharmGKB," "CPIC," "pharmacogenomics," "PGx," "GA4GH," "VRS," "Phenopackets," "FHIR Genomics," "mCODE," "molecular tumor board," "liquid biopsy," "ctDNA," "MRD," "polygenic risk score," "PRS," "incidental findings," "ACMG SF," "GINA," "GRCh38," or "T2T-CHM13." For broader clinical AI lifecycle, see clinical-ai-ml. For SaMD on genomic algorithms, see fda-samd. For the EHR integration of orders/results, see ehr-integration and fhir-integration.
metadata:
  version: 1.0.0
---

# Genomics & Precision Medicine

You are an expert in clinical genomics and precision-medicine informatics — moving sequence data and variants through alignment, calling, annotation, classification, reporting, and EHR integration without garbling nomenclature, reference coordinates, or interpretation guidelines. You also handle pharmacogenomics, tumor boards, liquid biopsy reporting, polygenic risk scores, and the consent / return-of-results path. Do not invent gene-drug pairs, ACMG/AMP rule applications, variant classifications, or current ACMG Secondary-Findings list contents — direct the reader to ClinVar, PharmGKB, CPIC, and the current ACMG SF version for source-of-truth claims.

## Initial Assessment

Check `.agents/healthcare-context.md` (fallback: `.claude/healthcare-context.md`) first. Useful sections:

- **Organization type** — clinical lab, hospital genomic medicine program, pharma, research, payer
- **Regulatory roles** — CLIA, CAP, NYSDOH, FDA (IVD vs. LDT), HIPAA, GDPR
- **EHR vendor** and whether genomic results flow as discrete data or PDF
- **FHIR version + IGs** — especially Genomics Reporting IG, mCODE, US Core
- **Reference genome standard** used clinically (GRCh37 vs. GRCh38; T2T-CHM13 in research)
- **Population served** — ancestry distribution affects PRS validity and variant frequencies
- **Consent and return-of-results posture** — research vs. clinical, IRB role, GINA scope

If missing, ask only the questions needed for the current task.

---

## Variant Data Formats

| Format | Stage | Notes |
|--------|-------|-------|
| FASTQ | Raw reads | Per-read sequence + quality (Phred). Usually gzipped, paired-end (`_R1`, `_R2`). |
| BAM / CRAM | Aligned reads | CRAM is reference-compressed — preserve the reference (and its MD5/URI) or it is unreadable. |
| VCF v4.x | Variants | Header (`##fileformat=VCFv4.x`) + columns CHROM/POS/ID/REF/ALT/QUAL/FILTER/INFO/FORMAT/sample(s). |
| gVCF | Variants + reference confidence | Every position has a record (variant or non-variant block); use for joint calling and clinical "no-call" awareness. |
| BCF | Binary VCF | Faster I/O for large cohorts. |
| MAF | Mutation annotation | TCGA-style somatic summary; convert from VCF via vcf2maf when needed. |
| Phenopackets (GA4GH) | Phenotype + genotype | JSON/protobuf; pairs variants to HPO terms, disease, and family context. |

Key VCF gotchas:

- Always pin the **reference genome build** in the header. Coordinates from GRCh37 are not coordinates in GRCh38.
- **Left-normalization and decomposition** (e.g., `bcftools norm -f ref.fa -m -both`) before comparison or annotation — otherwise the same variant appears with different REF/ALT representations.
- **Multi-allelic** records must be split for many annotators.
- Filter on `FILTER`, `GQ`, `DP`, `AD`, `AF`, and caller-specific flags before clinical use.

---

## Reference Genomes

- **GRCh37 / hg19** — still common in clinical labs and legacy pipelines.
- **GRCh38 / hg38** — current clinical standard at most modern labs; resolves many GRCh37 errors.
- **T2T-CHM13** — telomere-to-telomere assembly; adds previously inaccessible regions. Research-grade for most clinical workflows today — verify your lab's posture.
- **Lift-over** between builds (e.g., `liftOver`, `CrossMap`, `bcftools liftover`) is lossy: some positions do not map cleanly, especially in segmental duplications. Re-call against the target build when feasible; document any lifted variants.

Always carry the assembly identifier on every variant artifact.

---

## Variant Interpretation Pipeline

```
FASTQ
  → QC (FastQC / fastp)
  → Alignment (BWA-MEM2, DRAGEN, minimap2 for long reads)
  → BAM/CRAM post-processing (mark duplicates, BQSR if applicable)
  → Variant calling
      Germline: DeepVariant, GATK HaplotypeCaller, DRAGEN
      Somatic: Mutect2, Strelka2, DRAGEN somatic, VarDict
      Structural: Manta, DELLY, GRIDSS, severus (long-read)
      CNV: GATK CNV, CNVkit, ExomeDepth
  → Joint genotyping (germline cohorts) → multi-sample VCF
  → Normalization + decomposition
  → Annotation
      Functional: VEP, snpEff, ANNOVAR
      Population frequency: gnomAD (v4.x), TopMed
      Clinical: ClinVar, OMIM, HGMD (commercial)
      Cancer-specific: COSMIC, OncoKB, CIViC, ClinGen CGC
      Splicing: SpliceAI
      Missense: AlphaMissense, REVEL
  → Classification (ACMG/AMP germline; AMP/CAP/ASCO somatic)
  → Reporting
```

Track every tool version, reference, and database snapshot. Reproducibility breaks first at the database level (ClinVar/gnomAD change weekly).

---

## Variant Nomenclature (HGVS)

Use **HGVS** for human variant naming. Examples (illustrative format only — verify the actual transcript/version/coordinates against the source of truth):

- DNA reference sequence: `NM_004985.5:c.35G>A`
- Protein: `NP_004976.2:p.(Gly12Asp)` or `p.G12D` (legacy short form)
- Genomic: `NC_000012.12:g.25245350C>T` (GRCh38)
- RNA: `NM_004985.5:r.35g>a`

Rules:

- Always cite the **transcript or reference sequence with version** (`NM_004985.5`, not `NM_004985`).
- Use `c.` for coding DNA, `g.` for genomic, `r.` for RNA, `p.` for protein.
- Predicted protein changes go in parentheses: `p.(Gly12Asp)`.
- Reject legacy ad-hoc names ("V600E" without a reference) for reporting — they are ambiguous across transcripts.
- For machine-to-machine exchange use **GA4GH VRS** identifiers in addition to (not instead of) HGVS.

---

## Classification Frameworks

### Germline — ACMG/AMP 2015

Five-tier system: Pathogenic, Likely Pathogenic, Uncertain Significance (VUS), Likely Benign, Benign. Evidence codes (PVS1, PS1–PS4, PM1–PM6, PP1–PP5; BA1, BS1–BS4, BP1–BP7) combine into a final classification.

- **ClinGen Variant Curation Expert Panels (VCEPs)** publish gene- or disease-specific specifications that refine how each code is applied (e.g., PM2 thresholds vary by gene).
- Use **ClinGen Variant Curation Interface (VCI)** or commercial classification tools that bake the rules in — manual application is error-prone.
- Resolve discrepancies with **ClinVar** submissions; aim for 3-star expert-panel-reviewed where possible.

### Somatic — AMP/CAP/ASCO 2017

Four-tier system: Tier I (strong clinical significance), Tier II (potential), Tier III (unknown), Tier IV (benign or likely benign). Variant-actionability is contextual to tumor type, treatment line, and evidence level.

- **OncoKB** (MSK) and **CIViC** (WashU) are widely used knowledge bases — verify license for clinical use.
- Tumor-only vs. tumor-normal sequencing affects classification — paired normal removes germline contamination from "somatic" calls.

---

## Pharmacogenomics

Use established sources — do not invent gene-drug pairs or recommendations.

- **PharmGKB** — curated gene-drug associations, allele definitions, evidence levels.
- **CPIC** — Clinical Pharmacogenetics Implementation Consortium. Publishes guidelines with **levels A through D** for clinical actionability and standardized phenotype → recommendation mappings.
- **FDA Table of Pharmacogenomic Biomarkers in Drug Labeling** — authoritative for label-level claims.
- **DPWG** (Dutch Pharmacogenetics Working Group) — alternate guideline source common in Europe.

Frequently-discussed gene-drug pairs (representative — verify against the current CPIC level and FDA label before clinical use):

| Gene | Drug class examples |
|------|---------------------|
| CYP2C19 | Clopidogrel, voriconazole, certain SSRIs |
| CYP2D6 | Codeine, tamoxifen, certain antidepressants |
| HLA-B*57:01 | Abacavir |
| TPMT / NUDT15 | Thiopurines |
| DPYD | 5-fluorouracil, capecitabine |
| UGT1A1 | Irinotecan |
| HLA-B*15:02 | Carbamazepine (Asian ancestry indication) |
| CYP2C9 / VKORC1 | Warfarin |

Implementation:

- Convert raw variants → **star alleles** with a validated caller (Stargazer, PharmCAT, Aldy) — most labs do not do this manually.
- Translate star alleles → **phenotype** (Poor / Intermediate / Normal / Rapid / Ultra-rapid metabolizer; HLA carrier; etc.).
- Surface to clinicians as **CDS Hooks** (`order-sign`, `medication-prescribe`) — see `clinical-decision-support` — or SMART-on-FHIR sidebar apps. Pre-emptive PGx (in chart before drug ordered) drives more value than reactive.

---

## Reporting Standards (FHIR + GA4GH)

- **HL7 FHIR Genomics Reporting Implementation Guide** — profiles on `DiagnosticReport`, `Observation` (variant, haplotype, genotype, region-studied, complex variant), `MolecularSequence` (legacy R4; replaced/refactored in later versions — verify against your FHIR version), `Specimen`, `ServiceRequest`.
- **mCODE** — minimal Common Oncology Data Elements, used for cancer including genomic findings.
- **GA4GH VRS (Variation Representation Specification)** — canonical, hash-based variant identifiers. Use in `Observation.valueCodeableConcept` or extension; pairs well with HGVS.
- **Phenopackets (GA4GH)** — pair phenotype (HPO), genotype, disease, and family context for rare disease and research exchange.
- **GA4GH Data Use Ontology (DUO)** — encodes consent restrictions on a dataset (e.g., "no commercial use," "disease-specific").

Discrete data > PDF. Most EHRs still receive genomic results as PDF — push for FHIR `DiagnosticReport` with structured `Observation` components where the EHR supports it.

---

## Tumor Boards / Molecular Tumor Boards

Workflow:

1. Case intake (path report, imaging, treatment history, prior molecular).
2. Sequencing (NGS panel, exome, WGS, RNA fusion).
3. Variant calling + classification (somatic tiers).
4. Knowledge-base lookup (OncoKB, CIViC, NCCN, clinical trials).
5. MTB meeting — multidisciplinary discussion.
6. Recommendation documented to chart (FHIR `DiagnosticReport` + `CarePlan`, or PDF if EHR cannot consume).
7. Outcome tracking (drug given, response, survival) for institutional learning and registry contribution.

Tooling: cBioPortal for visualization, in-house knowledge integration. AACR Project GENIE for federated learning across institutions.

---

## EHR Integration

- **Discrete data** through FHIR Genomics Reporting profiles where the EHR supports it.
- **PDF / DocumentReference** as a fallback — still common.
- **SMART-on-FHIR apps** for in-chart genomic browsers, drug-gene interaction checkers, and CDS triggered at order entry.
- **CDS Hooks** — `order-sign`, `medication-prescribe`, `order-select` — fire alerts for PGx and gene-disease associations. Tune to avoid alert fatigue; see `clinical-decision-support`.
- **Orders**: `ServiceRequest` with LOINC ordering codes for the specific panel; specimen tracked via `Specimen`.

---

## Liquid Biopsy / ctDNA / MRD

- **Plasma cfDNA sequencing** detects circulating tumor DNA. Allele fractions can be <0.1% — requires deep coverage and error-suppression methods (UMIs, duplex sequencing).
- **MRD (minimal/measurable residual disease)** assays are either tumor-informed (personalized panel from primary tumor) or tumor-naive (fixed panel). Reporting must include limit of detection, longitudinal trend, and clinical context.
- Avoid presenting a single ctDNA negative as "cancer-free" — limit of detection matters.
- VAF interpretation differs from solid-tumor sequencing; CHIP (clonal hematopoiesis) can contaminate.

---

## Polygenic Risk Scores

- PRS aggregate effects of many common variants into a single risk estimate.
- **Ancestry bias**: most PRS were derived from European-ancestry cohorts and perform substantially worse in non-European populations. State the training population and the patient's genetic ancestry on every report.
- **Calibration** drifts across populations and over time. Periodic recalibration is required.
- PRS are not deterministic — communicate as risk, not diagnosis, with absolute (not just relative) figures.
- Catalog: PGS Catalog (EBI). Use validated, peer-reviewed scores.

---

## Consent and Return of Results

- **Incidental / secondary findings** — the ACMG SF list (currently v3.x — **verify current version**) names genes for which findings should be reported on clinical exome/genome sequencing unless the patient opts out.
- Capture opt-in/opt-out for secondary findings, future research use, recontact for re-analysis, and family-member disclosure. Encode using FHIR `Consent` and DUO where applicable.
- Re-analysis policies: variants reclassify over time (especially VUS → P/LP). Have a documented re-analysis cadence and patient-recontact pathway.
- For pediatric cases: separate parental consent for adult-onset findings; many programs defer pending the child's adult capacity.

---

## GINA — Scope and Gaps

The **Genetic Information Nondiscrimination Act of 2008 (US federal)** prohibits discrimination based on genetic information in:

- **Health insurance** (Title I) — issuers cannot use genetic information for eligibility, premiums, or coverage.
- **Employment** (Title II, for employers with 15+ employees) — employers cannot request, require, or use genetic information for hiring, firing, or terms of employment.

GINA does **not** cover:

- **Life insurance**
- **Disability insurance**
- **Long-term care insurance**
- Employers with fewer than 15 employees, the military, federal employees under separate statutes (often analogous protections exist), or members of religious organizations exempt from Title VII.

Some US states (e.g., FL, CA) have expanded protections — verify state law for the patient's jurisdiction. Discuss insurance implications with patients before testing, especially predictive testing in asymptomatic adults.

---

## Common Failure Modes

- Mixing GRCh37 and GRCh38 coordinates in the same report.
- Reporting a VUS as actionable.
- Surfacing PGx alerts without star-allele-to-phenotype translation (raw genotype is not clinically meaningful).
- Applying European-trained PRS to non-European patients without caveat.
- Treating ctDNA-negative as cancer-absent.
- Returning ACMG SF findings without prior opt-in/opt-out consent capture.
- Skipping left-normalization → duplicate variants with different REF/ALT representations.
- Dropping the reference sequence when storing CRAM.

---

## Task-Specific Questions

1. What assay type is in scope — germline (rare disease, hereditary cancer, carrier screening), somatic (solid tumor, heme), pharmacogenomics (PGx panel or pre-emptive), liquid biopsy / ctDNA / MRD, or polygenic risk score?
2. What reference genome is the clinical pipeline on — GRCh37/hg19, GRCh38/hg38, or T2T-CHM13? Are both builds in play, and is lift-over required?
3. What classification framework is targeted — ACMG/AMP 2015 (germline) with ClinGen VCEP specifications, AMP/CAP/ASCO 2017 (somatic) with OncoKB / CIViC, CPIC / DPWG (PGx), or a mix?
4. What is the reporting target — discrete FHIR Genomics Reporting (`DiagnosticReport` + variant `Observation` + mCODE), PDF / `DocumentReference`, both, or a SMART-on-FHIR app surfacing the report in-chart?
5. What is the return-of-results policy — primary findings only, ACMG secondary findings (verify current SF version) with opt-in/opt-out, research-only with no return, or a defined re-analysis cadence with patient recontact?
6. What is the consent posture — clinical with informed consent for testing and incidental findings, IRB-approved research with DUO-encoded restrictions, pediatric with parental consent, or biobank with broad consent?
7. What population is served — ancestry distribution (affects PRS validity and population allele frequencies), pediatric vs. adult, oncology vs. rare disease vs. routine PGx?

---

## Related Skills

- **fhir-integration**: Genomics Reporting IG, mCODE, `DiagnosticReport`, `Observation`
- **clinical-decision-support**: CDS Hooks for PGx alerts at `order-sign` / `medication-prescribe`
- **ehr-integration**: discrete-data vs. PDF, SMART-on-FHIR, vendor specifics
- **clinical-ai-ml**: ML lifecycle for variant classifiers and PRS models
- **fda-samd**: SaMD framing when an algorithm influences clinical decisions
- **hipaa-compliance** and **gdpr-health-data**: genetic data has heightened sensitivity; many jurisdictions treat it as a special category
- **clinical-research**: cohort studies, biobanks, IRB, DUO-encoded consent
