# HIPAA Safe Harbor Identifiers — Removal Guidance

The 18 categories of identifiers that must be removed for HIPAA Safe Harbor de-identification, with practical removal guidance for each. Verify the current text of the rule before publishing or sharing de-identified data; consult your privacy/compliance counsel for binding interpretation. Even after removing all 18, the covered entity must have "no actual knowledge" that the residual data could re-identify an individual — verify residual risk per the Expert Determination process if risk is non-trivial.

## The 18 Identifier Categories

### 1. Names
**Remove**: First, middle, last, nicknames, maiden names, prefixes/suffixes, names of relatives, names of employers, names of family members appearing in narrative.
**Notes**: NER for clinical text. Don't forget free-text fields like "Reason for visit," "History," "Notes."

### 2. Geographic subdivisions smaller than a state
**Remove**: Street address, city, county, precinct, ZIP code.
**Retain**: State, first three digits of ZIP only if the geographic unit defined by those three digits contains more than 20,000 people; otherwise mask to `000`.
**Notes**: Verify current Census-based determination of qualifying 3-digit ZIPs.

### 3. All elements of dates (except year) directly related to an individual
**Remove**: Date of birth, admission date, discharge date, death date, date of service when tied to an individual; any date narrower than year.
**Retain**: Year alone is acceptable.
**Aggregate**: All ages over 89 and dates that would convey such age must be collapsed to `90+` or equivalent.

### 4. Telephone numbers
**Remove**: All forms — landline, mobile, fax-as-phone, international.

### 5. Fax numbers
**Remove**: All fax numbers.

### 6. Email addresses
**Remove**: All email addresses, including those in narrative text.

### 7. Social Security numbers
**Remove**: Full SSN, last-4 SSN (which is still an identifier), and any masked SSN where the masking could be reversed by linkage.

### 8. Medical record numbers
**Remove**: MRN as assigned by the source EHR. If a non-identifying internal key is needed, generate a random token; do not derive it from the MRN.

### 9. Health plan beneficiary numbers
**Remove**: Subscriber IDs, group IDs, Medicare/Medicaid identifiers tied to the individual.

### 10. Account numbers
**Remove**: Patient account numbers, billing account numbers, any financial account number tied to the individual.

### 11. Certificate / license numbers
**Remove**: Driver's license, passport, professional license numbers — both the individual's and any third party named in the record.

### 12. Vehicle identifiers and serial numbers, including license plate numbers
**Remove**: VINs, license plate numbers, any vehicle-linked identifier.

### 13. Device identifiers and serial numbers
**Remove**: Implanted device serials, pacemaker IDs, pump serials, RFID tags, smartphone IMEI/UDID where tied to the individual.

### 14. Web URLs
**Remove**: Personal URLs, patient portal URLs that contain identifying parameters, any URL fragment that resolves to the individual.

### 15. IP addresses
**Remove**: Source IP addresses tied to the individual's session, device, or home network.
**Notes**: Static IPs and high-resolution geolocated IPs can re-identify; pseudonymize at minimum.

### 16. Biometric identifiers
**Remove**: Fingerprint templates, voice prints, retinal/iris scans, palm vein, face recognition templates, gait signatures.

### 17. Full-face photographs and any comparable images
**Remove**: Patient photos, photos in scanned ID cards. Crop, blur faces, or obtain authorization.
**Notes**: Other photos (skin, dermatology, fundus) may still be identifying — assess.

### 18. Any other unique identifying number, characteristic, or code
**Catch-all**. Examples that have tripped organizations up:
- Very rare diagnoses combined with small geographic units
- Unique procedure dates combined with unique provider
- Internal patient nicknames or shorthand codes
- Open-text "comments" fields that contain identifying detail

## The "No Actual Knowledge" Clause

Even after removing all 18 above, Safe Harbor requires the covered entity has **no actual knowledge** that the remaining information could be used alone or in combination with other reasonably available information to identify an individual. Examples of when actual knowledge applies:

- Knowing the recipient has access to a re-identification dataset
- Knowing a specific patient has a rare condition that is in the dataset
- Knowing a public news event identifies a patient who would otherwise be in the dataset

If actual knowledge exists, Safe Harbor is not available — use Expert Determination or another path.

## Common Removal Pitfalls

- **Year-from-date columns**: dropping the day/month but leaving an admission timestamp accurate to the minute
- **3-digit ZIP override**: leaving the full ZIP because the system rejected the masked value
- **Free text**: structured fields are clean but narrative still contains "I saw Mr. Smith on Tuesday"
- **Quasi-identifiers in combination**: each individual field passes Safe Harbor but the combination is uniquely identifying (rare diagnosis + year + state)
- **Internal IDs derived from MRN**: hashed MRN is still re-identifiable if anyone holds the hash + table
- **Burned-in DICOM PHI**: image headers cleaned, but pixel-baked text overlooked
- **Audio**: transcript de-identified, but the voice itself is a biometric

## When Safe Harbor Is Not Enough

If the residual data is still identifying or if dates/geography are essential to the use case, consider:

- **Expert Determination** — see de-identification-techniques.md
- **Limited Data Set + Data Use Agreement** — retains dates and limited geographic detail, still PHI, requires DUA
- **Strong access controls plus pseudonymization** — keeps the data as PHI but limits blast radius
- **Synthetic data** — only if the synthesis method has documented privacy guarantees

Consult your Privacy Officer and a qualified statistician before relying on any of these for an external release.
