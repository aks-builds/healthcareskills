# Drug-Drug Interaction Data Sources

Drug-drug interaction (DDI) data is the foundation of every safety check at order entry and at medication reconciliation. The major US sources differ in severity grading, mechanism narrative, management recommendation, currency, and licensing model. Pick **one** authoritative source per installation — running multiple in parallel produces conflicting alerts and confused clinicians.

> Verify current licensing terms, content currency, and EHR-vendor integration support directly with each provider. Pricing, content scope, and supported integrations change.

## Comparison

| Source | Vendor | Model | Strengths | Considerations |
|--------|--------|-------|-----------|----------------|
| **First Databank MedKnowledge** | First Databank (FDB) | Commercial license | Widely embedded in US EHRs; mature DDI, dose, allergy, duplicate content; structured severity and management | Commercial licensing required; specific scope per module |
| **Wolters Kluwer Medi-Span** | Wolters Kluwer | Commercial license | Widely embedded in US EHRs and pharmacy systems; structured content; Drug Information Framework module for DDI | Commercial licensing required |
| **Cerner Multum** | Cerner / Oracle Health | Commercial license | Embedded in some Cerner installs and other systems; structured DDI | Cerner ecosystem affinity |
| **Lexicomp** | Wolters Kluwer | Commercial license | Combined reference + DDI; clinician-readable content; mobile and integrated forms | Reference-heavy; integration footprint varies |
| **Truven Micromedex** | Merative (formerly IBM Watson Health) | Commercial license | Reference + DDI; structured content for integration | Licensing varies by use case |
| **RxNav DDI API** | NLM (NIH) | Public, no license fee | Free; useful for prototyping, research, public-health work | Coverage is limited; underlying content has changed over time and currency depends on the source; production use should be evaluated carefully |
| **DrugBank** | DrugBank | Commercial + academic tiers | Used by some open-source tools; structured drug data including DDI | Licensing tier determines use; academic vs commercial usage rights differ |

## Differentiators to Evaluate

When choosing a source, evaluate against the use cases you actually have:

### Severity Grading

How does the source classify interaction severity? Do classes line up with how your CPOE alerts (interruptive vs passive vs informational)? Typical levels include contraindicated / severe / moderate / minor — but specific category names and thresholds vary by source.

### Mechanism Narrative

For each interaction, is there a human-readable explanation? Clinicians override less when they can see why the alert fired and what the management recommendation is.

### Management Recommendation

Does the source provide actionable guidance (avoid combination, monitor specific parameter, dose-adjust, separate by time) — not just "interacts"?

### Currency

How often is the content updated? How is critical new information (FDA safety alert, new mechanism discovered) propagated to subscribers?

### Coverage

- US drug list completeness (RxNorm-mappable, NDC-mappable).
- International formulations if you operate outside the US.
- Drug-disease interactions, not just drug-drug.
- Drug-food and drug-alcohol where relevant.
- Drug-allergy cross-sensitivity.

### Integration Surface

- Native APIs (REST, SOAP, batch file).
- EHR-vendor adapters already built.
- RxNorm code mapping support.
- NDC-level resolution.
- Schema stability across releases.

### Licensing Model

- Per-bed, per-clinician, per-API-call, or flat institutional.
- Use rights (live patient care vs research vs commercial product).
- Sublicensing if you embed in a product distributed to customers.

## Integration Patterns

### Pattern 1 — Embedded in the EHR

The EHR has a native integration with one source. The CPOE alert engine fires on order-sign using that source's content. This is the default for most large EHR installs.

Your responsibility: tune the alert thresholds and override workflow, not the underlying content.

### Pattern 2 — Pulled at Order Time

Your application calls the DDI source's API at order time with the patient's active med list and the proposed order. The source returns interactions; you render alerts.

Pros: content is always current; you control the alert UI.
Cons: latency in the order-sign path; API uptime is a dependency; cost per call.

### Pattern 3 — Cached Locally

You replicate the source's content locally (per the license) and query in-process. Updates land via daily / weekly content drops.

Pros: no per-call latency or cost; predictable.
Cons: licensing typically requires the replica; staleness window between drops; you own the update pipeline.

### Pattern 4 — Service Behind Your Stack

You build a DDI microservice fronting one source's API or local cache, used by both CPOE and med rec. Single source of truth.

This is the typical pattern for institutions with multiple ordering surfaces (inpatient CPOE, ambulatory e-Rx, ED tracker, transition-of-care tools).

## DDI in Med Rec vs CPOE

DDI checks should run against the **post-reconciliation** active medication list, not the pre-rec list:

- During admission med rec, the working list is being built — running DDI on every intermediate state creates noise.
- After the reconciler signs off the BPMH and the admission orders, the merged active list is the right substrate for DDI.
- For discharge, run DDI against the planned discharge list before it goes to the patient and the outpatient pharmacy.

## RxNorm — The Glue

Whichever DDI source you choose, you will need to map your application's medication concepts to the source's internal IDs. RxNorm is the normalizing terminology:

- RxCUI at SCD (Semantic Clinical Drug) level is the typical comparison key.
- NDCs in fill history map to RxCUIs via NLM's published mappings.
- Brand-name and generic concepts link via RxNorm relationships.
- Use RxNav APIs (NLM) to walk relationships and normalize.

Without a clean RxNorm mapping layer, the DDI source's content cannot be reliably queried against the patient's actual med list.

## Quality and Governance

Even with a good source, your DDI program needs governance:

- **One source per installation.** Decide and document.
- **Alert tuning per service line.** ICU sometimes legitimately co-administers drugs that the source flags; tune.
- **Measure fire rate, accept rate, override-with-reason.** Retire low-acceptance alerts.
- **Annual content review.** Walk the top 100 most-fired alerts; confirm they are still firing for the right reasons.
- **Trigger-driven review.** New FDA safety alert, new high-risk drug added to formulary, internal safety event — review immediately.
- **Document who owns the DDI tuning** — a specific person, not "informatics."

## Common Pitfalls

- Running multiple DDI sources in parallel and showing the union of alerts. This is the fastest path to clinician override fatigue.
- Using RxNav DDI as the primary source in a production hospital without acknowledging its limitations.
- Skipping the RxNorm normalization layer and matching by string. Brand vs generic, repackager NDCs, and synonyms all break this.
- Firing every interaction at every severity. Tune by severity, by drug class, by service line.
- Treating an override as a system failure. Most overrides are clinically valid; high override rates mean tuning is wrong, not that clinicians are wrong.
- Forgetting to update local caches. Stale DDI content can miss FDA safety alerts.

> All vendor names, licensing assumptions, and integration patterns described here must be confirmed against current vendor terms. Coverage, content, and pricing change over time.
