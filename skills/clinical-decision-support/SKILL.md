---
name: clinical-decision-support
description: When the user wants to design, build, deploy, or evaluate a clinical decision support (CDS) tool. Also use when the user mentions "CDS," "CDS Hooks," "patient-view hook," "order-sign hook," "order-select hook," "encounter-start," "appointment-book," "CDS card," "alert fatigue," "interruptive alert," "5 rights of CDS," "order set," "clinical pathway," "CQL," "Clinical Quality Language," "BPA," "best practice advisory," "HHS CDS exemption," "Cures Act CDS criteria," or "FDA SaMD CDS." For SMART app OAuth, see smart-on-fhir. For order entry plumbing, see cpoe-orders. For SaMD regulatory pathway, see fda-samd.
metadata:
  version: 1.0.0
---

# Clinical Decision Support

You are an expert in clinical decision support (CDS) design and implementation. Your goal is to help build CDS that clinicians actually use — meaning it lands in the workflow at the right moment, with the right person, in the right format, and does not contribute to alert fatigue. CDS that is technically correct but ignored is, in practice, broken.

## Initial Assessment

Read `.agents/healthcare-context.md` first (fall back to `.claude/healthcare-context.md`). The context file tells you the EHR vendor (which dictates CDS Hooks support, BPA tooling, and order-set platforms), the clinical setting (inpatient vs. ambulatory vs. ED), and the regulatory posture (whether the tool is positioned as HHS-exempt CDS, an enterprise tool, or an FDA-cleared SaMD).

If the context file does not exist, ask:

1. What is the clinical decision being supported? Who makes it, in what setting?
2. What action should the CDS suggest, and what is the workflow consequence?
3. Which EHR(s)?
4. Is the CDS intended to be HHS-exempt CDS, or are you positioning it as a SaMD?
5. Will it be interruptive (modal) or passive (sidebar / FYI)?

---

## The 5 Rights of CDS

Every CDS evaluation should run through these 5 dimensions. If any one is wrong, the CDS will fail — even if the underlying logic is correct.

| Right | Question |
|-------|----------|
| Right **information** | Is the recommendation evidence-based, current, specific, and actionable? |
| Right **person** | Does it go to the clinician who can act on it (not every clinician who touches the chart)? |
| Right **format** | Modal, side panel, infobutton, banner, order set, dashboard — chosen to match urgency? |
| Right **channel** | EHR alert, secure message, email, page, mobile push? |
| Right **time** | At a moment when the clinician can act — not 12 hours after the decision was already made? |

The fifth right (right time) is where most CDS fails — fired too early (before the order is being placed), too late (after the order is signed), or all the time (no contextual filter).

---

## CDS Hooks

**CDS Hooks** is the HL7 standard that lets external services inject recommendations into the EHR at specific workflow moments. It is the canonical integration pattern for third-party CDS on modern EHRs.

### Hook types (CDS Hooks 1.0+)

| Hook | Fires when | Typical use |
|------|-----------|-------------|
| `patient-view` | Clinician opens a patient chart | Risk scores, gaps in care, screening reminders |
| `order-select` | An order is being chosen (not yet signed) | Suggest alternatives, show cost, show formulary status |
| `order-sign` | Orders about to be signed | Final safety checks — DDI, dose, duplicate, allergy |
| `encounter-start` | A new encounter begins | Show overdue preventive care, intake forms |
| `encounter-discharge` | Discharge is being prepared | Discharge med rec, follow-up gaps, transition-of-care risks |
| `appointment-book` | An appointment is being scheduled | Suggest a different visit type, prior auth check |

Always check the current CDS Hooks spec for the most recent hook list — additional hooks (e.g., `medication-prescribe` legacy, `patient-discharge` deprecated names) appear in older docs.

### Service discovery

A CDS Hooks service exposes a discovery endpoint:

```http
GET /cds-services
```

Response:

```json
{
  "services": [
    {
      "hook": "patient-view",
      "title": "Diabetes Gap Closer",
      "id": "diabetes-gap-closer",
      "description": "Flags patients with diabetes who are overdue for HbA1c or eye exam.",
      "prefetch": {
        "patient": "Patient/{{context.patientId}}",
        "conditions": "Condition?patient={{context.patientId}}&clinical-status=active",
        "observations": "Observation?patient={{context.patientId}}&category=laboratory&_sort=-date&_count=20"
      }
    }
  ]
}
```

The EHR fetches the prefetched FHIR data and bundles it with the hook request, so the service does not need a separate SMART token for read.

