# NIST 800-63 Identity Proofing Levels for Patient Portals

NIST Special Publication 800-63 (Digital Identity Guidelines) defines the assurance levels US federal systems and federally-aligned programs use for identity proofing, authentication, and federation. Healthcare patient portals — particularly any portal that must comply with the **CMS Patient Access Rule** — typically target NIST 800-63 levels for identity proofing (IAL) and authentication (AAL).

This reference covers the practical application of IAL and AAL to patient portals. **Verify against the current version of NIST 800-63 before locking architecture; NIST has revised the document family and SP 800-63-4 has been in public draft.**

## The three "AL" dimensions

NIST 800-63 separates three concerns. A patient portal makes a choice on each.

| Dimension | What it answers | Document in the family |
|-----------|-----------------|------------------------|
| **IAL** — Identity Assurance Level | How strongly the system verified the person's real-world identity | SP 800-63A |
| **AAL** — Authenticator Assurance Level | How strongly the system can verify the person at each session | SP 800-63B |
| **FAL** — Federation Assurance Level | How strongly an assertion from one system to another can be trusted | SP 800-63C |

Most patient-portal teams talk about "IAL2" without separating the three. Be precise — IAL2 + AAL1 is a legitimate but weak posture for sensitive operations.

## IAL — Identity Assurance Levels

### IAL1

- **No identity proofing.** A self-asserted identity. The user provides claims (name, DOB, email) and the system does not verify them.
- **Appropriate for**: anonymous content access, pre-account browsing.
- **Patient portal application**: marketing pages, account creation page itself, possibly a non-clinical waitlist form.

### IAL2

- **Either remote or in-person identity proofing.** Resolves the claimed identity to a single real-world identity, validates evidence (typically a government-issued ID + biometric / liveness check, or a combination of strong evidence), and verifies the identity belongs to the person presenting it.
- The CSP (credential service provider) follows the SP 800-63A procedures.
- **Appropriate for**: patient access to their own medical records, payer member portals subject to CMS-9115, anything with PHI access.
- **Patient portal application**: the floor for any portal that exposes clinical data. CMS-9115 effectively requires IAL2 patterns for the payer Patient Access API.

### IAL3

- **In-person or supervised remote proofing**, additional evidence requirements, and stronger anti-fraud controls.
- **Appropriate for**: high-risk transactions where remote-only IAL2 isn't sufficient.
- **Patient portal application**: rarely used as a floor; sometimes used as a step-up for high-risk operations (provider impersonation prevention, large financial transactions).

## AAL — Authenticator Assurance Levels

### AAL1

- **Single-factor authentication.** Password alone qualifies.
- **Patient portal application**: should be considered unacceptable as the floor for a portal exposing PHI. Use AAL2 minimum.

### AAL2

- **Two-factor authentication.** Something you know + something you have, or equivalent. Common factors: password + TOTP authenticator app, password + push notification, password + WebAuthn / passkey.
- **Patient portal application**: the practical floor for patient access. Step-up to AAL3 for high-risk operations only when justified.

### AAL3

- **Hardware-based, phishing-resistant authentication.** Cryptographic authenticators with verifier impersonation resistance (e.g., FIDO2 with attestation, smart cards).
- **Patient portal application**: usually overkill for patients; sometimes used for clinician portals or for the proxy granting access to highly sensitive categories.

## Patient-portal-specific application

### Typical floor

For a portal exposing patient clinical data:

- **IAL2** for account creation / first activation.
- **AAL2** for all authenticated sessions.
- **Step-up to AAL3** (or step-up proofing) for sensitive operations: changing a proxy relationship, downloading the full record, requesting medication refills for controlled substances, granting third-party API access.

### Activation-code flow (provider-tethered)

When the patient receives a printed activation code at a clinic visit, the in-person staff interaction is the proofing event. This can satisfy IAL2 for the resulting account, because the patient's identity was verified by staff at the encounter and the activation code is a tightly time-bound shared secret.

### Self-service proofing flow

When the patient creates an account online without a prior in-clinic event, IAL2 is achieved via a third-party CSP or in-house equivalent:

| Vendor | Notes |
|--------|-------|
| ID.me | Heavy government / VA presence; supports IAL2 patterns. |
| CLEAR | Identity + biometric, widely deployed in airports and consumer services. |
| Persona | Configurable KYC / IAL2 flows. |
| Stripe Identity | Fast integration, document + selfie. |
| Socure | Risk-scored identity verification; integrates with broader fraud signals. |
| LexisNexis Risk Solutions | Knowledge-based + identity-graph approaches. |

The CSP returns a proofing token; the portal binds it to the resulting account and persists the **assurance level**.

### Insurance-card + KBA (payer-side)

Common for payer member portals. The member provides plan ID, DOB, name, and answers knowledge-based authentication questions. Quality varies — KBA-only flows are increasingly seen as below IAL2 because the underlying data is widely breached. Many payers now combine card-based identifiers with document-and-selfie steps for any account that will access PHI.

### Step-up vs. floor

Two valid patterns:

- **Floor proofing**: every account is IAL2 at creation. Highest security but highest friction.
- **Just-in-time step-up**: account is created at a lower assurance for non-PHI tasks, then stepped up to IAL2 before the first PHI access.

The just-in-time pattern reduces account-creation drop-off but requires the portal to enforce the gate consistently — every PHI surface must check assurance level before render.

## Proxy and adolescent considerations

- **Each proxy** is a separate identity and must be proofed in their own right. A parent acting as a child's proxy must reach IAL2 themselves; the child's record does not lower the proofing requirement.
- **Adolescents** who gain direct access at a state-specified age (often 12-17) need their own proofing path that does not require parental involvement — proofing methods that depend on a parent's payment card or address would block legitimate adolescent access.

## Engineering implications checklist

- [ ] Decide and document IAL / AAL targets per persona (patient, proxy, adolescent, caregiver, clinician).
- [ ] Persist **assurance level** on the account, separate from authentication state.
- [ ] Enforce the assurance level at the resource gate, not only at login.
- [ ] Step up assurance for sensitive operations; revoke step-up after a defined session window.
- [ ] Log every proofing event as a FHIR `AuditEvent`.
- [ ] Re-proof on a new device for AAL2/3 (biometric unlock alone is not full reproofing).
- [ ] Capture vendor proofing-token IDs in the account record for audit / dispute / reproof.
- [ ] Provide an **accessible** proofing path — visually-impaired and motor-impaired patients should have a proofing flow that does not require selfie-with-ID gymnastics.

## Verify-current sources

- NIST SP 800-63 (current published version: SP 800-63-3; SP 800-63-4 has been in public draft — confirm latest)
- ONC / HHS guidance on patient identity for FHIR APIs
- CMS-9115-F (Patient Access Rule) — references identity proofing for the consumer API
- The HEART (Health Relationship Trust) working-group profiles for OAuth + IAL profiles in healthcare
