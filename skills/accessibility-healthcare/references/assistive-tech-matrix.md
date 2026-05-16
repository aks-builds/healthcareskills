# Assistive Technology Matrix for Healthcare

Reference for the assistive technologies that real patients and clinical staff use, and what each implies for testing and design. Verify current product capabilities directly with the vendor and the OS release notes before quoting features in deliverables.

## Contents

- How to Read This Matrix
- Screen Readers
- Magnification
- Switch Access
- Eye Tracking
- Voice Control and Speech-to-Text
- Captions
- Sign-Language Interpretation Routing
- TTY, Relay, and Video Relay
- Refreshable Braille Displays
- Cognitive Assistive Technology
- Test Matrix by Surface
- Common Pitfalls

---

## How to Read This Matrix

Each AT section covers:

- **What it is**
- **Who uses it** (the actual users to consider in healthcare)
- **Healthcare implications** (where it intersects clinical workflow and patient experience)
- **Testing notes** (what to actually do)

Test with the AT users actually use, on the platforms they actually use. Automated scanners cannot substitute.

---

## Screen Readers

| Screen Reader | Platform | Notes |
|---------------|----------|-------|
| VoiceOver | iOS, iPadOS, macOS | Built-in. Default for blind iOS users; widely used in patient populations. |
| TalkBack | Android | Built-in. Default for blind Android users. |
| JAWS | Windows | Commercial. Common in enterprise / employer environments; long the dominant Windows screen reader. |
| NVDA | Windows | Free, open source. Common in QA and many home users. |
| Narrator | Windows | Built-in. Less feature-rich than JAWS/NVDA but improving. |
| Orca | Linux / GNOME | Less common in healthcare but exists in research / lab environments. |

**Healthcare implications**:
- Blind and low-vision patients are over-represented among older adults, diabetics (diabetic retinopathy), and patients with macular degeneration — populations many health systems serve heavily.
- Clinical staff include screen-reader users (medical coders, billing, transcription, some clinicians).
- Form labels, error association, status messages, and dynamic content (results loading, secure-message arriving, video-call ringing) all need to be announced.

**Testing notes**:
- At minimum: VoiceOver on iOS, TalkBack on Android, NVDA on Windows, VoiceOver on macOS. JAWS if enterprise procurement requires.
- Test the full task, not isolated controls — login, schedule a visit, send a secure message, download an AVS, complete a video visit.
- Listen to landmark navigation (Rotor on VoiceOver, swipe gestures on TalkBack). Headings should be logical (one h1, sensible h2/h3 outline).
- Test live regions: do toast notifications and validation errors get announced? Without focus shift?
- Test focus management on modals and SPA route changes.

---

## Magnification

| Tool | Platform | Notes |
|------|----------|-------|
| Zoom | macOS, iOS, iPadOS | Built-in OS-level magnifier. |
| Magnifier | Windows | Built-in. |
| ZoomText | Windows | Commercial, combines magnification + screen reading. |
| Browser zoom | All | 200% and beyond. WCAG 1.4.10 requires reflow at 320 CSS px. |
| Page zoom via OS settings | All | Increases default text size system-wide. |

**Healthcare implications**:
- Older adults often use OS-level text-size increase before they identify as "low-vision."
- Magnification users see a small viewport of the screen at a time; layout matters. Fixed sidebars that always occupy 30% of the screen leave little room for the actual content.

**Testing notes**:
- Test browser zoom to 200% (required) and 400% (good practice).
- Test OS text-size increase on iOS (Dynamic Type) and Android (font-size slider).
- Test with high-contrast / inverted-color modes; do not rely on background images for meaning.
- `prefers-contrast` and `prefers-color-scheme` should be honored.

---

## Switch Access

**What it is**: Single-button or two-button input that scans through interactive elements. Users select with a switch press.

**Who uses it**: Patients with motor disabilities — cerebral palsy, ALS, spinal cord injury, severe Parkinson's. Switch users may interact through head switches, sip-and-puff, foot switches, or eye-tracking-driven switch emulators.

**Healthcare implications**:
- Specialty clinics (neuro, rehab, ALS, MD) see a higher density of switch users.
- Long, complex forms are particularly punishing — every field is many scan cycles.
- Modal dialogs and time-pressured interactions (session timeouts, OTP windows) are blocking.

**Testing notes**:
- iOS Switch Control and Android Switch Access can be enabled for testing — even one external switch (or a Bluetooth keyboard with one mapped key) reveals issues.
- Forms must be navigable in logical order. Skip-links speed things up dramatically.
- No timeouts on consent or clinical decision flows (WCAG 2.2.1) — or provide a clearly visible extension.

