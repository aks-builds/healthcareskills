# IHE Profile Catalog

Practical catalog of IHE integration profiles you'll meet most often, organized by domain. Each entry includes purpose, common use cases, and the FHIR-based equivalent where one exists. **Always verify against the current IHE Technical Framework volumes** — profile names, transaction numbers, and supplements get updated annually.

## Contents

- IT Infrastructure (ITI) Profiles
- Patient Care Coordination (PCC) Profiles
- Radiology (RAD) Profiles
- Laboratory (LAB) Profiles
- Patient Administration Management (PAM)
- Pharmacy (PHARM)
- Quality, Research, and Public Health (QRPH)
- Devices (DEV) / Service Delivery (SD)
- Choosing the Right Profile
- FHIR-Based Equivalents at a Glance

---

## IT Infrastructure (ITI) Profiles

### Document Sharing Family

| Profile | Purpose | FHIR equivalent |
|---------|---------|-----------------|
| **XDS.b** Cross-Enterprise Document Sharing | Registry-based document sharing within an affinity domain. Source → Repository → Registry; Consumers query Registry, retrieve from Repository. SOAP/MTOM-based. | MHD |
| **XDR** Cross-Enterprise Document Reliable Interchange | XDS-shaped point-to-point push without a Registry. Use when no shared registry exists. | MHD push |
| **XDM** Cross-Enterprise Document Media Interchange | XDS-shaped metadata packaged for USB, CD, email attachment (ZIP containing METADATA.XML and document files). | — |
| **XCA** Cross-Community Access | Federation of XDS affinity domains. Initiating Gateway queries multiple Responding Gateways. | MHDS (in progress) |
| **XCA-I** Cross-Community Access for Imaging | XCA for DICOM imaging cross-community. | — |
| **XCPD** Cross-Community Patient Discovery | Patient identifier discovery across communities, prerequisite to XCA. | PIXm + PDQm cross-community |
| **MHD** Mobile access to Health Documents | FHIR-based document sharing using DocumentReference, DocumentManifest (or List), and Binary. | (this IS the FHIR version) |
| **MHDS** Mobile Health Document Sharing | All-FHIR document-sharing architecture — re-frames the entire XDS affinity domain on FHIR. | (this IS the FHIR version) |
| **DSUB** Document Metadata Subscription | Subscribe to events on the Registry (new documents matching criteria). | FHIR Subscription |
| **DSG** Document Digital Signature | Embed digital signatures in document submissions. | (FHIR Provenance + signature) |
| **NPFS** Non-Patient File Sharing | XDS-shaped sharing of non-patient documents (clinical guidelines, etc.). | mNPFS (FHIR-based variant) |
| **RMU** Restricted Metadata Update | Update document metadata after submission. | — |
| **RMD** Remove Metadata and Documents | Allow correction/withdrawal of submitted content. | — |

### Identity

| Profile | Purpose | FHIR equivalent |
|---------|---------|-----------------|
| **PIX** Patient Identifier Cross-Referencing | Maintain and query cross-references across patient ID domains. HL7 v2 ADT-based. | PIXm |
| **PIXm** Patient Identifier Cross-Referencing for Mobile | FHIR-based PIX — uses `$ihe-pix` operation on Patient. | (this IS the FHIR version) |
| **PDQ** Patient Demographics Query | Demographic search returning candidate patients. HL7 v2 QBP/RSP-based. | PDQm |
| **PDQm** Patient Demographics Query for Mobile | FHIR-based PDQ (Patient search). | (this IS the FHIR version) |
| **XCPD** Cross-Community Patient Discovery | Federation of PIX/PDQ across communities. | mPIXm + mPDQm federation |
| **PMIR** Patient Master Identity Registry | FHIR-based patient identity registry. | (this IS the FHIR version) |

### Security, Privacy, Audit

| Profile | Purpose | FHIR equivalent |
|---------|---------|-----------------|
| **ATNA** Audit Trail and Node Authentication | Audit message format + node-to-node mutual TLS. Audit messages use DICOM/IHE audit schema over syslog (RFC 5424) TLS. | ATNA still applies; supports FHIR AuditEvent option |
| **CT** Consistent Time | NTP synchronization so audit timestamps align across nodes. | — |
| **BPPC** Basic Patient Privacy Consents | Capture and enforce consent acknowledgments. | FHIR Consent (basic) |
| **APPC** Advanced Patient Privacy Consents | Fine-grained consent (per purpose, per resource type, expiration). | FHIR Consent (full) |
| **EUA** Enterprise User Authentication | Kerberos-based single sign-on (legacy). | — |
| **IUA** Internet User Authorization | OAuth 2.0 / OpenID Connect for FHIR / REST APIs. | (this IS the OAuth profile) |
| **XUA** Cross-Enterprise User Assertion | SAML 2.0 assertion conveyed across affinity domains. | — |
| **SeR** Secure Retrieve | Token-based retrieval for protected content. | — |
| **DEN** De-Identification | De-identification handling guidance. | — |
| **ARR** Audit Record Repository | Profile a central audit repository receiving ATNA messages. | — |

