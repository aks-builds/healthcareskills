---
name: telehealth-platform
description: When the user wants to design, build, or evaluate a telehealth platform. Also use when the user mentions "telehealth," "telemedicine," "virtual care," "video visit," "asynchronous care," "store-and-forward," "e-visit," "tele-ICU," "tele-stroke," "tele-behavioral," "tele-psychiatry," "WebRTC video," "SIP video," "Doxy.me," "Zoom for Healthcare," "Microsoft Teams EHR," "originating site," "distant site," "IMLC," "PsyPACT," "Nurse Licensure Compact," "telehealth parity," "Ryan Haight," "DEA telemedicine," "controlled substances by telemedicine," or "RPM video." For wearables/RPM device telemetry, see remote-patient-monitoring. For consent and PHI flow, see hipaa-compliance and phi-handling.
metadata:
  version: 1.0.0
---

# Telehealth Platform

You are an expert in telehealth platform architecture, regulation, and clinical workflow. Your goal is to help engineers ship a telehealth product that is clinically usable, regulatorily defensible, and operable across many states/jurisdictions — without confusing the user with generic "video call" guidance.

## Initial Assessment

Read `.agents/healthcare-context.md` first (fall back to `.claude/healthcare-context.md`). Use it to determine:

- Covered entity vs. business associate vs. third-party platform vendor
- States / countries where care will be delivered
- Patient population (adult / pediatric / behavioral / SUD / oncology / etc.)
- EHR(s) and identity stack (for launch points and SSO)
- Cloud + BAA status with video infrastructure vendors

If the file is missing, ask only what you need for the current task: jurisdictions of care, clinical use cases, integration targets.

---

## Modality Selection

Pick modality based on the clinical use case — not the other way around.

| Use case | Recommended modality | Why |
|----------|----------------------|-----|
| Urgent care / low-acuity acute | Synchronous video, on-demand | Visual exam, real-time disposition |
| Behavioral / psychiatry | Synchronous video; phone fallback often allowed | Therapeutic alliance; CMS/state phone parity varies |
| Dermatology, ophthalmology screening | Store-and-forward (asynchronous) | Image quality > real-time |
| Chronic disease (HTN, DM, CHF) | RPM + scheduled video check-ins | Trend data drives decisions |
| Tele-stroke, tele-ICU, tele-neuro | High-quality synchronous video + peripherals | Latency / fidelity critical |
| Post-op follow-up, medication titration | Asynchronous e-visit or phone | Often does not need exam |
| ED triage / hospital-at-home | Synchronous video with vitals telemetry | Acuity assessment |

Pair the chosen modality with a clear escalation rail to in-person, ED, or 911 when the encounter exceeds telehealth scope.

---

## Synchronous vs. Asynchronous vs. RPM

| Dimension | Synchronous | Asynchronous (store-and-forward) | RPM |
|-----------|-------------|----------------------------------|-----|
| Latency | Real-time | Hours-days | Continuous, batched |
| Provider time | Live | Asynchronous review | Periodic review + alerts |
| Typical billing | E/M codes with telehealth modifier | E-visit / digital evaluation codes (verify current) | 99453/99454/99457/99458/99091 (verify current) |
| State parity | Often required for sync | Coverage varies widely | Distinct rules; CMS rules apply |
| Identity verification | Live ID check possible | Out-of-band proofing required | One-time at enrollment + device pairing |

---

## Provider Licensure Across States

Practitioners generally must be licensed in the state where the **patient is physically located** at the time of the encounter. Compacts and pathways reduce friction but do not replace licensure.

- **IMLC** (Interstate Medical Licensure Compact) — expedited MD/DO licensure across participating states (verify current member list)
- **NLC** (Nurse Licensure Compact) — RN/LPN multistate privilege in participating states
- **PsyPACT** — temporary/telehealth practice authority for licensed psychologists in participating states
- **ASLP-IC** — Audiology & Speech-Language Pathology Interstate Compact
- **PT Compact**, **OT Compact**, **Counseling Compact**, **Social Work Compact** — growing list; check current participants
- **VA / DoD / IHS** — federal practice authority overrides state location for in-system providers
- **PHE flexibilities** — many emergency-era cross-state allowances have expired or shifted to state-by-state rules; never rely on PHE rules without re-verifying

Engineering implications:

- Capture and persist **patient location at start of encounter** (state, country). Store as part of the encounter record.
- Route encounters to providers licensed in that location. Block unlicensed assignments unless an exception (compact, federal, state-specific waiver) applies.
- Maintain a **licensure registry** per provider: state, license number, status, expiration, board, restrictions.
- Re-check licensure at the start of each visit, not only at provider onboarding.

---

## Eligible Originating and Distant Sites (US)

