# SMART on FHIR OAuth Flow Walkthrough

Step-by-step OAuth flows for the three SMART launch models: EHR launch (authorization code), standalone launch (authorization code), and Backend Services (client_credentials with JWT assertion). All flows use TLS, `state` for CSRF, and PKCE where applicable.

## EHR Launch — Sequence Diagram

```
+-----------+         +----------+         +--------------+         +----------+
|  EHR /    |         |  Your    |         |  EHR Auth    |         |  EHR     |
|  Hyperdrive|        |   App    |         |   Server     |         |  FHIR    |
+-----+-----+         +-----+----+         +------+-------+         +-----+----+
      |                     |                     |                       |
      | 1. open(launch_url) |                     |                       |
      | with iss, launch    |                     |                       |
      |-------------------->|                     |                       |
      |                     |                     |                       |
      |                     | 2. GET /.well-known/smart-configuration     |
      |                     |---------------------------->                |
      |                     |                     |                       |
      |                     |     {authorize, token, scopes_supported}    |
      |                     |<----------------------------                |
      |                     |                     |                       |
      |                     | 3. 302 to /authorize                        |
      |                     |   response_type=code, client_id,            |
      |                     |   redirect_uri, scope, state, aud,          |
      |                     |   launch, code_challenge, S256              |
      |                     |-------------------->|                       |
      |                     |                     |                       |
      |    4. User authenticates and consents     |                       |
      |<------------------- (EHR auth UI) ------->|                       |
      |                     |                     |                       |
      |                     | 5. 302 to redirect_uri                      |
      |                     |   ?code=...&state=...                       |
      |                     |<--------------------|                       |
      |                     |                     |                       |
      |                     | 6. POST /token      |                       |
      |                     |   grant_type=authorization_code             |
      |                     |   code, redirect_uri, code_verifier,        |
      |                     |   client_id (+ secret OR client_assertion)  |
      |                     |-------------------->|                       |
      |                     |                     |                       |
      |                     |     access_token, refresh_token,            |
      |                     |     expires_in, id_token (if openid),       |
      |                     |     patient, encounter, scope               |
      |                     |<--------------------|                       |
      |                     |                     |                       |
      |                     | 7. GET /Patient/{id}                        |
      |                     |   Authorization: Bearer <access_token>      |
      |                     |---------------------------------+---------->|
      |                     |                     |                       |
      |                     |                     |     FHIR resource     |
      |                     |<--------------------------------+-----------|
      |                     |                     |                       |
```

### Step-by-step

#### 1. EHR launches the app

The EHR opens the app's launch URL with two query parameters: `iss` (the FHIR base URL) and `launch` (an opaque launch token).

```
GET https://app.example.com/launch?iss=https://ehr.example.org/fhir/R4&launch=abc123
```

The app's launch endpoint extracts `iss` and `launch`, then proceeds to discovery.

#### 2. Discover the authorization endpoints

Fetch the SMART configuration document from the FHIR base URL:

```http
GET https://ehr.example.org/fhir/R4/.well-known/smart-configuration
```

Parse `authorization_endpoint`, `token_endpoint`, `scopes_supported`, `capabilities`, `code_challenge_methods_supported`.

#### 3. Build PKCE and redirect to authorize

Generate a PKCE pair:

- `code_verifier` — random 43-128 char base64url-safe string.
- `code_challenge` — base64url-encoded SHA-256 of the verifier.

Generate an opaque `state` value for CSRF defense. Store the verifier and state server-side (or in encrypted session storage) keyed by state.

Redirect the user agent:

```
GET https://ehr.example.org/oauth2/authorize
  ?response_type=code
  &client_id=example-app
  &redirect_uri=https://app.example.com/cb
  &scope=launch openid fhirUser patient/*.read offline_access
  &state=<opaque>
  &aud=https://ehr.example.org/fhir/R4
  &launch=abc123
  &code_challenge=<challenge>
  &code_challenge_method=S256
```

Mandatory: `response_type=code`, `client_id`, `redirect_uri`, `scope`, `state`, `aud`, `code_challenge`, `code_challenge_method`. `launch` is required for EHR launch (carries the launch token from step 1).

#### 4. User authenticates and consents

The EHR authenticates the user (often single sign-on if launching from the chart) and optionally prompts for consent to the requested scopes. The EHR may also enforce its own scope policy and grant fewer scopes than requested — your app must handle scope downgrade gracefully.

#### 5. EHR redirects back to your redirect URI

```
https://app.example.com/cb?code=xyz789&state=<opaque>
```

**Validate `state` against the stored value.** Reject the request if they don't match. This is the CSRF defense and is mandatory.

#### 6. Exchange code for tokens

POST to the token endpoint:

```http
POST https://ehr.example.org/oauth2/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&code=xyz789
&redirect_uri=https://app.example.com/cb
&client_id=example-app
&code_verifier=<original_verifier>
```

For confidential symmetric clients, add `client_secret`. For confidential asymmetric clients, replace `client_secret` with `client_assertion_type=urn:ietf:params:oauth:client-assertion-type:jwt-bearer` and `client_assertion=<signed_jwt>`.

Response:

```json
{
  "access_token": "<bearer>",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "launch openid fhirUser patient/Patient.r patient/Condition.rs",
  "refresh_token": "<opaque>",
  "id_token": "<jwt, if openid requested>",
  "patient": "example-1",
  "encounter": "example-enc-1"
}
```