### Subscription, Notification, Directory

| Profile | Purpose | FHIR equivalent |
|---------|---------|-----------------|
| **HPD** Healthcare Provider Directory | LDAP-based provider directory schema and protocol. | mCSD |
| **mCSD** Mobile Care Services Discovery | FHIR-based registry of organizations, locations, practitioners. | (this IS the FHIR version) |
| **mACM** Mobile Alert Communication Management | FHIR-based clinical alerting. | (this IS the FHIR version) |
| **MXQuery** Mobile Cross-Enterprise Query | FHIR query across communities. | (this IS the FHIR version) |

---

## Patient Care Coordination (PCC) Profiles

PCC profiles define **document content** — they constrain CDA / C-CDA structure for specific use cases. They typically pair with ITI document-sharing profiles for transport.

| Profile | Purpose |
|---------|---------|
| **XPHR** Personal Health Records | Patient-managed health summary document |
| **XDS-MS** Medical Summary | CDA-based medical summary for transitions of care |
| **APS** Antepartum Summary | Pregnancy summary document |
| **APR** Antepartum Record | Pregnancy record details |
| **APHP** Antepartum History and Physical | H&P during pregnancy |
| **APLI** Antepartum Lab Results | Lab results during pregnancy |
| **APE** Antepartum Education | Education materials |
| **PPHP** Postpartum History and Physical | Postpartum H&P |
| **LDS** Labor and Delivery Summary | Labor and delivery record |
| **CRC** Cancer Registry Content | Reporting to cancer registries |
| **NRP** Neonatal Record Profile | Neonatal care summary |
| **IC** Immunization Content | Immunization document content |
| **EDR** Emergency Department Referral | ED referral content |
| **EDES** Emergency Department Encounter Summary | ED visit summary |
| **DSC** Dental Summary | Dental summary content |
| **BPPC** (PCC variant) | Patient privacy consent content |
| **QRPH-CRD** Care Record | Care record content |

Most PCC profiles are now being re-published or augmented as US Core / FHIR IG aligned content.

---

## Radiology (RAD) Profiles

Heavily DICOM-centric. Pair with ITI for security and audit.

| Profile | Purpose | FHIR equivalent |
|---------|---------|-----------------|
| **SWF.b** Scheduled Workflow | RIS + modality + PACS workflow — MWL, MPPS, image storage. The core radiology workflow. | — |
| **SWF.m** Scheduled Workflow for Mobile | Mobile-friendly variant. | — |
| **XDS-I.b** Cross-Enterprise Document Sharing for Imaging | Share imaging studies across enterprises via KOS manifest documents. | — |
| **XCA-I** Cross-Community Access for Imaging | Cross-community imaging federation. | — |
| **IID** Invoke Image Display | Launch an image viewer from another application with patient/study context. | — |
| **RAD-69** Retrieve Imaging Document Set | Pull imaging studies referenced in a KOS manifest. | — |
| **REM** Radiation Exposure Monitoring | Aggregate dose data via RDSR (Radiation Dose Structured Report). | — |
| **CPI** Consistent Presentation of Images | Ensure consistent display across vendors. | — |
| **PIR** Patient Information Reconciliation | Reconcile patient identity changes affecting imaging. | — |
| **MRRT** Management of Radiology Report Templates | Standard template authoring. | — |
| **RPID** Reason for Imaging Documentation | Capture reason / indication for imaging. | — |

---

## Laboratory (LAB) Profiles

| Profile | Purpose |
|---------|---------|
| **LSWF** Laboratory Scheduled Workflow | Lab order → specimen → result workflow |
| **LCSD** Laboratory Code Set Distribution | Distribute lab code sets (LOINC, local codes) |
| **LTW** Laboratory Testing Workflow | Within-lab testing workflow |
| **LBL** Laboratory Barcode Labeling | Specimen barcode and labeling |
| **LPOCT** Laboratory Point of Care Testing | POCT workflow |
| **LDA** Laboratory Device Automation | Automation of lab instruments |
| **PaLM-RT** Pathology and Laboratory Medicine Result Reporting | Pathology / lab medicine reporting |
| **PaLM-OL** Pathology and Laboratory Medicine Order to Lab | Order-to-lab transactions |
| **APRR** Anatomic Pathology Reporting to Public Health | Public health pathology reporting |
| **APSR** Anatomic Pathology Structured Report | Pathology DICOM SR |