### Hook request structure

```json
{
  "hook": "patient-view",
  "hookInstance": "d1577c69-dfbe-44ad-ba6d-3e05e953b2ea",
  "fhirServer": "https://ehr.example.org/fhir/R4",
  "fhirAuthorization": { ... },
  "context": {
    "userId": "Practitioner/example-1",
    "patientId": "example-1",
    "encounterId": "example-enc-1"
  },
  "prefetch": { "patient": { ... }, "conditions": { ... } }
}
```

### Card structure (the response)

```json
{
  "cards": [
    {
      "summary": "HbA1c overdue (last result 9 months ago)",
      "indicator": "warning",
      "detail": "Patient has type 2 diabetes; last HbA1c was 9 months ago. Guideline recommends every 3-6 months for patients above target.",
      "source": {
        "label": "ADA Standards of Care",
        "url": "https://professional.diabetes.org/standards-of-care"
      },
      "suggestions": [{
        "label": "Order HbA1c",
        "actions": [{
          "type": "create",
          "description": "Order HbA1c lab",
          "resource": { "resourceType": "ServiceRequest", "...": "..." }
        }]
      }],
      "links": [{
        "label": "Open diabetes flowsheet",
        "url": "https://app.example.com/diabetes-flowsheet?patient={{patientId}}",
        "type": "smart"
      }]
    }
  ]
}
```

Indicators are `info`, `warning`, or `critical`. Reserve `critical` for true safety alerts — overuse erodes the entire CDS surface.

### Authorization

CDS Hooks 1.0+ supports the EHR passing a short-lived FHIR access token to the CDS service in `fhirAuthorization`, so the service can fetch additional FHIR data scoped to that user/patient. Treat this like any other SMART token — never store it beyond the request.

---

## Alert Fatigue

Alert fatigue is the single largest threat to CDS effectiveness. Override rates above 90% are common; some studies report 96%+. Every false-positive alert trains the user to dismiss the next one.

### Reducing alert fatigue

| Tactic | How |
|--------|-----|
| Raise the threshold | Only fire when severity, evidence, and actionability all justify interrupting. |
| Move to passive surfaces | Sidebar, banner, dashboard — not modal. |
| Suppress per session/encounter | If the alert fired once, don't re-fire the same hook instance. |
| Suppress per patient | If overridden with rationale yesterday, don't re-fire today unless context changed. |
| Tier by role | Some alerts go to pharmacists, not prescribers. |
| Localize | Tune thresholds per service line, per care setting. |
| Measure and prune | Track fire rate, accept rate, override-with-reason. Retire any alert below 5-10% acceptance. |

### Interruptive vs. non-interruptive

| Style | Use when | Examples |
|-------|----------|----------|
| Interruptive (modal) | Imminent patient harm; cannot proceed without acknowledgment | Severe DDI, contraindicated dose, allergy to ordered med |
| Non-interruptive | Useful but not safety-critical | Gap-in-care reminders, cost transparency, formulary status, risk scores |

Default to non-interruptive. Earn the modal.

---

## CQL (Clinical Quality Language)

**CQL** is the HL7 standard for authoring clinical logic. It is used by eCQMs (electronic clinical quality measures), CDS Connect artifacts, and many CDS Hooks services.

CQL separates **logic** from **the engine that runs it** and **the data model** (typically FHIR via QI-Core / US Core). The same CQL library can run against multiple FHIR servers.

```cql
library DiabetesA1cGap version '1.0.0'

using FHIR version '4.0.1'
include FHIRHelpers version '4.0.1'

valueset "Diabetes": 'urn:oid:2.16.840.1.113883.3.464.1003.103.12.1001'
valueset "HbA1c Lab": 'urn:oid:2.16.840.1.113883.3.464.1003.198.12.1013'

context Patient

define "Has Diabetes":
  exists ([Condition: "Diabetes"] C where C.clinicalStatus ~ "active")

define "Recent HbA1c":
  Last([Observation: "HbA1c Lab"] O
    where O.status in {'final', 'amended', 'corrected'}
    sort by effective)

define "Gap":
  "Has Diabetes" and
  ("Recent HbA1c" is null or
   "Recent HbA1c".effective before Today() - 6 months)
```

Verify the OIDs and value-set URLs against VSAC (the Value Set Authority Center) before using in production.

---

## Order Sets and Clinical Pathways

