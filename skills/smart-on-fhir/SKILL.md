---
name: smart-on-fhir
description: When the user wants to build a SMART on FHIR app, configure OAuth against an EHR's FHIR API, or implement Backend Services. Also use when the user mentions "SMART," "SMART on FHIR," "SMART App Launch," "EHR launch," "standalone launch," "well-known/smart-configuration," "smart-configuration," "launch context," "launch/patient," "launch/encounter," "SMART scopes," "patient/*.read," "user/*.write," "system/*.read," "SMART v2 scopes," "fhirUser," "PKCE," "JWT assertion," "Backend Services," "client credentials FHIR," or "Bulk Data Access." For deeper FHIR resource work, see fhir-integration. For vendor-specific OAuth quirks, see ehr-integration. For CDS Hooks integration, see clinical-decision-support.
metadata:
  version: 1.0.0
---

# SMART on FHIR

You are an expert in SMART on FHIR — the OAuth 2.0 profile that lets third-party apps and services securely access FHIR APIs in EHRs. Your goal is to help engineers implement a correct, secure SMART flow end-to-end, with the right scopes, the right client type, and the right launch model for the use case.

## Initial Assessment

Read `.agents/healthcare-context.md` first (fall back to `.claude/healthcare-context.md`). The context file tells you which EHR(s) you are targeting (each one publishes slightly different SMART support), which FHIR version, and whether the app is patient-facing, clinician-facing, or system-to-system.

If the context file does not exist, ask:

1. Which EHR(s) and which FHIR version?
2. App type — clinician-facing web app, patient-facing app, mobile, server-to-server?
3. Launch model — EHR launch (from within the chart) or standalone (user starts in the app)?
4. Public or confidential client?
5. Which FHIR resources do you need to read or write?

---

## Launch Models

| Model | Who starts | When to use |
|-------|-----------|-------------|
| **EHR launch** | Clinician clicks the app from within the EHR chart | Clinician-facing apps, ambient scribes, contextual tools |
| **Standalone launch** | User opens the app first, then signs in | Patient-facing apps, cross-institution tools |
| **Backend Services** | No user — system-to-system | Bulk data pipelines, analytics, automated jobs |

You can support multiple models in the same app — register separately for each.

---

## SMART App Launch Sequence (EHR Launch)

```
1. EHR opens the app URL with a `launch` token + `iss` (FHIR base URL).
2. App fetches `<iss>/.well-known/smart-configuration` to discover endpoints.
3. App redirects user agent to `authorization_endpoint` with:
     response_type=code, client_id, redirect_uri, scope, state,
     aud=<iss>, launch=<launch_token>, code_challenge, code_challenge_method=S256
4. EHR authenticates the user, optionally prompts for consent.
5. EHR redirects back to app's redirect_uri with `code` and `state`.
6. App POSTs to `token_endpoint` with code, redirect_uri, client_id, code_verifier
   (and client_secret or JWT assertion for confidential clients).
7. EHR returns access_token + (optionally) refresh_token + id_token + context:
     { access_token, token_type: "Bearer", expires_in, scope, patient, encounter, ... }
8. App calls FHIR APIs with `Authorization: Bearer <access_token>`.
```

`state` is mandatory for CSRF protection. PKCE (`S256`) is required for public clients and strongly recommended for confidential clients.

### Standalone launch difference

In standalone launch, the user starts in the app. The app initiates step 3 directly with no `launch` parameter; it requests `launch/patient` scope so the EHR prompts the user to select a patient (typically the user is the patient — a patient portal flow).

---

## .well-known/smart-configuration

Every SMART-enabled FHIR server should publish:

```http
GET https://ehr.example.org/fhir/R4/.well-known/smart-configuration
```

Response (selected fields):

```json
{
  "authorization_endpoint": "https://ehr.example.org/oauth2/authorize",
  "token_endpoint": "https://ehr.example.org/oauth2/token",
  "introspection_endpoint": "https://ehr.example.org/oauth2/introspect",
  "revocation_endpoint": "https://ehr.example.org/oauth2/revoke",
  "scopes_supported": [
    "openid", "fhirUser", "launch", "launch/patient", "launch/encounter",
    "patient/*.read", "user/*.read", "offline_access", "online_access",
    "system/*.read"
  ],
  "response_types_supported": ["code"],
  "capabilities": [
    "launch-ehr", "launch-standalone",
    "client-public", "client-confidential-symmetric", "client-confidential-asymmetric",
    "context-ehr-patient", "context-ehr-encounter",
    "permission-patient", "permission-user", "permission-online", "permission-offline",
    "sso-openid-connect",
    "permission-v2"
  ],
  "code_challenge_methods_supported": ["S256"]
}
```

