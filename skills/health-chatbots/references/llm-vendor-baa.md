# HIPAA-Eligible LLM Vendors and BAA Paths

This reference covers the major LLM providers that offer paths to a Business Associate Agreement (BAA) for healthcare workloads. **Sending PHI to a non-BAA-covered LLM endpoint is the single most common compliance failure in healthcare LLM projects.**

**Verify all BAA scopes, covered models, regions, and contractual terms directly with each vendor before relying on any of the content below. BAA scopes evolve, and which specific models and features are covered changes frequently.**

## Why a BAA matters

Under HIPAA, a covered entity (provider, payer, clearinghouse) or its business associate cannot disclose PHI to a third party without either patient authorization or a Business Associate Agreement with that third party. An LLM provider receiving PHI is a business associate in that flow.

Without a BAA: the disclosure is generally not permitted; using the LLM with PHI puts the covered entity in non-compliance and the LLM provider potentially in liability.

## The major paths

### 1. Microsoft Azure OpenAI Service

- **BAA**: covered under the Microsoft Online Services BAA (part of the Microsoft Online Services DPA / Online Services Terms framework). Healthcare customers enroll in the BAA via their Microsoft volume-licensing or enterprise agreement.
- **Models**: OpenAI-hosted models (GPT-4 family, GPT-3.5 family, embeddings models) running on Azure infrastructure. **Verify which specific models are in BAA scope** at the time of use; new models often have a lag before BAA-eligibility is confirmed.
- **Regions**: data residency / processing region matters; pin inference to a region that supports your HIPAA posture. Some preview features may use different regions than GA features.
- **Security posture**:
  - Private endpoints (Azure Private Link).
  - Customer-managed keys (CMK) for encryption at rest.
  - Network isolation via VNet integration.
  - Diagnostic logs in your subscription.
- **Notes**: Azure OpenAI is what most teams mean by "ChatGPT" in healthcare contexts — but it is **not** the consumer ChatGPT product, which is not BAA-covered.

### 2. AWS Bedrock

- **BAA**: AWS HIPAA-eligible services list; Amazon Bedrock is on the list. Healthcare customers operate under the AWS BAA executed at the account / organization level.
- **Models**: Bedrock provides multiple foundation models from various providers (Anthropic Claude, Meta Llama, Mistral, Cohere, AI21, Amazon Titan, Amazon Nova). **Verify which specific foundation models within Bedrock are covered** — the AWS HIPAA Eligibility scope can differ per model and per Bedrock feature.
- **Regions**: BAA-covered usage typically restricted to specific AWS regions; verify your regions.
- **Security posture**:
  - VPC endpoints for private connectivity.
  - KMS customer-managed keys.
  - CloudTrail logging.
  - IAM-scoped access.
- **Notes**: When using Anthropic Claude on Bedrock, the BAA path runs through AWS — confirm with AWS account team.

### 3. Google Vertex AI

- **BAA**: Google Cloud Platform HIPAA-covered products; Vertex AI is on the covered list. Healthcare customers operate under the Google Cloud BAA.
- **Models**: Vertex AI provides Google's first-party models (Gemini family, PaLM family) and integrates third-party models. **Verify which specific models and features within Vertex AI are BAA-covered** — partner-model and tuning-feature coverage can differ.
- **Regions**: BAA-covered usage typically restricted to specific GCP regions; verify.
- **Security posture**:
  - VPC Service Controls.
  - Customer-managed encryption keys (CMEK).
  - Cloud Audit Logs.
  - IAM-scoped access.
- **Notes**: When using Anthropic Claude on Vertex AI, the BAA path runs through Google — confirm with Google Cloud account team.

### 4. Anthropic (direct)

- **BAA**: Anthropic offers a BAA for direct API customers on appropriate plans. **Verify current BAA terms and the specific plans / commitments required**.
- **Models**: Claude family (Opus, Sonnet, Haiku — current naming).
- **Regions**: data processing regions per Anthropic's current data-handling practices; verify.
- **Notes**: Three paths to Anthropic Claude with a BAA exist — direct Anthropic, Amazon Bedrock, Google Vertex AI. Each path's BAA terms come from the platform offering the path (direct from Anthropic, AWS, or Google respectively). Confirm which path your architecture uses and which BAA is operative.

### 5. Other paths to be aware of (verify per vendor)

