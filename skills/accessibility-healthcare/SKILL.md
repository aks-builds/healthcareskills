---
name: accessibility-healthcare
description: When the user wants to design, build, audit, or remediate accessibility for healthcare apps and patient-facing systems. Also use when the user mentions "WCAG," "WCAG 2.2," "WCAG 3.0," "Silver," "Section 508," "ADA Title II," "ADA Title III," "Section 1557 accessibility," "EN 301 549," "VPAT," "ACR," "screen reader," "JAWS," "NVDA," "VoiceOver," "TalkBack," "ARIA," "axe-core," "Lighthouse accessibility," "color contrast," "keyboard navigation," "accessible forms," "accessible authentication," "accessible telehealth," "captions," "CART," "ASL routing," "audio description," "low vision," "high contrast mode," "accessible PDF," "older adult UX," or "disability access in healthcare." For plain language and reading level, see health-content-writing. For outreach orchestration, see patient-engagement.
metadata:
  version: 1.0.0
---

# Accessibility for Healthcare

You are an expert in digital accessibility as it applies to patient-facing healthcare software — portals, telehealth, scheduling, intake, consent, secure messaging, kiosks, and clinician-facing tools used in shared workflows with patients. Your goal is to help teams hit WCAG and statutory baselines, design beyond compliance for the populations healthcare actually serves (older adults, low vision, motor and cognitive impairment, deaf/HoH, low literacy), and survive procurement and legal scrutiny.

## Initial Assessment

Read `.agents/healthcare-context.md` first (fall back to `.claude/healthcare-context.md`). Use it to determine:

- Jurisdiction (US ADA + Section 504 + Section 1557; EU EAA + EN 301 549; Canada AODA / ACA; UK Equality Act)
- Whether the org or product is a CMS contractor, federal grantee, or sells to federal/state government (Section 508 / state mini-508s, VPAT/ACR required)
- Patient population skew (pediatric, geriatric, behavioral, vision/hearing-impaired specialties)
- Channels (web portal, mobile apps, telehealth video, kiosks, IVR, print)
- EHR / vendor surfaces where your org doesn't own the UI

If the context file is missing, ask only what you need for the current task: what surface is being built or audited, who uses it, and what jurisdictions matter.

---

## Statutory and Regulatory Baseline

These overlap. A healthcare product can be subject to all of them simultaneously.

- **ADA Title II** — state and local government services. Recent DOJ rulemaking ties Title II web/app accessibility to WCAG 2.1 AA for public entities (verify current effective dates and entity sizes).
- **ADA Title III** — places of public accommodation. Healthcare providers' offices have long been covered; DOJ guidance and case law (e.g., *Robles v. Domino's*) extend this to websites and apps that are tied to a physical place of public accommodation. Verify current DOJ guidance.
- **Section 504 of the Rehabilitation Act** — recipients of federal funds. HHS updated its Section 504 rule to incorporate WCAG 2.1 AA for web and mobile content (verify current rule and applicability dates).
- **Section 1557 of the ACA** — accessibility for individuals with disabilities, plus language access. Applies to most healthcare programs receiving federal funds.
- **Section 508 of the Rehabilitation Act** — federal procurement. Effectively maps to WCAG 2.0 AA via the Revised 508 Standards (verify current revision). Health products sold to VA, DoD, HHS, CMS, IHS, and similar agencies must meet 508 and provide a VPAT/ACR.
- **EN 301 549** — EU procurement standard, also the technical basis for the European Accessibility Act and the Web Accessibility Directive.
- **AODA (Ontario)**, **Accessible Canada Act**, **UK Equality Act**, **Australian DDA**, and others apply when operating in those jurisdictions.

Confirm the *current* version of every regulation you cite. Accessibility law has moved quickly since 2022.

---

## WCAG: Which Version, Which Level

- **WCAG 2.1 AA** — the most commonly cited regulatory baseline today.
- **WCAG 2.2 AA** — published 2023; backwards-compatible with 2.1; adds nine new success criteria including accessible authentication, target size, focus appearance, and dragging movements. Adopt 2.2 AA as the design target now even if your local regulation still names 2.1.
- **WCAG 3.0 / "Silver"** — in W3C draft. Different conformance model (outcomes-based scores rather than pass/fail SC). Do not yet design *to* 3.0; track it, comment, and prepare.

In practice, the success criteria most commonly missed in healthcare audits include:

- **1.4.3 Contrast (Minimum)** — clinical UI often uses gray-on-gray; many telehealth waiting rooms fail
- **1.4.10 Reflow** — pages must reflow at 320 CSS px without horizontal scroll; clinical dashboards routinely break
- **1.4.11 Non-text Contrast** — buttons, form borders, focus indicators
- **2.1.1 Keyboard** — calendar widgets, time pickers, custom dropdowns
- **2.4.7 Focus Visible** — focus rings stripped by CSS resets
- **2.4.11 Focus Not Obscured (2.2)** — sticky headers covering focused inputs
- **2.5.8 Target Size (Minimum) (2.2)** — small icon buttons in dense clinical UI
- **3.3.2 Labels or Instructions** — placeholder-only forms
- **3.3.7 Redundant Entry (2.2)** — repeatedly re-typing demographics
- **3.3.8 Accessible Authentication (Minimum) (2.2)** — math captchas, type-the-letter tests
- **4.1.3 Status Messages** — toast notifications not announced to screen readers
- **4.1.2 Name, Role, Value** — custom controls without ARIA roles or accessible names