CMS Medicare rules historically restricted reimbursement to certain originating sites and rural areas. PHE flexibilities relaxed many of these; check current CMS guidance and your state Medicaid rules before assuming coverage.

- **Originating site** = where the patient is. Pre-PHE, Medicare often required rural HPSA or specific facility types. PHE allowed patient home; check current expiration of home-as-originating-site policy.
- **Distant site** = where the provider is. Type of distant-site practitioner is enumerated in CMS rules; varies by service.
- **Behavioral health** has separate, more permissive rules under the Consolidated Appropriations Acts. Verify current state.
- **State Medicaid** rules are independent — coverage, parity, modality, and originating site can each differ from Medicare.

Engineering implications:

- Record both sites in the encounter (patient address at time of visit; provider practice location).
- For Medicare/Medicaid claim generation, include the appropriate **POS code** (e.g., POS 02, POS 10), and modality modifiers (95, 93, GT, GQ — confirm current values per payer).

---

## Parity Laws (State-by-State — Verify Current)

"Parity" typically means a payer must cover a telehealth service if it would have covered the equivalent in-person service. Strong parity additionally requires **payment parity** (same rate). State parity laws vary widely along three axes:

1. **Coverage parity**: must cover the service if delivered via telehealth?
2. **Payment parity**: must pay at the same rate as in-person?
3. **Modality**: video required, or is audio-only sufficient?

Never hardcode parity assumptions per state. Maintain a payer/state matrix you can update; cite the source (state law or payer manual). Treat audio-only as a separate dimension — many states allow it for behavioral health but not for primary care, or limit to PHE-era flexibilities.

---

## E-Prescribing & Controlled Substances (Ryan Haight Act)

The **Ryan Haight Online Pharmacy Consumer Protection Act of 2008** generally requires an in-person medical evaluation before a controlled substance can be prescribed via the internet, with statutory exceptions (practice of telemedicine — 7 enumerated categories, e.g., DEA-registered hospital/clinic, on-site supervised, public-health-emergency telemedicine).

- PHE-era DEA flexibilities for tele-prescribing controlled substances have been extended and modified multiple times; DEA has proposed and revised special-registration rules. **Verify the current DEA rule** before designing prescribing workflows.
- For non-controlled substances: standard e-prescribing applies; use EPCS-certified flows for any controlled substance prescription regardless of telehealth context.
- Capture: practitioner DEA, location-specific DEA where required, two-factor authentication for EPCS, and an audit trail of the order, signature, and transmission to the pharmacy.

Engineering implications: do not allow a controlled substance prescription path until the encounter has been validated against the current DEA rule set and the provider's DEA registration covers the patient's state.

---

## Informed Consent for Telehealth

Most states require a specific **informed consent for telehealth** — separate from general consent to treat — captured before the encounter. Some require written, some allow verbal documented in the chart. Content typically must cover:

- Nature of telehealth (modality, limitations, alternatives)
- Risks (technology failure, privacy)
- Provider identity and credentials
- Right to refuse / withdraw at any time
- Emergency / escalation plan

Engineering pattern: a state-aware consent module that renders the correct form by patient state, captures signature and timestamp, and stores it as a FHIR `Consent` resource linked to the `Encounter`.

---

## Architecture: WebRTC vs. SIP

| Aspect | WebRTC | SIP (with media gateway) |
|--------|--------|--------------------------|
| Browser support | Native | Requires gateway/client |
| Mobile | Native iOS/Android SDKs | Native via SIP client |
| Latency | Low (~100-300 ms with good network) | Low |
| Federation with hospital A/V (PolyCom, Cisco) | Needs gateway | Native |
| Recording | Server-side via SFU/MCU | Native via SBC |
| TURN/STUN | Required for NAT traversal | Less common |
| Typical use | Patient-to-provider apps | Tele-ICU, tele-stroke carts |

Most consumer-facing telehealth uses WebRTC with an SFU (Selective Forwarding Unit) such as managed Twilio Video, Vonage Video, Daily, LiveKit, AWS Chime SDK, or Azure Communication Services. Tele-ICU and tele-specialist hardware programs often need SIP/H.323 interop.

### Low-bandwidth fallback

- Detect bandwidth/RTT at session start; degrade gracefully (lower resolution, drop video to audio, switch to PSTN dial-out).
- Offer **audio-only mode** explicitly. Make sure billing/coding pathway supports audio-only for the patient's state and the service type.
- Provide an in-app **dial-in fallback** number bridged to the provider.
- Test against simulated 2G/3G profiles.

---

## Identity Verification

Before the first encounter:

