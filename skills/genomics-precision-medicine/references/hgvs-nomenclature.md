# HGVS Nomenclature

Reference for the Human Genome Variation Society (HGVS) naming standard for sequence variants. Verify the latest recommendations at https://hgvs-nomenclature.org/.

## Contents

- Why a Standard
- Reference Sequence Identifiers
- Prefix Letters (Sequence Type)
- DNA Variant Syntax
- Protein Variant Syntax
- RNA Variant Syntax
- Mitochondrial Variants
- Common Pitfalls
- Worked Examples

---

## Why a Standard

The same biological variant can be written many ways:

- "BRCA1 5382insC" (legacy, ambiguous transcript)
- "c.5266dupC" (HGVS, but transcript unstated)
- "NM_007294.4:c.5266dup" (HGVS, transcript and version pinned)
- "NC_000017.11:g.43057062dup" (HGVS, genomic on GRCh38)
- a GA4GH VRS canonical identifier (hash-based, build-aware)

A clinical report that lists "BRCA1 5382insC" can be misinterpreted across labs and across genome builds. HGVS provides an unambiguous, machine-parseable name when used correctly — meaning **with a versioned reference sequence**.

---

## Reference Sequence Identifiers

Every HGVS name needs a reference sequence with version.

| Prefix | Source | Example | Notes |
|--------|--------|---------|-------|
| NM_ | RefSeq mRNA | NM_007294.4 | Versioned curated mRNA. Use for `c.` and `r.`. |
| NR_ | RefSeq non-coding RNA | NR_046018.2 | For non-coding transcripts. |
| NP_ | RefSeq protein | NP_009225.1 | For `p.` (protein) variants. |
| NC_ | RefSeq chromosome | NC_000017.11 (chr17, GRCh38) | For `g.` (genomic) variants. Different for GRCh37 vs GRCh38. |
| NG_ | RefSeq gene region | NG_005905.2 | Whole gene region with introns; for `g.` relative to the gene. |
| ENST | Ensembl transcript | ENST00000357654.9 | Ensembl alternative to NM_; pin the version. |
| ENSP | Ensembl protein | ENSP00000350283.4 | Ensembl protein. |
| LRG | Locus Reference Genomic | LRG_292t1 | Stable per-gene reference designed for clinical reporting; recommended where available. |

The **version suffix** (`.4`, `.11`, etc.) is part of the identifier. `NM_007294` without a version is ambiguous and should be rejected for clinical reporting.

For genomic coordinates always carry the assembly identifier separately (GRCh37 / GRCh38 / T2T-CHM13). `NC_000017.11` is GRCh38; `NC_000017.10` is GRCh37. The version conveys the assembly indirectly but most reports also state the assembly explicitly to reduce confusion.

---

## Prefix Letters (Sequence Type)

| Letter | Meaning | Example |
|--------|---------|---------|
| `c.` | Coding DNA — numbered from the A of the ATG start codon, with negative numbers for 5' UTR and "+/-" offsets for intronic positions | `c.35G>A`, `c.-10G>A`, `c.123+1G>T` |
| `g.` | Genomic — numbered along the chromosome (or `NG_` region) | `NC_000017.11:g.43057062dup` |
| `m.` | Mitochondrial | `m.3243A>G` |
| `r.` | RNA — same numbering as `c.` but lowercase bases | `r.35g>a` |
| `p.` | Protein | `p.(Gly12Asp)`, `p.G12D` short form |
| `n.` | Non-coding RNA | `n.123A>G` |
| `o.` | Other (rare) | |

Pick the right prefix for the level of evidence:

- DNA evidence (sequencing reads): use `c.` or `g.`.
- Inferred protein change from DNA: predicted, parenthesize: `p.(Gly12Asp)`.
- Direct protein evidence (mass spec, etc.): unparenthesized: `p.Gly12Asp`.

---

## DNA Variant Syntax

### Substitutions

`reference_position>alternate`

- `c.35G>A` — at coding position 35, G is substituted by A.
- `g.43057062C>T` — genomic substitution.

### Deletions

`reference_position_position?del` (with optional deleted sequence)

- `c.76del` (single base) — del syntax without the deleted base is preferred in current recommendations.
- `c.76_78del` (multi-base).
- `c.76delG` (older form; current recommendations prefer `c.76del`).

### Duplications

- `c.76dup` (single base).
- `c.76_78dup` (multi-base).
- `c.5266dup` (BRCA1 example; older literature wrote "5382insC" — same variant, different naming).

### Insertions

- `c.76_77insT` — insertion between positions 76 and 77.
- Always specify the two flanking positions.

### Indels (delins)

When a stretch is deleted and replaced:

- `c.76_78delinsGAA` — delete 76–78, insert GAA.
- `c.76_77delinsTT` — same as a multi-base substitution; HGVS prefers delins notation.

### Intronic Positions

Within `c.` numbering, intronic positions use `+` (after exon end) or `-` (before exon start):

- `c.123+1G>T` — splice donor +1, the conserved GT.
- `c.124-1G>A` — splice acceptor -1, the conserved AG.

### Inversions

- `c.76_78inv` — invert positions 76 through 78.

### Repeats

- `c.123_125[20]` — 20 copies of the trinucleotide at positions 123–125.

---

## Protein Variant Syntax

Protein names use three-letter amino acid codes by current recommendation, though one-letter forms are widely used in literature.

### Substitutions

- `p.(Gly12Asp)` — predicted from DNA, parenthesized.
- `p.Gly12Asp` — observed protein change.
- `p.G12D` — short form, widely used in oncology literature.

### Stop Codons

