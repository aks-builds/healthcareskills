# SMART Scopes Reference

Side-by-side reference for SMART v1 and v2 scope syntax, common scope patterns, and how to choose the right scope set for an app.

## v1 vs v2 — When to Use Which

| Question | Answer |
|----------|--------|
| Does the EHR advertise `permission-v2` in its `.well-known/smart-configuration` capabilities? | If yes, use v2. If no, use v1. |
| Does the EHR also advertise v1 capabilities? | Most EHRs advertise both; v1 is still widely accepted. |
| Should you mix v1 and v2 scopes in one request? | No — pick one syntax per launch. |

v2 is strictly more expressive: it adds action letters (c/r/u/d/s) and optional search-parameter restrictions. If you can use it, you should.

## v1 Scopes

### Launch and context

| Scope | Means |
|-------|-------|
| `launch` | This is an EHR launch; expect a `launch` token in the authorize call |
| `launch/patient` | Standalone launch — prompt the user to pick a patient |
| `launch/encounter` | Standalone launch — prompt the user to pick an encounter |
| `openid` | Issue an `id_token` (OIDC) |
| `fhirUser` | Include a `fhirUser` claim in the `id_token` (a FHIR reference to the user, e.g., `Practitioner/example-prov-1`) |
| `online_access` | Refresh token valid only while the user has an active session |
| `offline_access` | Long-lived refresh token; survives user logout |

### Patient context (patient-facing apps)

`patient/<Resource>.<action>` — bound to the patient in the launch context.

| Scope | Means |
|-------|-------|
| `patient/Patient.read` | Read the patient record |
| `patient/Observation.read` | Read Observations for the patient |
| `patient/Condition.read` | Read Conditions for the patient |
| `patient/*.read` | Read every resource type the EHR exposes for the patient |
| `patient/DocumentReference.write` | Write a DocumentReference scoped to the patient |
| `patient/*.write` | Write any writable resource for the patient |

### User context (clinician-facing apps)

`user/<Resource>.<action>` — bound to what the authenticated user is allowed to see.

| Scope | Means |
|-------|-------|
| `user/Practitioner.read` | Read Practitioner resources the user can see |
| `user/Patient.read` | Read Patients the user has access to |
| `user/MedicationRequest.write` | Write MedicationRequest as the user |
| `user/*.read` | Read every resource the user can see |
| `user/*.write` | Write any writable resource the user can write |

### System context (Backend Services, no user)

`system/<Resource>.<action>` — bound to whatever the client is registered for.

| Scope | Means |
|-------|-------|
| `system/Patient.read` | System read for Patient |
| `system/*.read` | System read for all resources |
| `system/Group.read` | System read for Group (often used to identify export cohorts) |

## v2 Scopes (Granular)

v2 replaces `.read` / `.write` with action letters: `c` (create), `r` (read), `u` (update), `d` (delete), `s` (search), and optionally adds a search-parameter constraint.

### Action letters

| Letter | Means |
|--------|-------|
| `c` | Create (POST) |
| `r` | Read (GET by id) |
| `u` | Update (PUT) |
| `d` | Delete (DELETE) |
| `s` | Search (GET with query) |

Combine in alphabetical order: `cruds`, `rs`, `cr`, etc.

### Examples

| v1 scope | v2 equivalent |
|----------|---------------|
| `patient/Observation.read` | `patient/Observation.rs` (read + search) |
| `patient/Observation.write` | `patient/Observation.cu` (create + update) |
| `user/Patient.read` | `user/Patient.rs` |
| `system/*.read` | `system/*.rs` |
| (no v1 equivalent — read only, no search) | `user/Patient.r` |
| (no v1 equivalent — search only) | `user/Patient.s` |

### Search-parameter restriction

v2 allows you to restrict a scope to a specific search-parameter slice. The constraint appears as a query string after the action letters.

| Scope | Means |
|-------|-------|
| `patient/Observation.rs?category=laboratory` | Read + search Observations, but only labs |
| `patient/Observation.rs?category=vital-signs` | Read + search Observations, but only vitals |
| `user/MedicationRequest.cruds?status=active` | Full CRUD + search on MedicationRequest, scoped to active orders |

EHR support for parameterized v2 scopes varies — verify per vendor.

## Choosing Scopes — Practical Patterns

### Patient-facing portal app (read-only)

```
launch/patient openid fhirUser
patient/Patient.rs
patient/Condition.rs
patient/Observation.rs
patient/MedicationRequest.rs
patient/AllergyIntolerance.rs
patient/Procedure.rs
patient/Immunization.rs
patient/DocumentReference.rs
offline_access
```

(v2 equivalents shown — use `.read` if v1.)

### Clinician-facing chart-view app (read-only)

```
launch openid fhirUser
user/Patient.rs
user/Condition.rs
user/Observation.rs
user/MedicationRequest.rs
user/Encounter.rs
```

### Ambient scribe / note write-back app

```
launch openid fhirUser
user/Patient.r
user/Encounter.r
user/Condition.rs
user/Observation.rs
user/MedicationRequest.rs
user/DocumentReference.cu
```

Minimum required to read context and write the note. Avoid `user/*.write`.

### Quality-measure bulk export (Backend Services)

```
system/Patient.rs
system/Condition.rs
system/Observation.rs
system/MedicationRequest.rs
system/Procedure.rs
system/Encounter.rs
system/Group.rs
```

For `$export`, also include any resource types you need beyond those listed.

### CDS Hooks service (uses fhirAuthorization token)

The token passed in the hook's `fhirAuthorization` field is scoped narrowly by the EHR — typically `patient/*.read` for that patient and hook duration. The CDS service should not request additional scopes beyond what the hook delivers.

## Principles for Scope Choice

1. **Least privilege.** Request only the resources and actions you actually use. Approvers (and security reviewers) reject over-broad scope requests.
2. **Per-launch.** Launch context is per-session, per-launch. Do not assume a previous launch's scopes carry forward.
3. **Read before write.** Adding a write scope ratchets up the review effort at every customer. Don't request write unless you actually write.
4. **Granular over wildcard.** `patient/Observation.rs` is better than `patient/*.rs`. Wildcards make security review harder and trigger pushback.
5. **Search-parameter constraints (v2).** If you only need labs, ask only for labs.
6. **`offline_access` only if needed.** Long-lived refresh tokens are a higher-risk credential than session-scoped ones.
7. **Plan for scope downgrade.** The EHR may grant fewer scopes than requested. Code should handle missing scopes gracefully — feature-flag UI off, not crash.

## Common Scope Mistakes

- Requesting `patient/*.write` for an app that writes one resource type. Reviewers will reject.
- Requesting `offline_access` for a kiosk app that does not need long-lived sessions.
- Using `user/*` for a patient-facing app. Patient-facing apps should use `patient/`.
- Mixing v1 and v2 syntax in a single request.
- Forgetting that the EHR may not support every scope you request — always read the actual `scopes_supported` from `.well-known/smart-configuration`.
- Treating `patient/Patient.read` as the same as `launch/patient`. They are different: `launch/patient` is the context-selection scope; `patient/Patient.read` is the data-access scope.

> Verify supported scopes and any vendor-specific scope extensions against the EHR's `.well-known/smart-configuration` and the vendor's developer documentation. Coverage varies and changes.
