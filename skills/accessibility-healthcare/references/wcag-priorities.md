# WCAG 2.2 AA Priorities for Healthcare

Reference for the success criteria most commonly missed in healthcare audits, with healthcare-specific guidance for each. Use this to triage an audit, scope remediation, and prioritize design-system fixes. Verify all regulatory citations and WCAG versions against current sources before quoting.

## Contents

- Why These Criteria
- Perceivable
  - 1.3.5 Identify Input Purpose
  - 1.4.1 Use of Color
  - 1.4.3 Contrast (Minimum)
  - 1.4.10 Reflow
  - 1.4.11 Non-text Contrast
  - 1.4.12 Text Spacing
- Operable
  - 2.1.1 Keyboard
  - 2.4.7 Focus Visible
  - 2.4.11 Focus Not Obscured (Minimum) — 2.2
  - 2.5.7 Dragging Movements — 2.2
  - 2.5.8 Target Size (Minimum) — 2.2
- Understandable
  - 3.2.6 Consistent Help — 2.2
  - 3.3.2 Labels or Instructions
  - 3.3.7 Redundant Entry — 2.2
  - 3.3.8 Accessible Authentication (Minimum) — 2.2
- Robust
  - 4.1.2 Name, Role, Value
  - 4.1.3 Status Messages
- Patterns That Fail Multiple Criteria At Once
- Triage Order
- Common Pitfalls

---

## Why These Criteria

WCAG 2.2 has 50+ success criteria. In healthcare audits, the same dozen come up repeatedly:

- Healthcare UIs are dense, which strains contrast, target size, and focus visibility.
- Clinical workflows include long forms (intake, history, consent), which strain labels, error handling, and redundant entry.
- Authentication is often heavy (MFA, MRN lookup, security questions), which strains accessible authentication and autocomplete.
- Real-time clinical content (alerts, results, vitals) needs live regions, which are frequently missing or misused.
- Telehealth video adds captions, sign-language interpretation, and audio description on top of every other surface.

This file prioritizes those criteria. AAA is desirable where feasible (notably 1.4.6 enhanced contrast for body text); design for it where the population skews older or low-vision.

---

## Perceivable

### 1.3.5 Identify Input Purpose

Forms must use `autocomplete` on inputs that match the standard taxonomy (`given-name`, `family-name`, `email`, `tel`, `street-address`, `postal-code`, `bday`, `language`, `username`, `current-password`, `new-password`, `one-time-code`).

**Healthcare gotchas**:
- MRN, member ID, plan ID — no standard `autocomplete` value; use `inputmode` and consider custom data attributes.
- `one-time-code` for OTP fields enables OS-level autofill; never disable it.
- Allow paste in all fields including OTP; many users rely on password managers.

### 1.4.1 Use of Color

Color cannot be the only way to convey information, indicate an action, prompt a response, or distinguish a visual element.

**Healthcare gotchas**:
- Lab results: don't show "abnormal" by color alone. Add a label, an icon, or both.
- Form validation: don't show errors with red border only. Add text and an icon.
- Status indicators (online / offline, scheduled / canceled, urgent / routine): pair color with shape or text.
- Triage acuity (ESI levels, MEWS scores): always paired with number and label.

### 1.4.3 Contrast (Minimum)

Text needs at least 4.5:1 contrast against its background (3:1 for large text — 18pt regular or 14pt bold). AAA (1.4.6) raises this to 7:1 / 4.5:1.

**Healthcare gotchas**:
- Gray-on-gray placeholder text in clinical UIs routinely fails.
- Telehealth waiting-room overlays on busy video backgrounds fail.
- Disabled state styling that drops to <3:1 is a common audit finding (and usually fine to leave at 3:1 if the control is genuinely inert and conveys disabled state another way).
- Print AVS designs need to be checked too — the screen color profile is not the same as the printer.

Enforce contrast at the design-token level so it can't drift; pair with automated checks in CI.

### 1.4.10 Reflow

Content must reflow at 320 CSS px width without horizontal scrolling, except for content where two-dimensional layout is essential (data tables, maps, complex visualizations).

**Healthcare gotchas**:
- Clinical dashboards and EHR overlays routinely break at 320px. Provide a mobile-specific layout or a documented exception.
- Vital-signs charts may legitimately require two-dimensional layout; flag and document, don't force-fit.
- DICOM viewers and medical images are exempt for the image itself; the surrounding chrome is not.