- `p.(Gln45*)` or `p.(Gln45Ter)` — substitution to stop.
- Star (`*`) and `Ter` are equivalent; `*` is more common in informatics.

### Deletions, Duplications, Insertions

- `p.(Lys100del)` — single residue deletion.
- `p.(Lys100_Asn102del)` — range deletion.
- `p.(Lys100dup)` — single residue duplication.
- `p.(Lys100_Ala101insGlyThr)` — insertion of GlyThr between Lys100 and Ala101.

### Frameshifts

- `p.(Arg97Profs*23)` — frameshift starting at Arg97, replaced by Pro, with a new stop 23 codons later.

### Extension (loss-of-stop)

- `p.(*807Argext*?)` — termination codon replaced by Arg, extending the protein by an unknown number of residues.

### No Protein Effect

- `p.(=)` — synonymous, no predicted amino acid change.
- `p.0` — no protein produced (e.g., promoter loss).

---

## RNA Variant Syntax

RNA uses lowercase, same numbering as `c.`:

- `r.35g>a` — RNA substitution (lowercase).
- `r.123_124insu` — RNA insertion of uracil.
- `r.(?)` — predicted RNA effect unknown.
- `r.spl?` — predicted splicing effect, details unknown.

RNA names are usually predicted from DNA + a splicing model (SpliceAI, MaxEntScan, in vitro assays). Use parentheses on the `r.` predicted form if not directly observed.

---

## Mitochondrial Variants

- `m.3243A>G` — mitochondrial substitution; uses `m.` and chromosome NC_012920.1.
- Heteroplasmy is reported separately (percentage), not in the HGVS name itself.

---

## Common Pitfalls

### Missing Transcript Version

`NM_007294:c.5266dup` is ambiguous. `NM_007294.4:c.5266dup` is unambiguous. Reject the former in clinical reports.

### Wrong Transcript Picked

A gene with multiple clinically relevant transcripts (e.g., BRCA1, MLH1, NF1) gives different `c.` numbering on different transcripts. Pick the **lab's canonical clinical transcript** consistently. The MANE Select / MANE Plus Clinical set from NCBI / EBI publishes recommended canonical transcripts where available.

### Mixing Builds

`NC_000017.10` is GRCh37 chromosome 17. `NC_000017.11` is GRCh38 chromosome 17. Same gene region, **different coordinates**. Carrying the assembly on every variant artifact prevents silent build mixing.

### Legacy "V600E"-Style Names

BRAF V600E without a transcript is ambiguous (the canonical clinical transcript NM_004333.x with version is required to disambiguate). For literature search the legacy name is fine; for clinical reporting it is not.

### Old Forms vs. Current Recommendations

HGVS recommendations evolve. Older forms (`c.76delG`, `c.5382insC`, `IVS1+1G>T`) are widely understood but the current canonical forms (`c.76del`, `c.5266dup`, `c.123+1G>T`) are preferred. Validate with a tool that implements the current spec (Mutalyzer, VariantValidator, hgvs Python library).

### Predicted vs. Observed Protein

Use parentheses for predicted protein consequences from DNA evidence: `p.(Gly12Asp)`. Drop parentheses only when the protein change was directly observed (proteomics).

### Indels Misclassified

A two-base substitution at adjacent positions is a `delins`, not two separate substitutions. Variant callers and annotators must left-normalize and properly classify before HGVS names are generated.

### One-Letter vs Three-Letter

Choose one and be consistent in a report. `p.G12D` (one-letter) is dominant in cancer literature; `p.(Gly12Asp)` (three-letter) is the HGVS-preferred form for clinical reports.

---

## Worked Examples

The following are illustrative format examples. **Verify transcripts, versions, and coordinates against the current source of truth (RefSeq / MANE / ClinVar) before any clinical use.**

### Example 1 — Coding SNV

Variant: a missense in KRAS at codon 12 changing glycine to aspartate.

- DNA (coding): `NM_004985.5:c.35G>A`
- DNA (genomic, GRCh38): `NC_000012.12:g.25245350C>T`
- Protein (predicted): `NM_004985.5(KRAS):p.(Gly12Asp)`
- Short form: `p.G12D`

Note the genomic ALT differs from the coding ALT because KRAS is on the minus strand.

### Example 2 — Frameshift Deletion

Variant: a single-base deletion causing frameshift.

- Coding: `NM_007294.4:c.5266del`
- Protein: `NM_007294.4(BRCA1):p.(Gln1756Profs*74)` — illustrative; verify against the canonical transcript and current annotation.

### Example 3 — Splice-Site

- Coding: `NM_000218.3:c.123+1G>T`
- RNA (predicted): `r.spl?`
- Protein (predicted, unknown): `p.?`

### Example 4 — Intragenic Duplication

- Coding: `NM_004006.3:c.500_700dup` — duplication of 201 bases inside DMD.
- Protein: depends on whether the duplication preserves frame.

### Example 5 — Mitochondrial

- Mitochondrial: `m.3243A>G` (MELAS-associated; verify clinical interpretation before reporting).

---

## Validation Tools

- **Mutalyzer** (https://mutalyzer.nl) — HGVS name validation and normalization.
- **VariantValidator** (https://variantvalidator.org) — validation against current RefSeq / Ensembl / LRG.
- **hgvs Python library** — programmatic parsing and validation.
- **biocommons / hgvs** — open-source library with current grammar.
- **MANE Select** — NCBI/EBI canonical-transcript set.

For any clinical reporting pipeline: validate every generated name with one of these tools, fail loudly on rejected names, and store the validation result in the audit trail.
