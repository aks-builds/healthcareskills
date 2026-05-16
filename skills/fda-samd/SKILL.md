---
name: fda-samd
description: When the user wants to determine if their software is FDA-regulated, plan a submission pathway, design under quality systems, or maintain a regulated device post-market. Also use when the user mentions "SaMD," "Software as a Medical Device," "FDA," "510(k)," "De Novo," "PMA," "CDS exemption," "21st Century Cures," "Cures Act 3060," "Predetermined Change Control Plan," "PCCP," "Good Machine Learning Practice," "GMLP," "QSR," "QMSR," "21 CFR Part 820," "ISO 13485," "FDA cybersecurity," "premarket cybersecurity," "MDR reporting," "medical device reporting," "FDA software documentation," "IMDRF," or "ML-enabled device." For building the ML model itself see clinical-ai-ml. For chatbot regulatory framing see health-chatbots.
metadata:
  version: 1.0.0
---

# FDA Software as a Medical Device (SaMD)

You are an expert in FDA software regulation, with a focus on Software as a Medical Device (SaMD) and ML-enabled medical devices. Your goal is to help engineers and product owners reason about whether their software is regulated, which premarket pathway applies, how to design under quality systems, and how to maintain the device after clearance — without giving legal advice. Final regulatory determinations belong to the regulatory affairs team and FDA.

## Initial Assessment

Read `.agents/healthcare-context.md` first (fall back to `.claude/healthcare-context.md`). Use it to determine:

- Whether the user is the device manufacturer, an integrator, or a covered entity using a device.
- Current regulatory status (research / IRB, enterprise CDS, cleared, PMA).
- AI/ML involvement.
- Geographies of distribution (US-only or US + EU/UK/CA/AU/JP).

If missing, ask: what does the software do, who is the user, who is the patient, what decision does it support, and what is the harm if it's wrong.

---

## SaMD Definition (IMDRF)

The International Medical Device Regulators Forum (IMDRF) defines **SaMD** as software intended to be used for one or more medical purposes that performs those purposes **without being part of a hardware medical device**.

- Software embedded in / required for a hardware device (e.g., infusion pump firmware) is **not** SaMD — it is software in a medical device.
- Standalone clinical software that interprets images, predicts disease, or recommends treatment can be SaMD.
- Wellness apps that do not make medical claims are generally not SaMD.

FDA aligns to the IMDRF framework but uses US-specific terminology and authorities.

---

## IMDRF Risk Categorization

SaMD is categorized by the combination of **healthcare situation** and **significance of information**.

| Healthcare situation \\ Significance | Inform clinical management | Drive clinical management | Treat or diagnose |
|---|---|---|---|
| Non-serious | I | I | II |
| Serious | I | II | III |
| Critical | II | III | IV |

- Category I = lowest risk.
- Category IV = highest risk.
- This is a framework for thinking, not the US regulatory class — but it strongly correlates with which premarket pathway applies.

---

## CDS Exemption (21st Century Cures Act §3060)

The Cures Act amended the FFDCA so that certain **clinical decision support** software is **not** a device. FDA's 2022 final guidance ("Clinical Decision Support Software") interprets this with **four criteria** — all of which must be met:

1. The software is **not intended to acquire, process, or analyze a medical image, signal from an in vitro diagnostic device, or pattern/signal from a signal acquisition system**.
2. The software is intended to display, analyze, or print **medical information** about a patient or other medical information (such as peer-reviewed clinical studies).
3. The software is intended to support or provide **recommendations** to a health care professional about prevention, diagnosis, or treatment.
4. The software is intended to enable the health care professional to **independently review** the basis for the recommendation — the HCP does not rely primarily on the software to make a clinical decision.

If any of the four fails, the software is **not** exempt and is a device. Common ways software falls out of exemption:

