# De-identification Techniques — When to Use Each

A practical comparison of the methods that surface in healthcare de-identification. Use this to pick the right approach for the data and the recipient. The legal/regulatory framing should be confirmed with your privacy/compliance counsel; statistical techniques should be reviewed by a qualified statistician (or an "expert" for HIPAA Expert Determination purposes).

## At a Glance

| Method | Legal effect under HIPAA | Effort | When it fits |
|---|---|---|---|
| Safe Harbor | Out of HIPAA scope (with no-actual-knowledge) | Low — mechanical | Many fields can be lost; recipient doesn't need granular dates/geo |
| Expert Determination | Out of HIPAA scope | High — requires qualified expert | Need to retain dates, geographic detail, or rare attributes |
| Limited Data Set + DUA | Still PHI; permitted use under DUA | Medium | Research, public health, healthcare operations; need dates and limited geo |
| Pseudonymization | Still PHI / personal data | Low-medium | Security control, blast-radius reduction — not a regulatory off-switch |
| k-anonymity / l-diversity / t-closeness | Statistical control, often used inside Expert Determination | Medium-high | Aggregate releases, public-use tables |
| Differential privacy | Statistical guarantee with bounded privacy budget | High | Releases where adversary may have arbitrary side information |
| Synthetic data | Depends on the synthesis method's privacy guarantees | High | Internal ML pipelines, sharing with limited privacy guarantees |

## Safe Harbor (HIPAA §164.514 Safe Harbor method)

- Remove all 18 identifier categories (see safe-harbor-identifiers.md)
- No actual knowledge that the residual data could re-identify
- Result: not PHI

**Strengths**: Mechanical, auditable, no statistician needed.
**Weaknesses**: Loses dates of service (only year retained), loses ZIP detail, loses ages over 89. Often too lossy for episode analytics, cohort studies, longitudinal trajectories.

## Expert Determination (HIPAA §164.514 Expert Determination method)

- A qualified person with appropriate knowledge of and experience with generally accepted statistical and scientific principles and methods determines that the risk is "very small" that the information could be used, alone or in combination with other reasonably available information, to identify an individual
- The expert documents methods and results
- Result: not PHI

**Strengths**: Can retain useful detail (dates, geography, rare fields) by quantifying and controlling re-identification risk.
**Weaknesses**: Requires access to qualified expertise, documentation discipline, re-assessment on dataset change.

## Limited Data Set + DUA (HIPAA §164.514(e))

A Limited Data Set removes 16 of the 18 identifiers but may retain:
- Dates (admission, discharge, service, birth)
- Limited geographic info (city, state, ZIP, but not street or fully-identifying combinations)

It is still PHI but may be disclosed for research, public health, or healthcare operations under a Data Use Agreement that binds the recipient. The DUA includes:
- Permitted uses and disclosures
- Safeguards
- Reporting of impermissible uses
- Prohibition on re-identification or contacting individuals

**Strengths**: Keeps dates and limited geography. Good fit for academic research with a clear DUA.
**Weaknesses**: Still PHI. Recipient is bound by DUA; CE/BA still has obligations.

## Pseudonymization / Tokenization

Replace direct identifiers with a random token. The mapping (lookup table or keyed function) is held by an authorized custodian.

**Under HIPAA**: Still PHI if the organization or a downstream party can re-identify.
**Under GDPR**: Explicitly still personal data; pseudonymization is a recognized safeguard under Art. 32.

**Strengths**: Reduces blast radius if downstream is compromised; supports re-linking when clinically necessary; relatively cheap.
**Weaknesses**: Not a regulatory off-switch. Token + linkage table together = PHI. Tokens that encode anything about the patient (e.g., hashed MRN) can be reversed.

**Best practice**: Random tokens, mapping in a separate trust boundary with restricted access, per-purpose tokens to prevent cross-system linkage.

## k-anonymity

Each record in the released dataset is indistinguishable from at least k-1 others on a defined set of quasi-identifiers (e.g., age band, ZIP prefix, gender).

**Strengths**: Conceptually simple, used widely in public-use claims releases.
**Weaknesses**: Vulnerable to homogeneity attacks (everyone in a bucket has the same sensitive value) and background knowledge attacks. Often paired with l-diversity / t-closeness.

## l-diversity

In each k-anonymous group, the sensitive attribute has at least l well-represented values.

**Strengths**: Mitigates homogeneity attacks.
**Weaknesses**: Doesn't address distributional attacks (the attacker knows the global distribution).

## t-closeness

In each group, the distribution of the sensitive attribute is within distance t of the distribution in the whole dataset.

**Strengths**: Defends against skewness/distributional attacks.
**Weaknesses**: Can substantially reduce utility; harder to tune.

## Differential Privacy

Add calibrated noise to query results or to the data itself, bounded by a privacy budget ε. Smaller ε = more privacy, less utility.

**Strengths**: Strong mathematical guarantee that any single record's presence doesn't materially change outputs; composable; defends against arbitrary side information.
**Weaknesses**: Choosing ε is hard and consequential; utility loss for small datasets; team needs DP-literate engineering. Used by some public-health agencies and federated learning settings.

## Synthetic Data

Generate new records that statistically resemble the source but are not real individuals.

**Strengths**: Can preserve column relationships; useful for testing, ML pipelines.
**Weaknesses**: Privacy guarantees depend entirely on the synthesis method. Naive generators (e.g., simple resampling) can memorize and leak. Strong synthetic data uses DP-trained generators with documented guarantees. Don't treat synthetic data as automatically de-identified — verify.

## Combining Approaches

In practice, organizations rarely use one method in isolation:

- **Internal analytics**: pseudonymization + RBAC/ABAC + audit logging
- **External research collaborator**: LDS + DUA, or Safe Harbor / Expert Determination depending on dates/geography needs
- **Public release**: Expert Determination using k-anonymity + l-diversity + suppression
- **Public-health surveillance**: Differential privacy for aggregate releases
- **ML training data**: Layered — pseudonymize, redact free text, de-identify images, hold in a restricted environment, document training-purpose authorization

## Re-identification Risk Re-assessment

Whatever method is chosen, schedule periodic re-assessment:

- When the dataset changes (new fields, new patients, removed columns)
- When external linkage sources change (new public datasets, new search capabilities)
- On a defined cadence regardless (often annual for Expert Determination)
- When the recipient or use case changes

## Final Disclaimers

- Verify the current text of HIPAA §164.514 and any current HHS guidance
- Engage a qualified statistician for Expert Determination
- Consult your privacy/compliance counsel before relying on any method for external release
