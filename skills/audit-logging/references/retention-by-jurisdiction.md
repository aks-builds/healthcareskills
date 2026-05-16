# Audit Log Retention by Jurisdiction

A working reference for audit log retention requirements across the most common overlays a U.S. healthcare organization encounters. **All values below should be verified against the current text of the cited authority** — state retention requirements in particular change, and some carry separate rules for adult vs. minor records, paper vs. electronic, and different record categories (medical, billing, audit log). Consult your privacy/compliance counsel and records-management lead for binding decisions in your jurisdiction.

## The Floor — HIPAA Documentation Rule

HIPAA's documentation retention rule (45 CFR §164.530(j) for the Privacy Rule; the parallel Security Rule documentation requirement) requires retention of **policies and procedures, communications, and documentation** required by HIPAA for **6 years from the date of creation or the date when last in effect, whichever is later**.

The Security Rule audit-controls requirement (§164.312(b)) requires hardware/software/procedural mechanisms to record and examine activity in systems containing ePHI but does **not** explicitly set a retention period in the rule text — many compliance programs treat the audit log as required documentation and apply the 6-year floor. Verify the current HHS guidance and OCR enforcement posture.

The HIPAA right to an **accounting of disclosures** (§164.528) covers the prior **6 years** of non-TPO disclosures (with carve-outs) — operationally, the underlying audit records must be retained at least that long.

## The CMS Overlay

For Medicare-related records, CMS regulations commonly require **7 years** retention (some sources cite 10 years for specific categories). Audit logs that document access to or modification of Medicare-billed records may be subject to this longer window. Verify the specific CMS regulation that applies (Part A / B / C / D providers, ACOs, MA plans, MSSP) and the current CMS guidance.

For Medicare Advantage and Part D, retention requirements differ from fee-for-service. Verify Chapter 4 (Beneficiary Protections), Chapter 9 (Compliance Program), and current CMS guidance.

## State Law Overlays (Verify Current State Law — These Examples Are Illustrative)

State retention laws apply to medical records primarily; audit log retention may be set by state law, by interpretation of the medical-records retention rule, or left to organizational policy. The table below is **illustrative** and must be verified against the current statute/regulation in each state.

| State | Typical adult medical record retention (verify) | Audit log treatment (verify) |
|---|---|---|
| California | 7 years from last discharge (hospitals); CMIA does not set a single number for audit logs; verify | Often aligned with medical record retention |
| New York | 6 years for adults; 6 years after age of majority for minors; verify | Often aligned with medical record retention |
| Texas | 7 years for adults; until age 21 or 7 years past age of majority for minors; verify | Verify |
| Florida | 5 years (hospitals); 7 years (physicians, often); verify | Verify |
| Illinois | 10 years for hospitals; minor records extended; verify | Verify |
| Massachusetts | Hospitals 30 years per regulation in some categories; verify the current MA reg | Verify |
| Washington | Adult records 10 years; minor records extended; verify | Verify; HMHM Health My Data Act may add audit requirements |
| Pennsylvania | 7 years for adults; longer for minors; verify | Verify |
| Other states | Highly variable | Verify with state medical society / counsel |

**Verify against current state law before relying on any number above.** The numbers above are commonly cited but have changed at various times; specific retention is set by the current text of state hospital licensing regulations, professional licensing boards, and state-specific privacy laws.

## Special Categories

### Minors
Most states extend the medical record retention period for minors — commonly some-number-of-years past the age of majority (which varies by state, usually 18 or 21). Audit records related to a minor's care should follow the same extended retention.

### Mental Health and SUD Records
- **42 CFR Part 2** (substance use disorder records) has its own retention and disclosure regime — verify the current rule and recent alignment with HIPAA
- State mental health statutes (e.g., California's LPS Act, others) may impose specific retention

### Genetic Information
State genetic information privacy laws vary; some impose extended retention or destruction requirements. Verify.

### Imaging / DICOM Studies
- Hospital licensing rules typically govern; some states require specific minimum retention for radiology studies (often 5-7 years; mammography under the federal MQSA has its own minimum — verify)
- Audit logs of access to imaging studies follow the medical-record or organizational policy

### Research Records
- ICH GCP retention (typically 15 years for trial records in pharma — verify)
- FDA-regulated trial records under 21 CFR Part 312 — verify the current text
- IRB records under the Common Rule — typically 3 years after research closure; verify
- Audit logs of access to research data should follow the longest applicable retention

### Workers' Compensation, Disability, and Insurance
State-specific; verify with the carrier and state regulator.

## Federal Contract Overlays

| Overlay | Typical audit log retention (verify) |
|---|---|
| FedRAMP | Online (hot/warm) for a defined period (often 90 days), offline/archive for a longer period (often 1-3 years online + cold archive aligned to the agency's records schedule) — verify FedRAMP current control set |
| CMMC / NIST 800-171 | DoD CUI handling — verify current CMMC level requirement |
| VA / DoD contracts | Agency-specific — verify the contract |

## Litigation Hold

A litigation hold issued by counsel suspends destruction of records — including audit logs — within the scope of the hold. The hold supersedes the normal retention schedule. Operationally:

- Document the hold scope (what data, what date range, what custodians)
- Suspend automated deletion for in-scope data
- Preserve audit logs that may evidence access or modification
- Release the hold formally when counsel approves

Audit log integrity becomes especially important under hold — defense and plaintiff both may rely on the logs.

## Practical Retention Tiers

A common implementation pattern is tiered retention with the audit log accessible at different speeds and costs:

| Tier | Typical span | Storage |
|---|---|---|
| Hot — fully indexed in SIEM | 30-90 days | High-cost, fast query |
| Warm — searchable with delay | 90 days to 1-2 years | Medium-cost |
| Cold — archive, rehydrate for investigation | 2 years to retention horizon | Low-cost (object storage with lifecycle, immutable retention lock) |

The retention horizon is the **maximum** of the applicable overlays (HIPAA 6 years, CMS 7 years, state law, federal contract, research, litigation hold). Many organizations standardize on **7 years** as the floor and extend for special categories.

## Engineering Considerations

- **Cryptographic erasure** at end of retention is preferred to in-place deletion for very large archives — destroy the encryption key, leaving ciphertext effectively unrecoverable
- **Backups and replicas** must be in scope of the retention schedule — backups that outlive the policy create their own risk
- **Migration** during system replacement must preserve audit records as "true copies" — verify integrity post-migration
- **Lock periods** (S3 Object Lock, equivalent on other clouds) enforce retention against tampering and accidental deletion
- **Time synchronization** — UTC storage prevents retention-clock ambiguity around DST

## Final Disclaimers

- Verify every state-specific retention number against the current state hospital licensing regulation, professional board rule, or privacy statute
- Verify HIPAA, CMS, FedRAMP, and any other federal overlay against current text
- Verify research retention against the applicable sponsor / IRB / FDA / ICH GCP requirement
- Engage your records-management lead and privacy/compliance counsel for binding retention decisions
