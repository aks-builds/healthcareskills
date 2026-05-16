# PHI Encryption Cheatsheet

A working reference for encryption choices that protect PHI at rest, in transit, and in use, plus key-management patterns. Verify specific algorithm/keysize/protocol expectations against current NIST guidance (SP 800-131A, SP 800-175B, etc.), FIPS 140-3 module status, and the current text of the HHS guidance on encryption referenced by the HIPAA Breach Notification Rule. Encryption is one control among many — risk analysis, access control, audit logging, and contingency planning remain required. Consult your security architect for binding decisions.

## At Rest

### Algorithms
- **AES-256-GCM** for authenticated encryption (preferred)
- **AES-256-CBC + HMAC-SHA-256** (encrypt-then-MAC) where GCM is not available
- Avoid AES-CBC without an HMAC; avoid AES-ECB; avoid 3DES; avoid RC4

### Key sizes and modes (verify current NIST guidance)
- Symmetric: AES-128 minimum, AES-256 preferred for long-lived PHI
- Asymmetric: RSA-3072+ or ECDSA P-256+ for new deployments

### Where to encrypt
| Layer | Approach |
|---|---|
| Storage volume / disk | Provider-managed (cloud-side) AES-256 — turn it on by default; verify FIPS-validated module if required by the customer |
| Database transparent disk encryption (TDE) | Yes, but protects only against media theft; does not protect against logical compromise |
| Per-column / per-field application-layer encryption | For direct identifiers, free text containing PHI, and tokens — envelope encryption with KMS-managed keys |
| Object storage | Server-side encryption with customer-managed keys (SSE-KMS / SSE-CMK pattern) |
| Backups, snapshots, replicas | Inherit storage encryption; verify cross-region replicas also encrypted |
| Logs (which may contain PHI) | Encrypt at rest; restrict who can decrypt |

### Application-layer encryption pattern (envelope)
1. Application generates / fetches a data encryption key (DEK)
2. DEK encrypts the field
3. DEK is wrapped by a key encryption key (KEK) held in KMS / HSM
4. Encrypted DEK + encrypted field stored together
5. To read: app calls KMS to unwrap DEK, then decrypts locally

This pattern keeps the master key in the HSM and limits KMS calls to wraps/unwraps, not every read.

## In Transit

### Protocols
- **TLS 1.2 or higher**, prefer **TLS 1.3** for new deployments
- Disable SSLv2, SSLv3, TLS 1.0, TLS 1.1 (verify current NIST/PCI guidance — most environments treat 1.0/1.1 as deprecated)
- Cipher suites: modern AEAD (ChaCha20-Poly1305, AES-GCM); avoid RC4, 3DES, anonymous suites
- Strict certificate validation; HSTS for browser endpoints
- mTLS for service-to-service where the threat model warrants

### Healthcare protocol overlays
| Protocol | Approach |
|---|---|
| HL7 v2 over MLLP | Tunnel through TLS (MLLP-S), VPN, or SFTP for batch; raw MLLP on the internet is unsafe |
| FHIR over HTTPS | TLS + OAuth2 + SMART app authorization; verify SMART-on-FHIR scopes |
| DICOM | DICOM TLS profile aligned with IHE ATNA; DIMSE-TLS or DICOMweb over HTTPS |
| Direct (NHIN Direct) | S/MIME with cert-based trust |
| sFTP for bulk | SSH-based, key auth, no password fallback |
| Email | TLS where supported, S/MIME for end-to-end if required; secure messaging portals for patient-facing |

### Patient-facing channels
- SMS reminders: unencrypted in transit; document patient choice; minimum-necessary content
- Patient portal: TLS + strong auth; consider phishing-resistant MFA
- Email to patient: TLS where supported; patient should be informed of risk

## Key Management

### Key custody
- Cloud KMS (AWS KMS, GCP KMS, Azure Key Vault) for most workloads; Managed HSM tier for stronger isolation
- On-premises / dedicated HSMs validated to **FIPS 140-2 / 140-3** at the level your customer/regulator requires (verify current validation status of the specific module)
- Bring-your-own-key (BYOK) or hold-your-own-key (HYOK) patterns for tenants that require cryptographic control

### Key separation
- Distinct keys per tenant where blast radius matters (BAA scope, regulated customer)
- Distinct keys per data class (PHI vs. operational metadata)
- Distinct keys per environment (prod / staging / dev — and don't use production keys to encrypt non-production data)

### Key lifecycle
- Generation: HSM-backed RNG
- Distribution: never in plaintext outside the HSM
- Rotation: define and enforce a rotation policy; rotate on suspected compromise; envelope encryption makes rotation cheap (re-wrap DEKs, leave ciphertext alone)
- Revocation: ability to disable a KEK quickly (effectively cryptographic erasure of all data wrapped by it)
- Destruction: NIST SP 800-88 sanitization for keys at end of life

### Access to keys
- Strict IAM — separation of duties between key admin (creates / rotates) and key user (encrypts / decrypts)
- Logging every key-use operation to an immutable audit store
- Break-the-glass procedure for emergency decryption with enhanced logging and review (see audit-logging skill)

## The HIPAA Encryption "Safe Harbor"

The HIPAA Breach Notification Rule defines "unsecured PHI" as PHI not rendered unusable, unreadable, or indecipherable to unauthorized persons through a technology or methodology specified by the HHS Secretary. The guidance referenced under the Rule generally points to encryption meeting NIST standards (for data at rest and in transit) and destruction methodologies (NIST SP 800-88).

If PHI is encrypted to these specifications and the keys are not compromised, lost/stolen encrypted PHI typically does not trigger breach notification — this is sometimes called the "encryption safe harbor."

Verify the current text of the rule and the current HHS guidance — the specific standards referenced have evolved. Encryption alone does not satisfy the Security Rule.

## Encryption in Use (Confidential Computing)

Emerging patterns for sensitive workloads:
- **TEE / SGX / TDX / SEV-SNP**: hardware-isolated execution
- **Homomorphic encryption**: compute on encrypted data (high overhead; narrow use cases)
- **Secure multi-party computation**: split-trust computation across parties
- **Federated learning with secure aggregation**: keep PHI local, share only model updates

These are not yet standard controls; treat as defense-in-depth or as enabling for specific use cases (multi-party analytics, cross-organization research). Verify against current NIST and FIPS guidance.

## Common Pitfalls

- Treating "TLS 1.2 enabled" as sufficient without disabling old versions / weak ciphers
- Using the default cloud-managed key when the customer's BAA requires customer-managed
- Hard-coding keys or storing keys in source control
- Same key across prod, staging, and dev
- Encrypting at rest but logging plaintext PHI to a separate log store that isn't encrypted
- Forgetting backups, snapshots, and read replicas
- Long-lived bearer tokens that effectively bypass encrypted transport
- Cryptographic erasure without verifying the key was actually destroyed across all copies and HA replicas

## Final Disclaimers

- Verify current NIST cryptographic standards (SP 800-131A, SP 800-175B, etc.) before committing to specific algorithms and key sizes
- Verify current FIPS 140 validation status of any module you rely on
- Consult your security architect and privacy/compliance counsel; this cheatsheet is a working aid, not a binding standard