---

## Patient Administration Management (PAM)

| Profile | Purpose |
|---------|---------|
| **PAM** Patient Administration Management | Patient identification + encounter management workflow |
| **PEC** Patient Encounter Continuity | Encounter handoff support |

---

## Pharmacy (PHARM)

| Profile | Purpose |
|---------|---------|
| **PRE** Pharmacy Prescription | Electronic prescribing |
| **DIS** Pharmacy Dispense | Dispense reporting |
| **PADV** Pharmacy Pharmaceutical Advice | Advice / clinical decision feedback |
| **MTP** Medication Treatment Plan | Treatment plan content |
| **CMA** Community Medication Administration | Administration outside acute care |
| **CMPD** Community Pharmacy Dispense | Community dispense |

---

## Quality, Research, and Public Health (QRPH)

| Profile | Purpose |
|---------|---------|
| **eCH** Early Hearing Care Plan | Newborn hearing screening reporting |
| **VRDR** Vital Records Death Reporting | Death reporting to public health |
| **BFDR** Birth and Fetal Death Reporting | Birth / fetal death reporting |
| **CIRC** Cancer Incidence Reporting Content | Cancer registry reporting content |
| **PCC-RCC** Routine Reportable Condition | Public health case reporting |
| **eCR** Electronic Case Reporting | Public health case reporting (CDA + FHIR) |
| **DCP** Domain Coding Profiles | Code system distribution |
| **RPE** Research Public Health Exchange | Exchange for research/PH |
| **VRSR** Vital Records Birth and Fetal Death Reporting | Birth/fetal death |

eCR has both a CDA-based and a FHIR-based path (FHIR Public Health IG family).

---

## Devices (DEV) / Service Delivery

| Profile | Purpose |
|---------|---------|
| **PCD** Patient Care Device | Device-to-EHR data flow (vitals, monitors) |
| **WCM** Waveform Content Module | Continuous waveform support |
| **ACM** Alert Communication Management | Clinical alerting from devices |
| **IDCO** Implantable Device Cardiac Observation | Cardiac implantable device data |

---

## Choosing the Right Profile

| Use case | Recommended profile(s) |
|----------|------------------------|
| Document sharing within an affinity domain | XDS.b + ATNA + CT + PIX + BPPC |
| Cross-community document federation | XCA + XCPD + XUA + ATNA + CT |
| Push to one known recipient (no registry) | XDR + ATNA |
| Media interchange (USB / CD / email) | XDM |
| Mobile / web app document access | MHD + IUA + ATNA |
| All-FHIR document architecture | MHDS + IUA + ATNA |
| Cross-domain patient identifier resolution | PIX (legacy) or PIXm (FHIR) |
| Demographic search | PDQ (legacy) or PDQm (FHIR) |
| Provider directory | HPD (legacy) or mCSD (FHIR) |
| Audit + node auth | ATNA |
| Time synchronization | CT |
| Consent capture and enforcement | BPPC (basic) or APPC (granular) |
| Imaging cross-enterprise sharing | XDS-I.b + XCA-I |
| Imaging workflow (RIS + modality + PACS) | SWF.b + REM (dose) |
| External viewer launch | IID |
| Lab order-to-result workflow | LTW + LCSD + LSWF |
| Public health case reporting | eCR (CDA path) or eCR (FHIR path) |

---

## FHIR-Based Equivalents at a Glance

| Legacy | FHIR-based |
|--------|------------|
| XDS.b | MHD (and MHDS for full architecture) |
| XCA | (combinations of mPIXm + mPDQm + MHD with federation rules; MHDS unifies this) |
| PIX | PIXm |
| PDQ | PDQm |
| HPD | mCSD |
| ATNA (HL7 v3 audit messages) | ATNA still — and supports FHIR AuditEvent option |
| BPPC | APPC + FHIR Consent |
| EUA / XUA | IUA (OAuth 2.0 / OpenID Connect) |
| DSUB | FHIR Subscription |
| Document sharing across communities (XDS+XCA) | MHDS |

The direction of travel is FHIR-everywhere — but legacy XDS.b deployments are deeply embedded in HIEs, TEFCA QHIN exchange, and EHR vendors. Plan for hybrid for years.