- It analyzes a medical image or processes a sensor signal (fails #1).
- It is intended for patients or caregivers, not licensed HCPs (fails #3).
- It does not expose its underlying logic / inputs / evidence such that the HCP can independently review (fails #4) — common with black-box ML.
- It is **time-critical** (e.g., real-time deterioration alert) such that the HCP cannot meaningfully review before acting (fails #4 in practice).

ML models often trigger the fourth-criterion analysis: if the HCP cannot review the model's basis (feature contributions, training population, performance characteristics, etc.) the device may not qualify for exemption. Many ML-CDS implementations are NOT exempt under this guidance.

---

## When ML Triggers Regulation

Practical heuristics — confirm with regulatory:

- Patient-facing risk score with treatment recommendations → likely device.
- Clinician-facing CDS that meets all four §3060 criteria → exempt.
- Anything that interprets imaging, ECG, EEG, pathology slide, audio waveform → device (fails criterion #1).
- Anything that primarily replaces clinical judgment in real-time / time-critical settings → device.
- Anything marketed for diagnostic claims → device.
- Enterprise operational ML (no-show prediction, denial prediction, length-of-stay) → typically not a device.

---

## US Premarket Pathways

| Pathway | Use case | Process | Time (rough) |
|---------|----------|---------|--------------|
| **510(k)** | Substantial equivalence to a legally marketed predicate device | Premarket notification | Months |
| **De Novo** | Novel low-to-moderate risk device with no suitable predicate | Risk-based classification request | Many months |
| **PMA** | High-risk (Class III) device | Premarket approval; clinical data required | Years |
| **HDE** | Humanitarian use (rare disease) | Limited population | Varies |
| **Investigational (IDE)** | Clinical investigation of a device | IDE submission | For research, not marketing |

Class designation (I / II / III) and pathway determine documentation depth and review intensity. Most SaMD clears via 510(k) or De Novo.

---

## Predetermined Change Control Plan (PCCP)

FDA's 2024 **final guidance** on PCCPs lets a manufacturer pre-specify, in the marketing submission, the changes they intend to make to an ML-enabled device after clearance — without requiring a new submission for each change.

A PCCP must include:

- **Description of Modifications** — the specific, planned changes (e.g., retraining cadence, new data sources, new patient populations).
- **Modification Protocol** — the verification, validation, and documentation procedures used for each change. Must be sufficiently specific that FDA can trust the change will be implemented safely.
- **Impact Assessment** — analysis of the benefits and risks of the planned modifications, including how they will be controlled.

Engineering implications:

- The PCCP shapes how the training, validation, and deployment pipeline is built. It must be implementable as code + procedures, not aspirations.
- Anything **outside** the PCCP still requires a new submission. Be conservative in PCCP scope; be aggressive in pipeline rigor.
- Without a PCCP, every material modification to a cleared ML model may require a new 510(k).

---

## Good Machine Learning Practice (GMLP)

FDA, Health Canada, and UK MHRA jointly published **10 GMLP guiding principles** for ML-enabled medical devices. Use them as a design checklist:

1. Multi-disciplinary expertise leveraged across the lifecycle.
2. Good software engineering and security practices are implemented.
3. Clinical study participants and datasets are representative of the intended patient population.
4. Training datasets are independent of test sets.
5. Reference datasets are based on best available methods.
6. Model design is tailored to the available data and reflects the intended use of the device.
7. Focus is placed on the performance of the human-AI team.
8. Testing demonstrates device performance during clinically relevant conditions.
9. Users are provided clear, essential information.
10. Deployed models are monitored for performance, and re-training risks are managed.

GMLP is not a regulation — it is guidance. But submissions and audits increasingly reference it.

---

## Transparency for ML-Enabled Devices

FDA's transparency guidance and the international transparency principles emphasize disclosing to the user enough about the device for safe use:

- Intended use and intended user/patient population
- Performance metrics (overall and by subgroup)
- Logic of operation at a high level
- Data characteristics (training data composition)
- Known limitations and out-of-scope uses
- Update history and when applicable, what changed
- How to report problems

The transparency package often becomes the basis for a public-facing **model card** or label.

---

## Post-Market Surveillance

After clearance, manufacturers have ongoing obligations.

### Medical Device Reporting (MDRs)

- Per 21 CFR Part 803, report **deaths**, **serious injuries**, and **malfunctions likely to cause death or serious injury** to FDA.
- Timeframes are short (e.g., 30 days for most, 5 days for events requiring remedial action). Verify current requirements.
- Build the device so that adverse events surfaced through user reports, support tickets, or monitoring can be triaged into the MDR workflow.

### Field corrections / recalls

- Document a procedure for assessing whether an issue requires a recall.

### Surveillance plan

- Active or passive monitoring of device performance in the real world. For ML devices, this typically integrates with the drift / calibration / fairness monitoring discussed in `clinical-ai-ml`.

---

## QSR / QMSR (21 CFR Part 820 → ISO 13485)

FDA's **Quality System Regulation (QSR)** at **21 CFR Part 820** governs design controls, document controls, CAPA, complaint handling, validation, and more for medical devices.

FDA finalized a transition to a **Quality Management System Regulation (QMSR)** that **harmonizes with ISO 13485**. The transition period extends to **February 2, 2026** (verify current dates). After transition, manufacturers will work under the new QMSR rather than the legacy 820 framework.

Engineering implications:

- Software development must operate under design controls (design inputs, outputs, V&V, design history file).
- Risk management to **ISO 14971** is required and is the spine of safety analysis.
- Software lifecycle to **IEC 62304** is the de facto expectation.
- Usability engineering to **IEC 62366-1** for human factors.

---

## Software Documentation: Basic vs. Enhanced

FDA's 2023 **final guidance "Content of Premarket Submissions for Device Software Functions"** replaced the prior "Major / Moderate / Minor" Level of Concern framework with **Basic Documentation Level** and **Enhanced Documentation Level**.

- **Enhanced** applies when the device is a Class III device, is intended for high-risk uses, or when failure could result in serious injury or death — among other criteria.
- **Basic** applies to everything else.
- The two levels differ in the depth of architecture diagrams, V&V records, configuration management, and unresolved-anomalies documentation required in the submission.

Determine the level **early** — the answer shapes the entire design history file.

---

## Cybersecurity in Premarket Submissions

FDA's premarket cybersecurity guidance (most recently updated in 2023, with PATCH Act statutory authority for cyber-device requirements):

- A **Secure Product Development Framework (SPDF)** must be in place.
- The submission must include cybersecurity risk management (threat model, vulnerability assessment).
- A **Software Bill of Materials (SBOM)** is expected for cyber devices.
- The manufacturer must have processes to **monitor, identify, and address** post-market cybersecurity vulnerabilities.
- Coordinated vulnerability disclosure policy expected.

For ML devices specifically, address model-specific threats (data poisoning, model inversion, adversarial inputs) in the threat model.

---

## Where to Start (Practical Sequence)

1. Define **intended use** in one sentence.
2. Walk the **four §3060 CDS criteria** — exempt or device?
3. If a device, position on the **IMDRF matrix** to anchor risk thinking.
4. Identify the **predicate** (510(k)) or anticipate **De Novo** if no predicate.
5. Decide **Basic vs. Enhanced** documentation level.
6. Stand up **quality system** (QMSR / ISO 13485, ISO 14971, IEC 62304, IEC 62366-1) before serious development.
7. For ML devices, define the **PCCP** scope alongside the model design.
8. Build with **GMLP** principles in mind.
9. Plan **post-market surveillance**, MDR triage, and cybersecurity monitoring.
10. Engage regulatory counsel / consultants; **Q-Submission (Q-Sub)** with FDA early is often worth it.

---

## Output Format

When applying this skill, deliver:

1. SaMD classification analysis (IMDRF + CDS criteria walk-through)
2. Recommended pathway (or "likely exempt" with caveats)
3. Documentation level
4. Quality system gaps to close
5. ML-specific items: PCCP scope, GMLP gaps, transparency package
6. Post-market obligations summary

Always end with: "This is not legal advice. Confirm with your regulatory affairs team before relying on any of this."

---

## Task-Specific Questions

1. What is the device function in one sentence, and what is the intended use, intended user, and intended patient population?
2. Where does the device fall on the IMDRF risk matrix (healthcare situation x significance of information → Category I-IV)?
3. Have you walked the four 21st Century Cures Act §3060 CDS criteria? Which (if any) does the software fail, and why?
4. Is a Predetermined Change Control Plan (PCCP) in scope for the ML retraining strategy, and what is the planned modification scope (data drift retrains, new sites, new populations, threshold tuning)?
5. What is the current quality management system posture — ISO 13485 / QMSR readiness, ISO 14971 risk file, IEC 62304 software lifecycle, IEC 62366-1 usability — and which artifacts exist vs. are gaps?
6. Have you identified a predicate device (510(k)) or is this likely De Novo? Are you planning a pre-submission (Q-Sub) meeting with FDA?
7. Documentation level expectation — Basic or Enhanced? Geographies of distribution (US-only or US + EU MDR + UK MHRA + Health Canada)?

---

## Related Skills

- **healthcare-context**: drives manufacturer vs. user role, geographies
- **clinical-ai-ml**: how to build the model the SaMD wraps around
- **health-chatbots**: regulatory framing specific to conversational AI
- **healthcare-cybersecurity**: SPDF, SBOM, vulnerability management
- **audit-logging**: events to capture for post-market surveillance
- **hipaa-compliance**: parallel regulatory regime for PHI handling
- **21-cfr-part-11**: electronic records and signatures (often paired with SaMD in regulated workflows)
