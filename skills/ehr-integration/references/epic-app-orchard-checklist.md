# Epic Connection Hub Submission Checklist

Connection Hub is the current name of Epic's app program (formerly App Orchard, then briefly Showroom). This checklist captures the major artifacts and decisions third-party app developers should prepare. Verify the current submission requirements against Epic's developer portal — they are updated regularly.

## Naming Evolution

- App Orchard (original)
- Showroom (intermediate rename)
- **Connection Hub** (current)

Older docs and blog posts still use the old names. If a customer's contract references App Orchard, treat it as equivalent unless legal calls out otherwise.

## Membership Tiers

Connection Hub has multiple tiers. Higher tiers gate access to more APIs, more sandbox capabilities, and broader customer-distribution rights. Confirm current tier names, fees, and rights with Epic — they change.

The ONC-mandated USCDI patient-facing FHIR read endpoints are accessible **without** Connection Hub membership. Most provider-facing scopes and almost all write scopes require Connection Hub access.

## Pre-Submission Checklist

### Product

- [ ] Define the workflow precisely (who is the user, what do they do, where in Hyperdrive / Hyperspace / web does it live).
- [ ] Decide on launch model: EHR launch (Hyperdrive embed), standalone (web/mobile), or both.
- [ ] Inventory FHIR resources read and written; specify scopes (v1 or v2).
- [ ] Decide on client type: public (PKCE only) vs confidential (client_secret) vs confidential asymmetric (JWT assertion).
- [ ] Confirm whether Backend Services / bulk export is required.

### Security and Privacy

- [ ] Privacy policy URL ready and accessible.
- [ ] Terms of service URL ready.
- [ ] Security questionnaire response (SOC 2 Type 2 or equivalent typically expected).
- [ ] Penetration test results (recent — typically within last 12 months).
- [ ] Data flow diagram showing PHI handling end-to-end.
- [ ] Encryption at rest and in transit documented.
- [ ] Key management documented (HSM, KMS, rotation).
- [ ] Logging and audit retention documented (HIPAA accounting of disclosures).
- [ ] Incident response plan and breach-notification commitment.

### HIPAA

- [ ] Identify the app's role: Business Associate to each customer covered entity.
- [ ] BAA template ready for customer sign-off.
- [ ] Workforce HIPAA training documented.
- [ ] PHI minimum-necessary scoping documented (scope choices match this).
- [ ] De-identification strategy documented if any data leaves the customer's tenant for product analytics.

### Technical Validation

- [ ] App tested end-to-end against the Open Epic / fhir.epic.com sandbox.
- [ ] SMART App Launch flow validated (EHR launch + standalone if applicable).
- [ ] PKCE implemented; `state` validated; `aud` parameter included on every authorize call.
- [ ] All read scopes exercised against synthetic Epic patients.
- [ ] All write scopes exercised; behavior when scope denied is graceful.
- [ ] Token refresh path tested; refresh-token rotation handled.
- [ ] Error handling for 401, 403, 429, 5xx.
- [ ] Backoff + retry strategy documented.
- [ ] Bulk export path tested (if applicable) — async polling and NDJSON streaming.

### App-Specific Artifacts

- [ ] App description (clinical workflow, target users, value proposition).
- [ ] Screenshots of the user-facing flows.
- [ ] Support model documented (hours, response times, escalation).
- [ ] Customer onboarding playbook (per-customer client ID, redirect URI registration, security review).
- [ ] Pricing model (Epic publishes this in some cases).
- [ ] Reference customer commitments (often required to graduate from listed-only to fully distributed).

## After Submission

- [ ] Epic vendor services review (timing varies — months are typical).
- [ ] Each customer site still has its own go-live and security review.
- [ ] Each customer issues its own client ID — no client ID is reusable across customers.
- [ ] HIPAA BAA signed with each customer before any production PHI flows.
- [ ] AuditEvent capture in place for every API call (per HIPAA retention).

## Common Reasons Submissions Stall

- Privacy policy or BAA template is missing or weak.
- Penetration test is stale or limited in scope.
- Workflow description is vague — Epic wants to understand exactly where in Hyperdrive the app lives.
- Scope request is over-broad (`patient/*.write` when the app only needs `patient/DocumentReference.write`).
- No clear support model.
- No reference customer.

> All program details, artifact lists, and review timelines must be verified against Epic's current Connection Hub developer documentation. The above is a representative checklist, not an exhaustive or contractual list.