- **Oracle Cloud Infrastructure (OCI) Generative AI** — Oracle has healthcare-eligibility programs; verify per service.
- **IBM watsonx** — IBM offers BAA-eligible cloud paths for healthcare; verify per service.
- **Open-source models self-hosted** — if you host the model yourself (within your own BAA-covered cloud account), the BAA question is between you and your cloud provider; the model itself is just software. This shifts the operational burden to your team.

### What is NOT BAA-covered (typical)

- **OpenAI direct ChatGPT** (consumer or ChatGPT Plus / Pro) — no BAA. Even ChatGPT Enterprise has historically not included a HIPAA BAA from OpenAI directly — **verify current** OpenAI BAA terms.
- **OpenAI direct API** (api.openai.com without Azure) — verify current; historically not BAA-covered, though OpenAI has signaled changes. Confirm at the time of use.
- **Anthropic free-tier / consumer Claude.ai** — no BAA.
- **Google Gemini consumer / Gemini Advanced** — no BAA (Vertex AI is the BAA path for Google's models).
- **xAI Grok**, **Mistral direct**, **Cohere direct (non-Bedrock)** — verify per vendor's current BAA posture.

## Engineering pattern

### Default-deny outbound

- The gateway / proxy that handles LLM calls must default-deny any outbound call to an endpoint not on a BAA allowlist.
- Allowlist is **explicit per endpoint and per region** — `https://your-azure-openai.openai.azure.com/...` in your BAA-covered region, not "anything openai.com."

### PHI redaction for non-BAA paths

- If a non-BAA endpoint must be called for a legitimate non-PHI use case (e.g., summarizing a public policy document), the gateway redacts known PHI tokens (names, MRN, DOB, full address, phone, email, SSN) before egress.
- Re-inject after response only if needed.
- Log the redaction event for audit.

### Multi-vendor strategy

When you need multiple LLMs in the same product:

- **Vendor A (BAA)** for PHI-bearing turns.
- **Vendor B (no BAA)** only for de-identified or non-PHI prompts (e.g., summarizing a public policy document, generating non-PHI marketing copy).
- **Gateway routes** at the prompt level based on classification.

Document the routing rules. Re-evaluate routing when prompts change.

### Egress logging

For every LLM call, log:

- Timestamp
- User / session identifier
- Prompt template ID and rendered prompt (or a redacted version)
- Retrieved documents IDs
- Model ID and version
- Region
- Response (or a redacted version)
- Latency
- Token usage
- Safety classifier verdicts

Treat the log as PHI itself. Store in a BAA-covered location with the same access controls as the chart.

### Region pinning

- Pin inference to the BAA-covered region(s).
- Some preview / multi-region features can route inference outside the pinned region — disable those features in production for healthcare workloads.

### Customer-managed keys

- Where supported, use customer-managed encryption keys (CMK / CMEK).
- Key rotation policy aligned with your security baseline.

### Audit trail

- Every BAA allowlist change is recorded with who, when, and why.
- Every region change is recorded.
- Every model upgrade is recorded.

## Common failure modes

- **Wrong service** — using consumer ChatGPT instead of Azure OpenAI Service "because they're the same models."
- **Wrong region** — calling a BAA-covered service from a non-BAA region.
- **Preview features** — using a preview feature whose BAA status hasn't been confirmed.
- **New model** — using a model that's not yet in BAA scope for that platform.
- **Third-party plugin / connector** — adding a plugin that egresses data to a non-BAA destination.
- **Embedding service** — assuming the chat model is BAA-covered but using a separate embedding service that isn't.
- **Telemetry leak** — vendor telemetry that captures PHI in URLs or error logs.
- **Sub-processor surprise** — a vendor uses a sub-processor that handles PHI without its own BAA chain.

## Verify-current sources

- Microsoft Azure: HIPAA / HITECH compliance documentation; Azure OpenAI Service product documentation; Microsoft Online Services BAA terms.
- AWS: HIPAA Eligible Services Reference; AWS Business Associate Addendum; Bedrock documentation.
- Google Cloud: HIPAA Implementation Guide; Google Cloud BAA terms; Vertex AI documentation.
- Anthropic: trust / HIPAA / BAA documentation on Anthropic's website; sales / legal contacts.
- Your contracts / legal team for confirmed BAA scope per vendor.

Always confirm in writing with each vendor before going to production with PHI traffic.