---

## Eye Tracking

**What it is**: Camera-based gaze tracking enables users to "click" by dwelling on a target. Commercial systems include Tobii Dynavox; consumer support is growing (iOS Eye Tracking on iPadOS / iOS 18+, verify current OS support).

**Who uses it**: Patients with severe motor disabilities (ALS, locked-in syndrome, late-stage MS, severe cerebral palsy).

**Healthcare implications**:
- Same constraints as switch access — patience for long flows, intolerance for time-pressure.
- Target size (2.5.8) matters more — dwell-to-click requires hitting and resting on the target.

**Testing notes**:
- If accessible to your team, test with an eye tracker. Otherwise, simulate by testing keyboard-only with generous target sizes and zero hover-only interactions.

---

## Voice Control and Speech-to-Text

| Tool | Platform | Notes |
|------|----------|-------|
| Dragon NaturallySpeaking | Windows | Commercial, common in clinical and motor-impairment use. |
| Dragon Medical One | Windows | Healthcare-specific Dragon for clinical documentation. |
| Voice Control | macOS, iOS, iPadOS | Built-in. |
| Voice Access | Android | Built-in. |
| Windows Speech Recognition | Windows | Built-in. |
| Browser dictation | All | Limited but increasing. |

**Healthcare implications**:
- Dragon is widely used by physicians for clinical documentation — the EHR side of accessibility intersects clinical workflow.
- Patients with motor disabilities, RSI, fatigue, or post-stroke aphasia recovery use voice control.
- Voice control depends on programmatic names being announceable — "click sign-in button" only works if there's a button with an accessible name "Sign in."

**Testing notes**:
- Buttons and links need accessible names that match the visible label. Icon-only buttons need ARIA labels users can say out loud.
- Avoid identical names on multiple controls ("Edit" / "Edit" / "Edit") — users need disambiguation.
- Test with macOS or iOS Voice Control with the "Show Numbers" or "Show Grid" overlays — they expose what voice control can target.

---

## Captions

Captions are not optional for telehealth and patient-education video.

| Type | Where | Notes |
|------|-------|-------|
| Auto-captions (ASR) | YouTube, Zoom, Teams, Meet | Acceptable casually; inadequate for clinical content where errors carry consequence. |
| Human captions (CART) | Live, on-demand | Communication Access Realtime Translation. Standard for high-stakes clinical encounters when requested. |
| Edited captions | Pre-recorded video | Human-edited transcripts; the right baseline for patient-education videos. |
| Open captions | Pre-recorded video | Burned into the video; not toggleable. |
| Closed captions | Pre-recorded and live | Toggleable; required as default available. |

**Healthcare implications**:
- Deaf and hard-of-hearing patients (the latter is much larger and grows with age) require captions for video visits.
- Captions also serve patients in loud environments, with English as a second language, with auditory processing differences, and watching without sound in shared spaces.
- Default-on for users who set the preference; preference must persist across visits.
- Users must be able to resize, reposition, and restyle captions.

**Testing notes**:
- Test caption controls (toggle, size, position, color).
- For pre-recorded patient education, audit caption accuracy on clinical terms — drug names, condition names, anatomy.
- For live telehealth, test ASR quality with a sample of accented English and code-switched speech; document where CART is the right answer.

---

## Sign-Language Interpretation Routing

**What it is**: Live video interpretation in the patient's sign language during a telehealth visit (and increasingly in-person via tablet kiosks).

**Languages**: ASL is the most common in the US but **not universal**. Healthcare populations may need:
- **ASL** (American Sign Language)
- **LIBRAS** (Brazilian Sign Language) — significant in some Brazilian-diaspora communities
- **BSL** (British Sign Language) — different from ASL despite shared spoken language
- **LSF** (French Sign Language)
- **Mexican Sign Language (LSM)**, **Tagalog Sign Language**, and many others depending on the served population
- **PSE** / Signed Exact English / cued speech variants — match the patient's preference

**Healthcare implications**:
- VRI (Video Remote Interpreting) vendors commonly used in US healthcare include LanguageLine, Stratus, and Cyracom — verify current capabilities, language coverage, and BAA posture.
- In-person interpreters remain the right answer for high-stakes encounters (consent for major procedures, mental health emergencies, end-of-life conversations).
- The interpreter needs a third video tile, pinning, and sufficient resolution. Do not force the interpreter into a thumbnail.

