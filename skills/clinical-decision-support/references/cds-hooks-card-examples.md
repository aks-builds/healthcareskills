# CDS Hooks Card Examples

Full, valid CDS Hooks card response examples by indicator type. Indicators are `info`, `warning`, or `critical`. Reserve `critical` for true patient-safety events — overuse erodes the entire CDS surface.

All examples use synthetic patient and medication data. Verify clinical content against your institution's guidance before reusing.

## Discovery Endpoint

Every CDS Hooks service exposes a discovery endpoint listing the hooks it supports:

```http
GET /cds-services
```

```json
{
  "services": [
    {
      "hook": "patient-view",
      "title": "Preventive Care Gaps",
      "id": "preventive-care-gaps",
      "description": "Flags adult patients overdue for evidence-based preventive screenings.",
      "prefetch": {
        "patient": "Patient/{{context.patientId}}",
        "conditions": "Condition?patient={{context.patientId}}&clinical-status=active",
        "observations": "Observation?patient={{context.patientId}}&category=laboratory&_sort=-date&_count=50"
      }
    }
  ]
}
```

## Info Card (Non-Interruptive)

Use for FYI / passive recommendations. Should not interrupt the clinician's workflow.

```json
{
  "cards": [
    {
      "uuid": "card-info-1",
      "summary": "Colorectal cancer screening due",
      "indicator": "info",
      "detail": "Adult patient is age-eligible for colorectal cancer screening (average risk). No completed screening on file in the last 10 years for colonoscopy or 1 year for FIT.",
      "source": {
        "label": "USPSTF Colorectal Cancer Screening (verify current recommendation)",
        "url": "https://www.uspreventiveservicestaskforce.org/"
      },
      "links": [
        {
          "label": "Open patient screening dashboard",
          "url": "https://app.example.com/screenings?patient={{patientId}}",
          "type": "smart"
        }
      ]
    }
  ]
}
```

## Warning Card with Suggestion

Use when the recommendation is meaningful but does not require the clinician to stop. Includes an actionable suggestion the clinician can accept with a single click.

```json
{
  "cards": [
    {
      "uuid": "card-warning-1",
      "summary": "HbA1c overdue for diabetic patient",
      "indicator": "warning",
      "detail": "Patient has type 2 diabetes (active on problem list). Last HbA1c was over 6 months ago. Current guideline recommends HbA1c every 3-6 months for patients above target.",
      "source": {
        "label": "ADA Standards of Care (verify current edition)",
        "url": "https://professional.diabetes.org/standards-of-care"
      },
      "suggestions": [
        {
          "uuid": "suggestion-order-a1c",
          "label": "Order HbA1c",
          "actions": [
            {
              "type": "create",
              "description": "Create ServiceRequest for HbA1c lab",
              "resource": {
                "resourceType": "ServiceRequest",
                "status": "draft",
                "intent": "order",
                "category": [
                  {
                    "coding": [
                      {
                        "system": "http://snomed.info/sct",
                        "code": "108252007",
                        "display": "Laboratory procedure (verify against current SNOMED CT release)"
                      }
                    ]
                  }
                ],
                "code": {
                  "coding": [
                    {
                      "system": "http://loinc.org",
                      "code": "4548-4",
                      "display": "Hemoglobin A1c/Hemoglobin.total in Blood (verify against current LOINC release)"
                    }
                  ]
                },
                "subject": { "reference": "Patient/example-1" }
              }
            }
          ]
        }
      ],
      "links": [
        {
          "label": "Open diabetes flowsheet",
          "url": "https://app.example.com/diabetes-flowsheet?patient={{patientId}}",
          "type": "smart"
        }
      ]
    }
  ]
}
```

## Critical Card (Modal / Hard-Stop)

Use only for imminent patient harm. The clinician should not be able to proceed without acknowledgement. Overuse trains override behavior.