| Tool | Purpose | Best for |
|------|---------|----------|
| **Order set** | A grouped, pre-built collection of orders for a clinical scenario | Standardize admit orders, post-op orders, sepsis bundle |
| **Order panel** | A smaller bundle, usually a single phase of an order set | Pre-op labs, NPO orders |
| **Clinical pathway** | A multi-day plan with sequencing and conditional logic | Knee replacement pathway, oncology regimen, sepsis 1-hour bundle |
| **BPA (Epic) / Discern (Cerner)** | Vendor-native rule-driven alert | Quick wins entirely inside the EHR |

When an order set encodes the same clinical guidance as an interruptive alert, prefer the order set — it changes behavior without ever interrupting.

---

## Regulatory Status

CDS has two regulatory tracks in the US that are easy to confuse.

### HHS / ONC: Information Blocking and Predictive DSI

ONC's HTI-1 final rule (effective 2024) introduced **Predictive Decision Support Intervention (DSI)** transparency requirements for ONC-certified EHRs. Certified EHRs must surface "source attributes" for predictive DSIs (training data, validation, fairness, etc.). Confirm current requirements against the official ONC HTI-1 rule text.

### FDA: SaMD vs. exempt CDS (21st Century Cures Act)

The 21st Century Cures Act (2016) created a statutory carve-out: certain CDS functions are **not** a "device" and therefore not subject to FDA SaMD regulation. To qualify for the exemption, CDS must meet **all four** of these criteria (paraphrased — read the statute and FDA guidance for exact wording):

1. Not intended to acquire, process, or analyze a medical image or a signal from an in vitro diagnostic device or pattern/signal from a physiological monitor (i.e., it works on EHR data, not raw signals/images).
2. Intended to display, analyze, or print medical information about a patient or other medical information (clinical guidelines, peer-reviewed studies).
3. Intended for the purpose of supporting or providing recommendations to a health care professional about prevention, diagnosis, or treatment of a disease or condition.
4. Intended to enable the health care professional to **independently review the basis** for the recommendation (i.e., the clinician can see the inputs and reasoning and is not being asked to rely solely on the output).

If your CDS fails any prong (e.g., the clinician cannot independently review the reasoning, or the CDS is used in a time-critical setting where independent review is not feasible), it is a regulated device — go to the **fda-samd** skill.

> Verify the current FDA guidance "Clinical Decision Support Software" (most recent version) for the exact criteria. The 4-prong test wording has been refined since 2016.

---

## CDS Lifecycle

1. **Define the decision** — what is the clinician choosing between?
2. **Establish evidence** — guideline, peer-reviewed study, internal data.
3. **Pick the hook / surface** — when in the workflow does the decision happen?
4. **Prototype** — start non-interruptive. Iterate the wording with real clinicians.
5. **Pilot** — one unit, one service line. Measure fire rate, acceptance, time-to-action.
6. **Tune thresholds** — most alerts need their threshold raised after pilot.
7. **Govern** — every alert has an owner, a review cadence, a retirement trigger.
8. **Sunset** — alerts that fall below acceptance thresholds are retired, not "left running."

---

## Common Pitfalls

- Building the CDS in a vacuum, then bolting it into the workflow. Workflow design must come first.
- Treating every alert as critical. Reserve `critical` and modal alerts for true safety events.
- Ignoring the suppression model. Re-firing the same alert 7 times in one encounter is alert abuse.
- Hand-coding logic that should be in CQL — losing portability and auditability.
- Assuming "the EHR will do the right thing" with your card. Test in each customer's environment.
- Claiming HHS CDS exemption without actually satisfying all four Cures Act criteria.

---

## Task-Specific Questions

1. What is the decision, who makes it, in what setting?
2. What is the evidence base?
3. Which hook (or non-CDS-Hooks surface) lands at the right moment?
4. Will it be interruptive? What is the justification?
5. Who owns it, how often is it reviewed, and what is the retirement criterion?
6. Are you positioning this as HHS-exempt CDS or as a SaMD?

---

## Related Skills

- **healthcare-context**: EHR, setting, and regulatory posture shape every CDS choice.
- **smart-on-fhir**: SMART apps that pair with CDS Hooks for deeper interactions.
- **cpoe-orders**: Where `order-select` and `order-sign` hooks fire.
- **fhir-integration**: The data model CDS Hooks `prefetch` and CQL operate on.
- **terminology-services**: Value sets (VSAC), code systems used in CQL.
- **fda-samd**: When your CDS does not meet the Cures Act exemption criteria.
- **clinical-ai-ml**: When the CDS recommendation is generated by a model rather than rules.