Confirm the EHR's actual response — capabilities vary.

---

## SMART Scopes

### v1 (legacy, still widely supported)

| Scope | Means |
|-------|-------|
| `launch` | The app is being launched from an EHR; expect a `launch` token |
| `launch/patient` | Standalone launch; prompt user to pick a patient |
| `launch/encounter` | Standalone launch; prompt user to pick an encounter |
| `patient/Observation.read` | Read Observations for the patient in context |
| `patient/*.read` | Read every resource type for the patient in context |
| `user/Practitioner.read` | Read Practitioner resources the user can see |
| `user/*.write` | Write any resource the user can write |
| `system/*.read` | System-to-system read (Backend Services) |
| `openid` `fhirUser` | OIDC: get an id_token; `fhirUser` claim is a FHIR reference to the user |
| `online_access` | Allow refresh token only while user has an active session |
| `offline_access` | Allow long-lived refresh token even when user is offline |

### v2 (granular)

SMART v2 adds **resource-level granularity** and **search parameter restriction**:

| Scope | Means |
|-------|-------|
| `patient/Observation.rs` | Read + search for Observations in patient context |
| `patient/Observation.rs?category=laboratory` | Read + search only labs |
| `user/MedicationRequest.cruds` | Full CRUD + search on MedicationRequest |
| `user/Patient.r` | Read only (not search) |

v2 actions: `c` (create), `r` (read), `u` (update), `d` (delete), `s` (search). Constrain to the minimum needed.

### Choosing scopes

- Ask for the **least privilege** that satisfies the use case.
- Prefer `patient/` scopes for patient-facing apps; `user/` for clinician apps; `system/` for backend.
- Request `offline_access` only if you actually need long-lived background sync.
- If the EHR supports v2 (`permission-v2` capability), use v2 granular scopes.

---

## Public vs. Confidential Clients

| Client type | Where it runs | Can keep a secret? | Auth method |
|-------------|---------------|---------------------|-------------|
| Public | Browser SPA, mobile app | No | PKCE only |
| Confidential symmetric | Server-side web app | Yes | client_secret + PKCE |
| Confidential asymmetric | Server-side, high-assurance | Yes (private key) | JWT client assertion (RFC 7523) |

For SMART Backend Services, confidential asymmetric (JWT assertion with a JWKS-published public key) is the only standard option.

---

## PKCE

PKCE is mandatory for public clients and best-practice for confidential clients.

```
code_verifier      = random 43-128 char string (base64url, unreserved)
code_challenge     = base64url(SHA256(code_verifier))
code_challenge_method = "S256"
```

Send `code_challenge` + `code_challenge_method` on the authorize call; send `code_verifier` on the token call. The authorization server verifies.

---

## Token Introspection

Per RFC 7662. The resource server (your app, or another component) asks the auth server whether a bearer token is currently valid:

```http
POST /oauth2/introspect
Authorization: Basic <base64(client_id:client_secret)>
Content-Type: application/x-www-form-urlencoded

token=<access_token>
```

Response:

```json
{
  "active": true,
  "scope": "patient/*.read launch/patient",
  "client_id": "example-app",
  "sub": "Practitioner/example-prov-1",
  "exp": 1737200000,
  "patient": "example-1"
}
```

Not every EHR exposes introspection. For most apps, just validate the token signature and `exp` claim locally if it's a JWT, or simply rely on the FHIR API to reject expired tokens.

---

## FHIR Context

After token exchange, the response includes launch context for use in API calls:

| Field | Means |
|-------|-------|
| `patient` | Patient ID for context — use as `?patient=<id>` in FHIR calls |
| `encounter` | Encounter ID for context |
| `fhirUser` (id_token claim, if `openid fhirUser`) | FHIR reference to the user (e.g., `Practitioner/example-prov-1`) |
| `tenant` (vendor-specific) | Tenant/customer identifier |
| `need_patient_banner` (Epic / some vendors) | UI hint — does the EHR already display a patient banner? |

Persist this context across the session, not across sessions — a new launch may select a different patient.

---

## SMART Backend Services

For system-to-system, no-user scenarios (bulk export, automated quality reporting, registry submission).

1. Generate an asymmetric key pair (RS384 or ES384 per SMART Backend Services).
2. Register your public key with the EHR (either by uploading the JWK or pointing to a JWKS URL you host).
3. At token time, build a JWT:

```json
{
  "iss": "<your-client-id>",
  "sub": "<your-client-id>",
  "aud": "https://ehr.example.org/oauth2/token",
  "exp": <now + 5 minutes>,
  "jti": "<random unique>"
}
```

Sign with your private key (RS384 or ES384).

4. POST to the token endpoint:

```http
POST /oauth2/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
&scope=system/*.read
&client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer
&client_assertion=<signed_jwt>
```

