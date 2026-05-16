# State Licensure Compacts for Telehealth

Interstate licensure compacts reduce friction for clinicians delivering telehealth across state lines. **They do not replace licensure** — they create expedited pathways or multistate privileges among participating states. Always verify current member states with the issuing compact commission before relying on a compact for a specific encounter.

## Why compacts matter for engineering

The general rule under US law is that a practitioner must be licensed in the state where the **patient is physically located** at the time of the encounter. For a multi-state telehealth platform, that means either:

- The practitioner holds individual licenses in every state served, or
- The practitioner participates in a compact that grants them privilege to practice in the state where the patient is located, or
- A state-specific exception applies (e.g., federal practice authority for VA / DoD / IHS providers, narrow practice-of-medicine exceptions for consultation).

The platform must capture patient location at the start of each encounter and verify the practitioner is licensed (or compact-authorized) to practice in that location at that moment.

## Compact summary

### IMLC — Interstate Medical Licensure Compact

- **Discipline**: MD, DO (allopathic and osteopathic physicians)
- **What it enables**: An **expedited** pathway for physicians to obtain individual full medical licenses in multiple participating states. The physician still holds separate licenses; the compact streamlines the application process.
- **Eligibility**: Generally requires the physician to designate a State of Principal License (SPL) that is an IMLC member state and meets certain criteria (board certification, no disciplinary history, etc.).
- **Not a multistate privilege**: Unlike the NLC (below), IMLC does not grant a single license valid in all member states. The physician holds a separate license per state.
- **Member states**: Verify the current list at the IMLC Commission. Member roster changes annually.

### NLC — Nurse Licensure Compact

- **Discipline**: RN, LPN/VN (registered nurses and licensed practical/vocational nurses)
- **What it enables**: A **multistate license** held in the nurse's primary state of residence (PSOR) grants privilege to practice in all other NLC states without applying for additional licenses.
- **Eligibility**: PSOR must be an NLC state and the nurse must meet the Uniform Licensure Requirements.
- **APRNs**: The Advanced Practice Registered Nurse Compact (APRN Compact) is a separate, newer compact and has its own member-state list. Verify current.
- **Member states**: Verify the current list at the National Council of State Boards of Nursing (NCSBN).

### PsyPACT — Psychology Interjurisdictional Compact

- **Discipline**: Licensed psychologists
- **What it enables**: Authority to practice **telepsychology** across PsyPACT states and limited temporary in-person practice (up to 30 days/year). Requires an E.Passport (telepsychology) and IPC (temporary practice).
- **Eligibility**: Doctoral-level licensed psychologist with a home state that is a PsyPACT participant.
- **Member states**: Verify the current list at the Association of State and Provincial Psychology Boards (ASPPB).

### ASLP-IC — Audiology & Speech-Language Pathology Interstate Compact

- **Discipline**: Audiologists and speech-language pathologists
- **What it enables**: A privilege to practice in other ASLP-IC states based on a home-state license.
- **Member states**: Verify current with the ASLP-IC Commission.

### EMS Compact (REPLICA)

- **Discipline**: EMS personnel (EMT, paramedic)
- **What it enables**: Recognition of EMS personnel licensure across member states for short-term inter-state transport and disaster response. Less relevant to most telehealth platforms but matters for tele-EMS, mobile integrated health, and emergency telehealth.

### Other growing compacts (verify current status)

| Compact | Discipline |
|---------|------------|
| PT Compact | Physical therapists / PTAs |
| OT Compact | Occupational therapists / OTAs |
| Counseling Compact | Licensed professional counselors |
| Social Work Compact | Licensed social workers (LCSW etc.) |
| Dietitian Licensure Compact | Registered dietitians |
| Physician Assistant Compact (PA Licensure Compact) | Physician assistants |

Each compact has its own commission, member-state roster, and operational date. New compacts may be enacted into law in a state but not yet operational (rules pending). Verify both **enacted** and **operational** status before relying on a compact.

## Federal practice authority (overrides state location)

In specific federal systems, providers can practice across state lines without state-by-state licensure when caring for system beneficiaries:

- **VA** (Department of Veterans Affairs) — VA providers can practice telehealth to VA patients regardless of patient or provider state.
- **DoD** (Department of Defense) — TRICARE and military providers have similar federal authority.
- **IHS / Tribal** — Indian Health Service providers.
- **Federal disaster declarations** — narrow, temporary authorities may apply.

These do not generalize to a private multi-state telehealth platform.

## Public-health-emergency (PHE) flexibilities

Many state and federal PHE-era cross-state flexibilities have **expired or transitioned** to state-by-state rules. Do not assume any PHE flexibility is still in effect — verify each state's current law and any payer policy that may have followed the PHE rules.

## Engineering implications checklist

- [ ] Capture and persist **patient location at start of encounter** (state, country) as part of the encounter record.
- [ ] Maintain a per-provider **licensure registry**: state, license number, status, expiration, board, restrictions, compact status, compact privilege scope.
- [ ] Re-check licensure at the start of each visit — not only at provider onboarding.
- [ ] Route encounters to providers licensed (or compact-authorized) where the patient is. Block unlicensed assignments unless an exception applies and is documented.
- [ ] Refresh compact member-state lists from authoritative sources on a defined cadence (quarterly at minimum); flag stale data.
- [ ] Audit log every licensure check (verdict, license number, source) as a FHIR `AuditEvent`.

## Verify-current sources

- IMLC Commission — current member states and SPL criteria
- NCSBN — NLC member states and APRN Compact status
- ASPPB — PsyPACT participants and credential requirements
- ASLP-IC — member states
- Each respective compact commission for the smaller / newer compacts
- State board of medicine / nursing for state-specific exceptions and waivers