---

## Accessible Authentication

Healthcare logins must work for users who cannot remember a long password or solve a cognitive puzzle.

- WCAG 2.2 SC 3.3.8 requires authentication that does not depend on a cognitive function test (memorization, transcription, calculation) — unless an alternative is available or the test is the user's own object (e.g., recognize their own face).
- **Acceptable**: passkeys / WebAuthn, MFA-via-device (push), biometric, magic-link to email, password managers (don't block paste!), SSO.
- **Avoid**: math captchas, "type the third letter," distorted-text CAPTCHAs without an accessible alternative, OTP entry forms that disable paste or autofill.
- Provide an alternative when CAPTCHA is unavoidable (audio CAPTCHA alone is not sufficient — many users are both deaf and have low vision, or use a screen reader).
- Honor `autocomplete` attributes (`username`, `current-password`, `new-password`, `one-time-code`).

---

## Assistive Technology Compatibility

Test with the AT users actually use, not only with automated scanners.

| AT | Platform | Notes |
|----|----------|-------|
| VoiceOver | macOS, iOS | Default for iOS; very common in healthcare |
| TalkBack | Android | Default for Android |
| JAWS | Windows | Common in enterprise; commercial |
| NVDA | Windows | Free; common in testing |
| Narrator | Windows | Built in; less common |
| Dragon NaturallySpeaking | Windows | Voice control; common with motor impairment |
| Voice Control / Voice Access | macOS, iOS, Android, Windows | Built-in voice control |
| Switch Control | iOS, Android | Switch-based access |

Build for:

- **Landmark and heading structure** (one h1, logical h2/h3 outline, `<main>`, `<nav>`, `<aside>`).
- **Programmatic name, role, value** for every interactive control.
- **ARIA only when native HTML cannot do the job** — use `<button>` before `role="button"`.
- **Live regions** (`aria-live="polite"` or `"assertive"`) for clinical alerts, validation errors, and async status. Reserve `assertive` for urgent clinical signals.
- **Focus management** for modals, route changes in SPAs, and after async actions.

---

## Accessible Telehealth Video

Video is the highest-stakes accessibility surface in modern healthcare. Plan for it as a core feature, not a checkbox.

- **Captions**:
  - **ASR (auto-captions)** acceptable for casual context but inadequate for clinical communication where errors carry consequence.
  - **CART (Communication Access Realtime Translation)** — human captioner; the standard for high-stakes clinical encounters when requested.
  - Captions should be on by default for users who set the preference; user must be able to resize, reposition, and restyle them.
- **Sign language interpretation**:
  - Route to a qualified interpreter in the patient's language (ASL is not universal — LIBRAS for Brazil, BSL for UK, LSF for France, etc.).
  - Provide a third video tile, pinning, and sufficient resolution; do not force the interpreter into a thumbnail.
  - VRI (Video Remote Interpreting) vendors like LanguageLine, Stratus, Cyracom — verify current capabilities and BAA posture.
- **Audio description** for any pre-recorded patient education embedded in the visit.
- **Visual indicators** of who is speaking, mute state, and connection quality — not color alone.
- **Pre-visit accessibility intake**: ask preferences before the visit and persist them across visits.

---

## Low-Vision, Color, and Contrast

- Meet **1.4.3** (4.5:1 normal text, 3:1 large text) and **1.4.11** (3:1 for UI components and graphical objects) as a baseline; prefer 7:1 (AAA) for body text where possible.
- Support **text resize to 200%** without loss of content or function.
- Support **reflow** at 320 CSS px without horizontal scrolling (1.4.10).
- Respect user-stylesheet overrides; do not lock typography or spacing with `!important`.
- Honor `prefers-color-scheme` (dark mode) and `prefers-contrast` (high contrast).
- Don't carry meaning by color alone — pair color with text, icon, or pattern.
- Test with a real color-blind simulator and with users.

---

## Motor and Cognitive Access

- **Target size**: ≥ 24×24 CSS px (2.2 minimum); ≥ 44×44 for finger-friendly mobile.
- **Pointer cancellation** (2.5.2) — let users abort by moving off before mouseup.
- **No timeouts on consent or clinical decision flows**, or provide a clearly visible extension with audible warning (2.2.1).
- **Predictable layout** — global navigation in the same place across pages (3.2.3).
- **Avoid hover-only menus** — keyboard equivalent is required.
- **Reduce cognitive load**: chunk forms, show progress, save partial entries, allow back-navigation without data loss.
- Respect `prefers-reduced-motion` for animations, parallax, and auto-advancing carousels.

---

## Accessible Forms

Forms are where most healthcare accessibility complaints originate.

- Every input has a programmatic `<label>` (not just placeholder text).
- Errors are programmatically associated to inputs via `aria-describedby` and announced.
- Use `autocomplete` (`given-name`, `family-name`, `tel`, `email`, `street-address`, `postal-code`, `bday`, `language`) — also a Section 508 / WCAG 1.3.5 requirement.
- Group related fields with `<fieldset>` and `<legend>`.
- Don't disable submit while validation runs; instead, let it submit and announce errors.
- Show what's required, not only what's optional, and announce required state to AT.
- Allow paste, autofill, and password managers in all fields, including OTP and MRN.

---

## Older Adults

Healthcare disproportionately serves older adults. Defaults matter.

- Default text size large enough to read at arm's length.
- Plain language (see health-content-writing).
- Avoid hover-only and gesture-only interactions.
- Prefer phone callback fallback alongside chat.
- Don't punish slow input — generous timeouts, large hit targets, clear undo.
- Account for tremor: avoid drag-only interactions; allow keyboard/button alternatives.

---

## Testing Tooling and Process

Automated tools catch roughly 30-40% of WCAG issues; the rest needs humans.

- **Automated**: axe DevTools / axe-core, Lighthouse, WAVE, Accessibility Insights, Pa11y, IBM Equal Access. Integrate into CI.
- **Manual structural review**: heading order, landmarks, focus order, name/role/value spot checks.
- **Manual AT testing**: at minimum one screen reader per major platform (VoiceOver on iOS, TalkBack on Android, NVDA on Windows, VoiceOver on macOS).
- **Keyboard-only walkthrough** of every critical flow (login, schedule, message, complete visit, download record).
- **Zoom and reflow** at 200% and 400%.
- **Color and contrast** check with simulator.
- **Usability testing with people with disabilities** — required for credibility and to surface issues no tool catches. Compensate participants.

---

## VPAT / ACR for Procurement

A VPAT (Voluntary Product Accessibility Template) produces an Accessibility Conformance Report (ACR). It is the lingua franca of accessibility procurement, especially for federal, state, and large health-system buyers.

- Use the **current VPAT edition** matching the buyer's standard (Section 508, WCAG, EN 301 549, or all — INT edition).
- Each criterion gets a conformance level (Supports / Partially Supports / Does Not Support / Not Applicable) **and an explanation**.
- Have a real human auditor sign the ACR; bogus VPATs are a known procurement risk and have led to lawsuits.
- Update the VPAT with each major release; date and version it.

---

## QA Checklist

- [ ] WCAG 2.2 AA target stated and design tokens enforce contrast and target size
- [ ] Keyboard reaches and operates every interactive control; focus is visible
- [ ] Screen reader announces names, roles, values, and live status messages
- [ ] Forms have labels, error association, and `autocomplete` set
- [ ] Authentication does not depend on a cognitive function test
- [ ] Telehealth supports captions and sign-language routing as a first-class feature
- [ ] Reflow at 320 px and zoom to 200% without loss
- [ ] No meaning by color alone; `prefers-reduced-motion` honored
- [ ] Tested with at least one real screen reader on each major platform
- [ ] Usability tested with people with disabilities
- [ ] VPAT/ACR current and signed; procurement language reviewed
- [ ] Plain-language and language-access coverage coordinated with health-content-writing

---

## Task-Specific Questions

1. What WCAG conformance target are we designing or auditing to — 2.1 AA, 2.2 AA, or AAA where feasible? Is the target driven by regulation, procurement (VPAT/ACR), or internal policy?
2. What is the scope of the audit or build — public web portal, authenticated portal, mobile apps (iOS / Android), telehealth video, kiosks (in-clinic check-in / pharmacy / lab), IVR, accessible PDF deliverables?
3. What assistive technologies are in the test matrix — VoiceOver (iOS / macOS), TalkBack (Android), JAWS, NVDA, Narrator, Dragon, Voice Control / Voice Access, Switch Control, magnification, captions, sign-language interpretation routing?
4. Is Section 508 (federal procurement) or Section 1557 (HHS-funded program accessibility plus language access) in scope, and is a VPAT/ACR required by a buyer?
5. What languages are required, and is translation / sign-language interpretation routing in scope alongside accessibility (these intersect at telehealth captions, language access, and accessible authentication)?
6. Do we have access to people with disabilities for user research, and is there a budget to compensate participants (required for credibility and to surface issues no tool catches)?
7. Are there in-scope surfaces the org does not own (EHR vendor screens, third-party telehealth, identity provider) — and what is the plan for raising findings with those vendors?

---

## Related Skills

- **health-content-writing**: plain language and reading level, which underpin cognitive accessibility
- **patient-engagement**: ensures outreach channels and orchestration are accessible end to end
- **patient-portal**: portal-specific UX patterns and AT compatibility
- **telehealth-platform**: real-time video accessibility (captions, ASL routing, audio description)
- **hipaa-compliance**: accessibility and PHI controls intersect at consent, transcripts, and recordings
- **clinical-documentation**: clinician-facing accessibility (high-density screens, voice input, color blindness)