If `openid fhirUser` was granted, validate the `id_token` signature against the EHR's published JWKS and extract the `fhirUser` claim (e.g., `Practitioner/example-prov-1`).

#### 7. Call the FHIR API

```http
GET https://ehr.example.org/fhir/R4/Patient/example-1
Authorization: Bearer <access_token>
Accept: application/fhir+json
```

Use the launch context (`patient`, `encounter`) returned in the token response as filter parameters in subsequent FHIR queries.

#### Refresh

When the access token expires (or you receive a 401 from the FHIR API):

```http
POST https://ehr.example.org/oauth2/token
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token
&refresh_token=<refresh_token>
&scope=launch openid fhirUser patient/*.read offline_access
```

Some EHRs **rotate** the refresh token on each use. Always store the latest refresh token returned. If you store the old one after a rotation, the next refresh will fail.

---

## Standalone Launch

In standalone launch the user starts in the app (not the EHR chart). The app then drives the OAuth flow.

### Differences from EHR launch

- No `launch` parameter from the EHR.
- The app requests `launch/patient` (or `launch/encounter`) scope so the EHR's auth server prompts the user to select a patient.
- Otherwise, steps 2-7 are identical.

### Patient-portal pattern

For a patient-facing app, standalone launch typically authenticates the user as the patient. The `patient` in the token response is the patient ID matching the authenticated user. The app uses `patient/*.read` scopes scoped to that patient.

### Cross-institution pattern

A patient may use a single standalone app to read data from multiple EHRs. Each EHR has its own `iss`, its own client registration, its own auth server, and its own access tokens. Persist per-tenant.

---

## Backend Services (No User)

For system-to-system (bulk export, automated quality reporting, registry submission, scheduled data sync).

### Sequence

```
+----------+        +------------------+        +-----------+
|  Your    |        |  EHR Auth Server |        |  EHR FHIR |
|  Service |        |  (token endpoint)|        |           |
+-----+----+        +---------+--------+        +-----+-----+
      |                       |                       |
      | 1. Build & sign JWT   |                       |
      |    iss=client_id      |                       |
      |    sub=client_id      |                       |
      |    aud=token_endpoint |                       |
      |    exp=now+5min       |                       |
      |    jti=<unique>       |                       |
      |                       |                       |
      | 2. POST /token        |                       |
      |    grant_type=client_credentials              |
      |    scope=system/*.read                        |
      |    client_assertion_type=jwt-bearer           |
      |    client_assertion=<signed_jwt>              |
      |---------------------->|                       |
      |                       |                       |
      | 3. {access_token, expires_in, scope}          |
      |<----------------------|                       |
      |                       |                       |
      | 4. POST /$export (Bulk Data)                  |
      |    Prefer: respond-async                      |
      |    Authorization: Bearer <token>              |
      |------------------------------+--------------->|
      |                       |                       |
      | 5. 202 + Content-Location: <status URL>       |
      |<-----------------------------+----------------|
      |                       |                       |
      | 6. GET <status URL>   |                       |
      |    repeat until done  |                       |
      |------------------------------+--------------->|
      |                       |                       |
      | 7. 200 + {output: [{url: ..., type: ...}, ...]}
      |<-----------------------------+----------------|
      |                       |                       |
      | 8. GET each NDJSON output URL                 |
      |    stream and process                         |
      |------------------------------+--------------->|
      |                       |                       |
```

### Key checks

- Register your public key with the EHR via direct JWK upload or by publishing a JWKS URL.
- Sign the assertion JWT with `RS384` or `ES384` per the SMART Backend Services profile.
- Use a unique `jti` per request to prevent replay.
- Short `exp` (typically 5 minutes).
- No refresh token is issued — re-mint a fresh assertion JWT for each token request.
- For Bulk Data, plan for files in the 1-100 GB range; stream NDJSON rather than loading into memory.

---

## Security Checklist (All Flows)

- [ ] Verify `state` on every redirect.
- [ ] PKCE (S256) on every public client; recommended for confidential.
- [ ] Validate `iss` and `aud` on every token and `id_token`.
- [ ] Validate `id_token` signature against EHR JWKS (cache the JWKS, refresh on `kid` miss).
- [ ] Store refresh tokens encrypted, scoped to a single user + tenant.
- [ ] Never log tokens, codes, or assertion JWTs.
- [ ] TLS 1.2+ on every endpoint; HSTS on app domains.
- [ ] For Backend Services, rotate signing keys; publish via JWKS.
- [ ] Handle 401 (expired), 403 (scope), 429 (rate limit), 5xx cleanly with retry/backoff.
- [ ] AuditEvent (FHIR) or equivalent audit log of every FHIR API call.

## Common Errors and What They Mean

| Error | Cause |
|-------|-------|
| `invalid_request` with "missing aud" | `aud` parameter omitted from authorize call. Required by SMART. |
| `invalid_grant` on token exchange | Code expired, redirect_uri mismatch, code_verifier mismatch, or code already used. |
| `invalid_scope` | Requested a scope the EHR does not support or has not enabled for this client. |
| `unauthorized_client` | Wrong client_id for this tenant, or client_assertion signing key not registered. |
| 401 on FHIR call after fresh token | Token did not include the scope for that resource; or tenant routing is wrong. |
| 403 on FHIR call | Scope insufficient for the requested operation. |
| 429 | Rate limit. Back off with exponential delay + jitter. |
