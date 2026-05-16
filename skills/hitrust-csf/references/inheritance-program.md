# HITRUST Shared Responsibility and Inheritance Program

A working reference for how HITRUST's Shared Responsibility and Inheritance Program lets cloud customers and downstream vendors leverage their providers' HITRUST-certified controls. Verify the current program rules, the current list of participating CSPs, and the current published inheritance scope for each provider against HITRUST's documentation. Consult an Authorized External Assessor for binding inheritance decisions in your specific MyCSF assessment.

## What Inheritance Solves

Modern healthcare workloads run on shared infrastructure: public cloud (AWS, Azure, GCP, OCI), SaaS platforms, managed services, and downstream sub-processors. Without inheritance, every customer of a CSP would have to independently test and document the same underlying infrastructure controls — wasted effort and duplicate audit work.

HITRUST's Shared Responsibility and Inheritance Program lets the **service provider** undergo a HITRUST assessment of the controls it owns, and lets **customers inherit those controls** in their own MyCSF assessment — with documented mapping of which controls are provider-managed, which are customer-managed, and which are shared.

## Roles

| Role | What they do |
|---|---|
| **Service provider (CSP / vendor)** | Holds a HITRUST assessment covering its in-scope infrastructure / platform. Publishes an inheritance package describing the controls it covers and the conditions of inheritance. |
| **Customer** | Maps its workload to the provider's services, inherits the in-scope controls, and assesses its own customer-side controls. |
| **External Assessor** | Validates the customer's inheritance claims against the provider's published scope and the customer's actual usage. |
| **HITRUST** | Operates the program, maintains the published inheritance catalogs, and quality-assures the assessments. |

## Shared Responsibility Model

Inheritance only works if the customer and provider agree on which side owns which control. Typical split for cloud infrastructure:

| Control area | Provider owns | Customer owns |
|---|---|---|
| Physical security of data centers | Provider | — |
| Hypervisor / host OS | Provider | — |
| Network — provider backbone | Provider | — |
| Network — customer VPC / VNet config | — | Customer |
| Identity — provider control plane | Provider | — |
| Identity — customer's users, MFA, RBAC | — | Customer |
| OS patching | Provider for managed services; Customer for IaaS-VMs | Mixed |
| Application code | — | Customer |
| Data encryption | Provider for "encryption-at-rest by default"; Customer for app-layer encryption and key management when CMK | Mixed |
| Configuration of services | — | Customer |
| Logging | Provider emits service logs; Customer ingests, retains, monitors | Mixed |
| Backups | Provider for service-level; Customer for app-level | Mixed |

The exact split depends on the service (IaaS vs. PaaS vs. SaaS) and the specific offering. The provider's published inheritance package documents the line.

## Publishing and Consuming Inheritance

### From the Provider Side

A HITRUST-certified provider publishes:
- The HITRUST assessment scope (which services / regions / products are in scope)
- The control inheritance map (which CSF controls the provider covers, fully or partially)
- The conditions of inheritance (e.g., "customer must be using service X in the us-east-1 region with default-encryption enabled")
- The validity window of the inheritance (tied to the provider's certification cycle)

### From the Customer Side

To inherit controls in your r2 assessment:
1. Identify the in-scope services you use
2. Verify your usage matches the provider's stated conditions
3. In MyCSF, claim inheritance for the applicable controls
4. Document the evidence of inheritance (provider's certification, the customer's matching usage, configuration evidence)
5. The External Assessor verifies the inheritance claims

## Major CSP Participation (Verify Current Status)

The major public CSPs (AWS, Microsoft Azure, Google Cloud, Oracle Cloud Infrastructure) have at various times participated in HITRUST's inheritance program. Verify the **current** participating providers, current covered service scope, and current HITRUST CSF version against HITRUST's published list. Coverage by service is granular — not every service is in scope; not every region is in scope; not every feature is in scope.

## What Inheritance Does NOT Cover

Inheritance is **not** "we use AWS, so we're HITRUST." Customers remain fully responsible for:

- **Configuration of inherited services**: A misconfigured S3 bucket / Blob container / GCS bucket is a customer-side failure regardless of CSP certification
- **Identity and access management** for the customer's users and workloads
- **Data classification, retention, and disposal**
- **Application-layer security**
- **Encryption choices above the platform default** (CMK / BYOK, key separation, rotation)
- **Logging configuration, retention, and review**
- **Incident response for customer-side events**
- **BAA execution** (HIPAA contractual layer — see hipaa-compliance)
- **Workforce training, sanctions, awareness**
- **The customer's own controls outside the cloud boundary** (offices, endpoints, employees, on-prem systems)

A common mistake is over-inheriting — claiming inheritance for controls the provider doesn't actually cover or the customer's configuration doesn't qualify for. External Assessors verify; over-claims become assessment findings.

## Inheritance from Non-CSP Vendors

Inheritance is not limited to public CSPs. Any HITRUST-certified vendor whose service is in scope of the customer's assessment can publish inheritance. Common cases:

- Identity-as-a-Service providers (the customer's IdP)
- Managed SOC / SIEM providers
- Healthcare-specific SaaS (EHR vendors, FHIR API vendors, terminology services)
- Application-platform-as-a-service providers

For multi-tenant SaaS that touches PHI, requesting the vendor's HITRUST inheritance package is part of vendor due diligence (alongside SOC 2 Type 2, BAA, and pen-test results).

## Lifecycle Considerations

- **Provider's certification expires** → customer's inheritance becomes invalid until the provider re-certifies. Plan for renewal cycles
- **Provider changes scope** → some services drop in or out of inheritance scope; verify on each annual / biannual cycle
- **Customer's usage changes** → adding a new region, a new service, or a new architecture may move controls from inherited to customer-owned
- **Customer's assessment cycle** is independent from the provider's; inheritance is a snapshot at assessment time

## Practical Workflow

1. **Inventory the cloud services and SaaS** in scope of your HITRUST assessment
2. **Pull each vendor's current HITRUST inheritance package**
3. **Map your usage** against the inheritance conditions
4. **Identify gaps** where you cannot inherit (out-of-scope services, regions, configurations)
5. **Claim inheritance in MyCSF** with documented evidence
6. **External Assessor validates** during the assessment
7. **Refresh annually** as part of vendor risk management

## Final Disclaimers

- Verify current HITRUST program rules and participating providers against HITRUST's published documentation
- Verify each provider's current inheritance scope and conditions
- Inheritance is per-control, per-service, per-condition — not a blanket transfer
- Engage your External Assessor for binding inheritance decisions in your specific assessment