```json
{
  "cards": [
    {
      "uuid": "card-critical-1",
      "summary": "Documented allergy to ordered medication",
      "indicator": "critical",
      "detail": "Patient has a documented allergy to the medication class of the ordered drug. Reaction history: anaphylaxis (severe, life-threatening). Reviewing prescriber must acknowledge and select an alternative.",
      "source": {
        "label": "Institution-curated allergy cross-sensitivity reference (verify current source)",
        "url": "https://docs.example.org/allergy-cross-sensitivity"
      },
      "suggestions": [
        {
          "uuid": "suggestion-cancel-order",
          "label": "Cancel order and select alternative",
          "actions": [
            {
              "type": "delete",
              "description": "Cancel the offending MedicationRequest",
              "resourceId": "MedicationRequest/example-order-1"
            }
          ]
        }
      ],
      "overrideReasons": [
        {
          "code": "verified-tolerance",
          "system": "http://example.org/cds-override-reasons",
          "display": "Allergy resolved — verified by patient interview"
        },
        {
          "code": "different-mechanism",
          "system": "http://example.org/cds-override-reasons",
          "display": "Cross-reactivity not clinically relevant for this agent"
        },
        {
          "code": "benefit-outweighs-risk",
          "system": "http://example.org/cds-override-reasons",
          "display": "No suitable alternative; pre-medication plan in place"
        }
      ]
    }
  ]
}
```

## Multiple Cards in One Response

A hook can return multiple cards. Order them from most to least urgent — most EHRs display them in card order.

```json
{
  "cards": [
    { "uuid": "c1", "indicator": "critical", "summary": "...", "detail": "...", "source": { "label": "..." } },
    { "uuid": "c2", "indicator": "warning",  "summary": "...", "detail": "...", "source": { "label": "..." } },
    { "uuid": "c3", "indicator": "info",     "summary": "...", "detail": "...", "source": { "label": "..." } }
  ]
}
```

## Empty Response

If the hook fires and the service has nothing to say, return an empty card array. Do not return cards that say "no findings" — that is noise.

```json
{ "cards": [] }
```

## Card Field Reference (Selected)

| Field | Required | Notes |
|-------|----------|-------|
| `uuid` | Recommended | Stable identifier per card; helps EHRs deduplicate and the service log acceptance |
| `summary` | Yes | One-line headline shown in compact UI; aim for under 80 chars |
| `indicator` | Yes | `info` / `warning` / `critical` |
| `detail` | Recommended | Markdown-formatted explanation |
| `source` | Yes | `label` and `url` (and optionally `icon`) — what evidence backs the card |
| `suggestions` | Optional | Pre-built FHIR resource actions the clinician can accept |
| `selectionBehavior` | Optional | `at-most-one` or `any` for how the EHR presents multiple suggestions |
| `links` | Optional | External or SMART app deep links |
| `overrideReasons` | Optional | Structured reasons for override (especially for critical cards) |

> Field semantics and required fields evolve across CDS Hooks spec versions. Confirm against the current CDS Hooks specification before implementing.

## Hook Request Payload (Reference)

```json
{
  "hook": "patient-view",
  "hookInstance": "d1577c69-dfbe-44ad-ba6d-3e05e953b2ea",
  "fhirServer": "https://ehr.example.org/fhir/R4",
  "fhirAuthorization": {
    "access_token": "<short-lived bearer>",
    "token_type": "Bearer",
    "expires_in": 300,
    "scope": "patient/Patient.r patient/Condition.rs patient/Observation.rs",
    "subject": "example-app-client-id"
  },
  "context": {
    "userId": "Practitioner/example-prov-1",
    "patientId": "example-1",
    "encounterId": "example-enc-1"
  },
  "prefetch": {
    "patient": { "resourceType": "Patient", "id": "example-1" },
    "conditions": { "resourceType": "Bundle", "entry": [] },
    "observations": { "resourceType": "Bundle", "entry": [] }
  }
}
```

## Implementation Notes

- Treat the `fhirAuthorization` token as ephemeral — use it for the duration of the hook handler only.
- Validate the hook context (patient, encounter, user) against expectations before producing cards.
- Time-box your hook handler aggressively. Most EHRs will time out a CDS service in 1-5 seconds.
- Log every card produced + the eventual acceptance/override outcome. This is the measurement substrate for retiring low-acceptance alerts.