**Testing notes**:
- Test interpreter routing in your telehealth platform.
- Confirm the patient can pin the interpreter video.
- Confirm interpreter video resolution is high enough to read signing clearly.
- Pre-visit accessibility intake should capture sign-language preference and persist it across visits.

---

## TTY, Relay, and Video Relay

**What it is**: Text-based telephone for deaf and hard-of-hearing users (TTY); operator-assisted relay (711) for connecting voice and text; Video Relay Service (VRS) for sign-language users to phone calls via interpreter.

**Healthcare implications**:
- Any voice channel (IVR, phone callback, after-hours line) must support 711 relay and TTY.
- VRS calls may appear from an unfamiliar caller ID; staff need to be trained not to hang up.
- ADA Title II and III have specific requirements for effective communication via TRS.

**Testing notes**:
- Verify the practice's appointment and triage lines work with 711 relay (test by calling 711 and asking the operator to dial your test number).
- Document accepted call types in patient-facing materials (TTY, VRS, voice).

---

## Refreshable Braille Displays

**What it is**: Hardware that renders text content (from the screen reader) as Braille via refreshable pins. Connects via USB or Bluetooth to phone or computer.

**Who uses it**: Some blind users — particularly deaf-blind users, professionals, and students. Less common than screen-reader-only use but high-impact when applicable.

**Healthcare implications**:
- Deaf-blind patients exist; for high-stakes encounters, plan an interpreter (tactile signing or pro-tactile) alongside accessible content.
- Braille users read at the rate of synthesized speech — they're not slower; they often catch errors auditory listeners miss.

**Testing notes**:
- Screen-reader-friendly content is generally Braille-friendly. Long unbroken paragraphs are harder; structured content (headings, lists) helps.

---

## Cognitive Assistive Technology

Less standardized than sensory and motor AT, but a real population.

- **Reminders and task-management** — patients with TBI, ADHD, early dementia.
- **Reading support** — text-to-speech tools (Read&Write, Natural Reader), reading guides, dyslexia-friendly fonts (OpenDyslexic, Lexie Readable).
- **Simplified-language tooling** — emerging.
- **Plain language** itself is cognitive AT for everyone (see health-content-writing).

**Healthcare implications**:
- Cognitive accessibility intersects plain-language work directly.
- Long forms with no save-and-resume punish patients with cognitive disabilities.
- Avoid timeouts on consent or clinical-decision flows.

**Testing notes**:
- Test text-to-speech on critical content (consent, AVS, medication instructions).
- Test save-and-resume on long forms.

---

## Test Matrix by Surface

| Surface | Minimum AT to test |
|---------|---------------------|
| Patient portal (web) | NVDA on Windows, VoiceOver on macOS, keyboard-only, 200% zoom |
| Patient portal (mobile web / app) | VoiceOver on iOS, TalkBack on Android, Dynamic Type max, Switch Control sample |
| Telehealth video | VoiceOver on iOS, TalkBack on Android, captions on/off, sign-language interpreter routing, keyboard for desktop |
| Telehealth video (clinician side) | Same plus Dragon (if used clinically) |
| Kiosk (check-in tablet) | TalkBack (Android kiosks), VoiceOver (iPad kiosks), large-touch-target test, voice control if supported |
| IVR | 711 relay test call, TTY test, multilingual voice menu test |
| Accessible PDF (AVS, consent) | VoiceOver / NVDA on the PDF; check tagged-PDF structure |
| Email | Render in plain-text and HTML; screen-reader test of the HTML; alt text on key images |
| SMS | Plain text; no AT-specific testing beyond confirming carrier rendering |

---

## Common Pitfalls

1. **Testing only with one screen reader and one OS**. Behaviors differ; each combination surfaces different issues.
2. **Hover-only interactions** — fails keyboard, switch, voice, and touch.
3. **Inaccessible authentication** — math CAPTCHA, type-the-letter, OTP forms that disable paste.
4. **Time pressure** — session timeouts that fire mid-task on consent, OTP, or scheduling flows.
5. **Color-only meaning** — universal failure across AT users (color blindness is the most common reason; screen-reader users get nothing at all).
6. **Long forms with no save-and-resume** — punishing for cognitive AT and switch users.
7. **Captions only as ASR** for clinical telehealth — error rates make this clinically unsafe.
8. **One sign language assumed** — ASL is not universal; route in the patient's actual language.
9. **Procurement of inaccessible vendor systems** without a VPAT review.
10. **No compensation for usability participants with disabilities**. Always pay.