5. Receive a short-lived access token. No refresh token — re-mint a JWT and request a new token when needed.

Use Backend Services for **Bulk Data Access** (`$export`) — kick off the export, poll the status endpoint, download NDJSON.

---

## End-to-End Authorization Code Walkthrough (EHR Launch)

### Step 1: EHR launches your app

```
https://app.example.com/launch?iss=https://ehr.example.org/fhir/R4&launch=abc123
```

### Step 2: Discover endpoints

```http
GET https://ehr.example.org/fhir/R4/.well-known/smart-configuration
```

### Step 3: Build PKCE + redirect to authorize

```
GET https://ehr.example.org/oauth2/authorize
  ?response_type=code
  &client_id=example-app
  &redirect_uri=https://app.example.com/cb
  &scope=launch openid fhirUser patient/*.read offline_access
  &state=opaque-csrf-token
  &aud=https://ehr.example.org/fhir/R4
  &launch=abc123
  &code_challenge=BASE64URL(SHA256(verifier))
  &code_challenge_method=S256
```

### Step 4: User authenticates in the EHR; EHR redirects back

```
https://app.example.com/cb?code=xyz789&state=opaque-csrf-token
```

Verify `state` matches what you sent. Reject if not.

### Step 5: Exchange code for tokens

```http
POST https://ehr.example.org/oauth2/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&code=xyz789
&redirect_uri=https://app.example.com/cb
&client_id=example-app
&code_verifier=<original_verifier>
```

(For confidential clients, add `client_secret` or use `client_assertion` instead.)

### Step 6: Use the token

```http
GET https://ehr.example.org/fhir/R4/Patient/example-1
Authorization: Bearer <access_token>
Accept: application/fhir+json
```

### Step 7: Refresh when needed

```http
POST https://ehr.example.org/oauth2/token
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token
&refresh_token=<refresh_token>
&scope=launch openid fhirUser patient/*.read offline_access
```

Note that some EHRs rotate refresh tokens on each use — store the latest one returned.

---

## CDS Hooks Integration

SMART apps and CDS Hooks complement each other:

- A CDS Hooks card with a `link` of `type: smart` opens a SMART app — the EHR launches it as an EHR launch with context already established.
- Inside that SMART app, you can do richer interaction than fits in a card.
- The CDS Hooks `fhirAuthorization` field gives the service a short-lived token to fetch additional data — same OAuth, smaller scope.

---

## Security Checklist

- [ ] Verify `state` on every redirect
- [ ] PKCE for every public client, recommended for confidential
- [ ] Store refresh tokens encrypted, scoped to a single user+tenant
- [ ] Validate `iss` and `aud` on every token / id_token
- [ ] Validate id_token signature against the EHR's published JWKS
- [ ] Never log tokens or `code` parameters
- [ ] TLS 1.2+ on every endpoint; HSTS on app domains
- [ ] Rotate signing keys for Backend Services; publish via JWKS
- [ ] Audit every FHIR API call (FHIR `AuditEvent` or your own audit log)
- [ ] Handle 401 (expired), 403 (scope), 429 (rate limit) cleanly

---

## Common Pitfalls

- Forgetting `aud` in the authorize URL — required by SMART; EHRs will reject.
- Using `client_secret` for a public SPA. SPAs cannot keep a secret; use PKCE alone.
- Requesting `*.write` scopes you don't actually use. Users (and approvers) will reject the app.
- Treating the launch context as global across users. It is per-launch, per-session.
- Hardcoding the FHIR base URL. Each customer's `iss` is different.
- Failing to handle scope downgrade — the EHR may grant fewer scopes than requested.
- Building Backend Services with a long-lived secret. The spec is JWT assertion only.

---

## Task-Specific Questions

1. Which EHR and which `iss` URL (sandbox + each prod tenant)?
2. EHR launch, standalone, Backend Services, or all three?
3. Public, confidential symmetric, or confidential asymmetric client?
4. Which scopes — minimum sufficient?
5. Where do you store the refresh token, and what is the rotation policy?
6. How do you handle a token revoked or scope-reduced mid-session?

---

## Related Skills

- **healthcare-context**: EHR + app type drive every launch decision.
- **fhir-integration**: What you actually fetch and write once authenticated.
- **ehr-integration**: Vendor-specific OAuth quirks (Epic, Cerner, Athena, etc.).
- **clinical-decision-support**: CDS Hooks pairs with SMART apps via `link.type=smart`.
- **hipaa-compliance**: Audit, access control, breach reporting for SMART app data flows.
- **audit-logging**: AuditEvent capture for FHIR access through your SMART app.
- **patient-portal**: Patient-facing standalone launch flows.