### 1.4.11 Non-text Contrast

3:1 minimum contrast for UI components (button borders, form field borders, focus indicators) and meaningful graphical objects (icons that convey information).

**Healthcare gotchas**:
- "Ghost" buttons and outline-only icon buttons frequently fail.
- Focus rings stripped to 1px light blue fail.
- Required-field asterisks rendered as faint orange fail.

### 1.4.12 Text Spacing

Users must be able to override text spacing (line height 1.5×, paragraph spacing 2× font size, letter spacing 0.12×, word spacing 0.16×) without content loss.

**Healthcare gotchas**:
- Fixed-height containers with text that clips when spacing is increased.
- Tooltips and notifications that get cut off at the bottom.

---

## Operable

### 2.1.1 Keyboard

Every interactive element must be reachable and operable by keyboard. No keyboard traps.

**Healthcare gotchas**:
- Custom date pickers, time pickers, dose-spinners: keyboard support is often half-built.
- Drag-and-drop ordering of medications, problems, or schedule slots: must have a keyboard alternative (move up / down buttons).
- Modal dialogs that trap focus inside them when intended, but also escape-key dismissal.
- Telehealth video controls (mute, camera, hang-up, raise-hand) routinely keyboard-inaccessible.

### 2.4.7 Focus Visible

Keyboard focus must be visually apparent.

**Healthcare gotchas**:
- CSS resets that remove default focus rings and don't replace them.
- Custom focus styles that fail 1.4.11 (non-text contrast 3:1).
- Focus rings hidden inside dense layouts because there's no padding.
- Use `:focus-visible` to scope rings to keyboard users without showing them on every click.

### 2.4.11 Focus Not Obscured (Minimum) — WCAG 2.2

When focused, the element must not be entirely obscured by author-created content (sticky headers, sticky footers, sticky toasts).

**Healthcare gotchas**:
- Sticky bottom-of-screen "Save / Submit" bars covering the focused field they're meant to operate on.
- Sticky top notifications obscuring the top of the focused element.
- Solution: ensure scroll-into-view padding or use `scroll-padding`.

### 2.5.7 Dragging Movements — WCAG 2.2

Functionality that uses dragging must offer a non-dragging alternative.

**Healthcare gotchas**:
- Pain scales that rely on dragging a slider — provide button steppers.
- Medication-reordering drag handles — provide up/down buttons.
- Signature capture — provide an alternative (typed name, e-sign service).

### 2.5.8 Target Size (Minimum) — WCAG 2.2

Pointer target size must be at least 24×24 CSS px, with exceptions for inline links and equivalent controls available elsewhere.

**Healthcare gotchas**:
- Dense clinical icon buttons (16×16 or 20×20) fail. Add padding to reach 24×24 hit target even if the visual icon stays small.
- Mobile / tablet recommended target: 44×44 (Apple), 48×48 (Material) — this is good UX, not strictly required at AA.
- Calendar / time picker cells in scheduling routinely fail.

---

## Understandable

### 3.2.6 Consistent Help — WCAG 2.2

When help mechanisms (contact info, help link, chat, FAQ) appear on multiple pages, they must occur in the same relative order.

**Healthcare gotchas**:
- "Contact us" link in the footer on most pages but in a side menu on the scheduling page — fail.
- Help chat widget on every page except the secure-message thread — flag the inconsistency.

### 3.3.2 Labels or Instructions

Inputs need labels or instructions.

**Healthcare gotchas**:
- Placeholder-only forms (placeholder text disappears on focus, screen readers may or may not announce, color usually fails contrast).
- Required-field convention: show what is required, not what is optional, and announce required state programmatically (`aria-required`).
- Insurance / pharmacy / referral forms with cryptic field labels ("Group #", "BIN", "PCN") need supplementary instructions.

### 3.3.7 Redundant Entry — WCAG 2.2

Information previously entered in the same process must be auto-populated or available for selection — except passwords, security-sensitive info, or when the information has changed.

**Healthcare gotchas**:
- Re-entering demographics on every intake form — fail.
- Re-entering insurance on every form when it's already on file — fail.
- Solution: pre-fill from the EHR / portal profile; let the patient confirm or edit.