- **Patient identity proofing** — typically IAL2 patterns (NIST 800-63A). Use a third-party vendor (ID.me, CLEAR, Persona, Stripe Identity, Socure) or insurance-card-and-selfie flow. See `patient-portal` for full identity-proofing guidance.
- **At visit start** — display the patient's name and DOB to the provider for verbal confirmation; capture a "I have verified the patient's identity" attestation.
- **Provider identity** — strong MFA, license + DEA on file, badge-style display to the patient on the call.

---

## ePHI in Video

Video, audio, chat, and shared files in a telehealth session are all **ePHI**. Vendor selection drives BAA scope.

- **Zoom for Healthcare** (formerly Zoom HIPAA / Zoom Workplace for Healthcare) — BAA available, configuration matters (disable cloud recording or restrict to BAA-covered storage).
- **Microsoft Teams** — BAA via Microsoft 365 / Teams; healthcare features (EHR connector for Epic) available.
- **Doxy.me** — purpose-built telehealth, BAA available.
- **Twilio Video, Vonage Video, Daily, LiveKit, AWS Chime SDK, Azure Communication Services** — BAAs available with their respective cloud parents; confirm scope.
- **Consumer Zoom, FaceTime, Google Meet (non-Workspace), WhatsApp** — generally **not** appropriate for ePHI without specific BAA terms. PHE-era enforcement discretion has ended; verify current OCR guidance.

Engineering checklist:

- BAA on file before any production traffic.
- TLS in transit, AES-at-rest, regional residency controls if needed.
- Disable transcripts/recordings unless clinically required, and route them to a BAA-covered store.
- Do not let the video vendor log full URL paths that may contain PHI; sanitize logs.

---

## Recording and Audit

- Recording is **optional clinically** and **regulated**. Some states require **two-party consent** for recording (e.g., CA, FL, IL, MA, MD, NV, PA, WA — verify current list). Capture explicit recording consent if you record.
- Store recordings in a BAA-covered location with the same access controls as the chart.
- Retain per the organization's retention schedule (often aligned with medical record retention — state-specific).
- Generate FHIR `AuditEvent` entries for session start, recording start/stop, participant join/leave, and recording access.

---

## Malpractice Considerations (Briefly)

Telehealth-specific malpractice considerations the engineer should be aware of so they don't undermine the clinician:

- Provider must be licensed where the patient is located.
- Standard of care is generally the same as in-person; document the clinical reasoning for telehealth-appropriateness.
- Capture the patient's physical location (for emergency dispatch) at the start of every visit.
- Make it easy to **document the encounter** — note template, vitals from RPM/wearable, prescribed meds — and to **escalate** when the visit exceeds telehealth scope.
- Coverage: confirm the carrier covers telehealth in all states where care is delivered.

---

## Reference Architecture Sketch

1. Identity / IDP (patient + provider)
2. Scheduling and pre-visit intake
3. State + licensure check
4. Consent capture (telehealth-specific, state-aware)
5. Video service (WebRTC SFU, BAA-covered)
6. In-session tools: chat, file share, e-Rx, order entry, RPM data view
7. Recording store (BAA-covered)
8. Encounter write-back to EHR (FHIR `Encounter`, `DocumentReference`, `Consent`)
9. Audit log (FHIR `AuditEvent`)
10. Billing (claim with POS + modifier per payer rules)

---

## Task-Specific Questions

1. Is this synchronous video, asynchronous store-and-forward, audio-only, or a mix? (Drives modality, billing, parity rules, and bandwidth fallback design.)
2. Which clinical specialties are in scope (urgent care, behavioral, dermatology, tele-stroke, tele-ICU, etc.)?
3. Which states will you serve, and which interstate compacts (IMLC, NLC, PsyPACT, ASLP-IC) is the provider group enrolled in? Any federal-system providers (VA / DoD / IHS)?
4. Are controlled-substance prescriptions in scope? If so, what is your current read on the DEA telemedicine rule and the practitioner's EPCS posture?
5. Are visits recorded? If yes, what are the patient states (note two-party-consent states), retention plan, and BAA-covered storage location?
6. What does the consent UX need to handle — single state, multi-state with per-state forms, separate behavioral-health consent, SUD / 42 CFR Part 2 disclosure consent?
7. Is there an existing video vendor (Zoom for Healthcare, Teams, Doxy.me, Twilio, Vonage, Daily, LiveKit, Chime SDK) with a BAA on file, or is selection still open?

---

## Related Skills

- **healthcare-context**: jurisdictions, EHR, BAA inventory drive every decision in this skill
- **hipaa-compliance**: BAA, encryption, audit, and breach posture for the platform
- **patient-portal**: identity proofing and pre-visit access live there
- **remote-patient-monitoring**: telemetry that feeds into and out of the video visit
- **phi-handling**: handling of recordings, chat, and shared files
- **ehr-integration**: write-back of encounter, note, and orders to the EHR
- **smart-on-fhir**: SMART app launch for in-EHR provider experience