### 3.3.8 Accessible Authentication (Minimum) — WCAG 2.2

Authentication must not require a cognitive function test (memorization, transcription, calculation) unless an alternative is available or the test is the user's own object (recognize own face).

**Healthcare gotchas**:
- Math CAPTCHAs — fail.
- "Type the third letter of your security question" — fail.
- Distorted-text CAPTCHA without alternative — fail.
- OTP forms that disable paste / autofill — fail (forces transcription).
- **Acceptable**: passkeys / WebAuthn, MFA-via-device (push), biometric, magic-link, password managers (allow paste), SSO. Audio CAPTCHA alone is not sufficient — deaf-blind users.

---

## Robust

### 4.1.2 Name, Role, Value

Every interactive component must expose its name (label), role (button, link, checkbox, etc.), and current value (state, selection) to assistive technology.

**Healthcare gotchas**:
- Custom controls built from `<div>` and `<span>` without ARIA — fail.
- Toggle buttons without `aria-pressed`.
- Combobox / autocomplete patterns with incorrect ARIA — fail in a way that automated tools may not catch.
- **Rule of thumb**: use native HTML before reaching for ARIA. `<button>` before `role="button"`. ARIA only when native can't do the job.

### 4.1.3 Status Messages

Status messages must be programmatically determinable through role or properties so AT can announce them without focus shift.

**Healthcare gotchas**:
- Toast notifications, save confirmations, and async loading states not announced — fail.
- Use `role="status"` or `aria-live="polite"` for routine status, `aria-live="assertive"` only for urgent clinical signals (test results, dose alerts).
- Avoid duplicate announcements (don't both shift focus *and* set aria-live).

---

## Patterns That Fail Multiple Criteria At Once

A single bad pattern often hits several criteria:

- **Placeholder-only forms**: 3.3.2 (labels), 1.4.3 (contrast), often 4.1.2 (name).
- **Custom dropdowns built from divs**: 2.1.1 (keyboard), 4.1.2 (name/role/value), 4.1.3 (status), 2.5.8 (target size).
- **Sticky bottom CTA bar in mobile**: 2.4.11 (focus not obscured), sometimes 1.4.10 (reflow).
- **Color-only status indicators**: 1.4.1 (use of color), often 1.4.3 (contrast) and 1.4.11 (non-text contrast).
- **Modal dialogs without focus management**: 2.1.1 (keyboard), 2.4.3 (focus order), 4.1.2 (name/role/value).
- **Math CAPTCHA / distorted-text CAPTCHA**: 3.3.8, often 1.4.3 / 1.4.11.

Fix the pattern, not the individual finding.

---

## Triage Order

When you have a long audit findings list and limited remediation capacity:

1. **Authentication and core flows first**. If users can't log in or complete a primary task, nothing else matters.
2. **Forms that block clinical or financial action** (intake, consent, scheduling, payment).
3. **Name/role/value gaps** on widely used components — design-system fixes ripple across surfaces.
4. **Live regions** for clinical alerts and validation errors.
5. **Contrast and focus visibility** — enforce at design-token level so they stay fixed.
6. **Keyboard parity** for video, drag-and-drop, custom widgets.
7. **WCAG 2.2 additions** (target size, focus not obscured, redundant entry, accessible authentication) — usually surface in design-system changes.
8. **Color-as-meaning** — easy fix once flagged.
9. **Reflow and zoom** — usually require component or layout-level work.
10. **Telehealth captioning and ASL routing** — high-stakes, treat as a feature, not a checkbox.

---

## Common Pitfalls

1. **Treating automated scans as sufficient**. They catch 30-40% of WCAG issues; the rest needs humans and real AT.
2. **Fixing one finding without fixing the pattern**. A broken custom dropdown appears in 50 places; fix the component.
3. **Adding ARIA to broken native HTML** instead of using the right native element.
4. **Confusing "alert" with "assertive live region"**. Reserve assertive for urgent clinical signals.
5. **Removing focus indicators** in CSS resets and not replacing them.
6. **Disabling paste / autofill in OTP and MRN fields** — fails 3.3.8 and 1.3.5 simultaneously.
7. **Not testing with real users with disabilities**. Compensate them.
8. **Skipping the design system**. Fix tokens and components; the surfaces inherit.
