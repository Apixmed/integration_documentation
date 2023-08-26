# Treatment checker integration guide

Here, you'll find step-by-step documentation and clear instructions for integrating your applications with Apixmed treatment checker.

Sandbox
- API URL: https://dev.api.apixmed.com
- WEB APP URL: https://dev.apixmed.com

Production
- API URL: https://api.apixmed.com
- WEB APP URL: https://apixmed.com

> [IMPORTANT]
> If your organization intends to utilize the sandbox as well, it is necessary to follow the same process.

## 1. Setup your organisation profile and account

Currently, Apixmed does not have a B2B backoffice in place. Our team will be responsible for setting up this system manually. Kindly prepare the following information and share it with our team.

- **Organization**
  - **Id** - (*required*) an alphanumeric string (English letters and numbers); this Id will be used in Apixmed API and front-end URLs.
  - **Name** - (*required*) the user-friendly name of the organization, intended for display to users on the Apixmed platform.
  - **Description** - (*required*) a concise description of your organization, also intended for user display on the Apixmed platform (recommended to not exceed 100 words or 500 characters).

- **Account**
  - **Id** - (*required*) an alphanumeric string (English letters and numbers); this Id will be used in Apixmed API URLs.
  - **Name** - (*required*) the user friendly name of account that will be used for internal and cross-team usage (users will not be able to see this).
  - **Secret** - (*required*) an alphanumeric string (English letters and numbers) that will be used for checksum generation.
  - **ApplyPayloadEncryption** - (*required*) boolean value where `true` means that an additional encryption will be applied to payload provided to Apixmed during treatment checker report generation.
  - **PublicKey** - (*required in case when **ApplyPayloadEncryption** is `true`*) a string value that will be used for decryption payload during the treatment checker report generation.
  - **UseFhirServer** - (*required*) a boolean value where `true` means that Apixmed will be allowed to use FHIR server (see [FHIR documentation](http://hl7.org/fhir/)) of your organization to retrieve patient information.
  - **FhirServerUrl** - (*required in case when **UseFhirServer** is `true`*) a base API URL of your organization FHIR server.
  - **FhirServerCreds** - (*required in case when **UseFhirServer** is `true`*) creds for Apixmed to access your FHIR server to retrieve patient data.

<details>
  <summary>Example</summary>

- **Organization**
  - Id = `"TestOrganization"`
  - Name = `"TestOrganizationName"`
  - Description = `"Test Organization Description: Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed eleifend nisl quis nisl egestas, at rhoncus tortor eleifend. Fusce finibus eros justo, nec vehicula sem elementum eu. Aliquam at ligula id dolor pretium tempus. Donec malesuada est non arcu consectetur aliquam. Curabitur ornare vitae justo vel ultricies. Nulla facilisi. Sed aliquet nulla enim, tristique consequat diam tincidunt at. Proin convallis venenatis mi in tincidunt. Aenean mi magna, molestie quis mauris id, aliquam finibus felis. Donec sed turpis a metus placerat ullamcorper. Mauris dapibus porta nibh, ac sodales eros fermentum et."`

- **Account**
  - **Id** = `"testorganizationaccount"`
  - **Name** = `"Test organization primary account"`
  - **Secret** = `"TestOrganizationSecret"`
  - **ApplyPayloadEncryption** = `false`
  - **PublicKey** = `null`
  - **UseFhirServer** = `false`
  - **FhirServerUrl** = `null`
  - **FhirServerCreds** = `null`

</details>

## 2. Collect user data for treatment checker

Collect user data via your application and let the user to define the treatment.

**Required data:**
- **Date** - represents the date (yyyyMMddhhmmsszzz) of the request payload's creation. It serves the purpose of setting a boundary for the object's lifetime, with a maximum allowable lifetime of 24 hours.
- **RequestId** - is a universally unique identifier (UUID/GUID) that is generated on the client side. It functions as a distinctive and exclusive identifier, ensuring the uniqueness of each request.

- **Email** - will be used to send Apixmed URL to see the report to user
- **Gender** - user gender
- **Age** - user age, in years
- **Weight** - user weight, in kilograms
- **Height** - user height, in centimeters
- **DiseaseIds** - array of diagnosted diseases
- **MedicationIds** - array of medications that are prescribed or used by the customer

Any non-mandatory data can be included within the custom section of the payload, as outlined in the subsequent step.

Diseases and medications are the required part of any treatment and Apixmed provide endpoints to retrieve this data as options for combobox controls.

### Diseases

- HTTP method: `GET`
- URL: `api/{organizationId}/{organizationAccountId}/tags/diseases?searchPattern={searchParameter}`
- Language can be defined by header: `Accept-Language` with allowed values `en` `uk`

**Request parameters:**

- organizationId - Id of your organization.
- organizationAccountId - account Id of your organization.
- searchParameter - parameter to filter diseases by name.

**Response** will contain array of DiseaseCombobox objects (array will contain no more than 50 items that corresponds to search parameter):

```
DiseaseCombobox
{
    "id": string,
    "displayText": string,
    "type": string
}
```

<details>
  <summary>Possible values for type property</summary>

- infectious
- neoplasm
- blood
- endocrine
- mental
- nerve
- eye
- ear
- circulatory
- respiratory
- digestive
- skin
- muscle
- genitourinary
- pregnancy
- perinatal
- congenital
- symptoms
- injury
- morbidity
- other
- specific
- healthStatus
- externalMorbidity
- undefined

</details>

<details>
  <summary>Example</summary>

**Request**

`GET` `api/TestOrganization/testorganizationaccount/tags/diseases?searchPattern=heart`

Headers:
- `Accept-Language`: `en`

**Response**

```json
[
  {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "displayText": "Heart failure",
    "type": "circulatory"
  },
  {
    "id": "08f33877-67e4-44cd-8006-1d57e4b827b6",
    "displayText": "Heart aneurysm",
    "type": "circulatory"
  }
]
```

</details>

### Medications

- HTTP method: `GET`
- URL: `api/{organizationId}/{organizationAccountId}/tags/medications?searchPattern={searchParameter}`
- Language can be defined by header: `Accept-Language` with allowed values `en` `uk`

**Request parameters:**

- organizationId - Id of your organization.
- organizationAccountId - account Id of your organization.
- searchParameter - parameter to filter diseases by name.

**Response** will contain array of MedicationCombobox objects (array will contain no more than 50 items that corresponds to search parameter):

```
MedicationCombobox
{
    "id": string,
    "displayText": string,
    "type": string
}
```

<details>
  <summary>Possible values for type property</summary>

- alimentaryTractAndMetabolism
- bloodAndBloodFormingOrgans
- cardiovascularSystem
- dermatologicals
- genitoUrinarySystemAndSexHormones
- systemicHormonalPreparations
- antiinfectivesForSystemicUse
- antineoplasticAndImmunomodulatingAgents
- musculoSkeletalSystem
- nervousSystem
- antiparasiticProductsInsecticidesAndRepellents
- respiratorySystem
- sensoryOrgans
- various
- undefined

</details>

<details>
  <summary>Example</summary>

**Request**

`GET` `api/TestOrganization/testorganizationaccount/tags/medications?searchPattern=heart`

Headers:
- `Accept-Language`: `en`

**Response**

```json
[
  {
    "id": "62620cca-cd26-459d-9668-79316e94b407",
    "displayText": "Aspirin",
    "type": "nervousSystem"
  },
  {
    "id": "e278a595-4f7e-4910-b4dc-97393b42130c",
    "displayText": "Ibuprofen",
    "type": "musculoSkeletalSystem"
  }
]
```

</details>

## 3. Perform treatment check via HTTP(S) API call

### Endpoint information: request

- HTTP method: `POST`
- URL: `api/treatment-checker/generate/{organizationId}/{organizationAccountId}`
- Language can be defined by header: `Accept-Language` with allowed values `en` `uk`

```
Payload
{
  "patientData": object (PatientData),
  "patientDataCustom" : object (PatientDataCustom),
  "date": string,
  "requestId": string
  "checksum": string
}
```

```
PatientData object
{
    "email": string,
    "gender": string,
    "age": number,
    "weight": number,
    "height": number,
    "diseaseIds": string[],
    "medicationIds": string[]
}
```

```
PatientDataCustom object
{
  // this object depends on data that your organization can provide
  // contact our team to agree its format
}
```

<details>
  <summary>Gender values</summary>

- male
- female
- other
- unknown

</details>

### Checksum calculation

Checksum should be calculated based on values of PatientData object and system predefined fields ('Date' and 'RequestId') and your organization account secret by concatenating all field to a single string:

`{date}{requestId}{email}{gender}{age}{weight}{height}{diseaseIds}{medicationIds}{secret}`

**arrays must be sorted alphabetically and only then concatenated to a single string*

This string should be hashed with SHA512 (see [wikipedia](https://en.wikipedia.org/wiki/SHA-2)).

### [OPTIONAL] Applying payload encryption

If your organization want to apply additional protection to patient data (PatientDataCustom will not be saved on Apixmed side after report creation - only the report will be saved), payload encryption can be used.

You should generate private and public keys via open ssl and share public key with our team.

When payload is created, just encrypt it with your organization private key and send to the same Apixmed endpoint.

### Endpoint information: response

```
Response
{
  "reportId": string,
  "reportUrl": string
}
```

### Report as JSON

Also you are able to get the report as `json` via Apixmed API endpoint:

- HTTP method: `GET`
- URL: `api/treatment-checker/{organizationId}/reports/{reportId}`
- Language can be defined by header: `Accept-Language` with allowed values `en` `uk`

### Examples

<details>
  <summary>Example for checksum calculation</summary>

- Define PatientData object
```
PatientData object
{
    "email": "example@email.com",
    "gender": "male",
    "age": 40,
    "weight": 85,
    "height": 182,
    "diseaseIds": [
      "disease-id-1",
      "disease-id-2"
    ],
    "medicationIds": [
      "medication-id-2",
      "medication-id-1"
    ],
}
```
- Concatenate fields: `example@email.commale4085182disease-id-1disease-id-2medication-id-1medication-id-2`
- Attach secret (example value secret is `example-secret`): `example@email.commale4085182disease-id-1disease-id-2medication-id-1medication-id-2example-secret`
- Hash with SHA-512: `bf3c70a67149dcb9d88a63647a9436dd369184193237f6c5cc9bfbade2067882a75972fe5139abbd4233efd49c9dc8822b3e191b00e2076200603bdc24b53e8f`

</details>

<details>
  <summary>Example of full payload</summary>

`POST` `api/treatment-checker/generate/TestOrganization/testorganizationaccount`

Header:
  - `Accept-Language`: `en`

```json
{
  "patientData": {
    "email": "example@email.com",
    "gender": "male",
    "age": 40,
    "weight": 85,
    "height": 182,
    "diseaseIds": [
      "disease-id-1",
      "disease-id-2"
    ],
    "medicationIds": [
      "medication-id-2",
      "medication-id-1"
    ],
  },
  "patientDataCustom": null,
  "checksum": "bf3c70a67149dcb9d88a63647a9436dd369184193237f6c5cc9bfbade2067882a75972fe5139abbd4233efd49c9dc8822b3e191b00e2076200603bdc24b53e8f"
}
```

</details>

<details>
  <summary>Example of response</summary>

```json
{
  "reportId": "0000000-1111-0000-0000-000000000001",
  "reportUrl": "https://apixmed.com/treatment-checker/TestOrganization/reports/0000000-1111-0000-0000-000000000001"
}
```

</details>

## 4. Redirect your user to the report

The most simple flow is to redirect user to the Apixmed Web App URL where he will be able to check his report by report Id:

`treatment-checker/{organizationId}/reports/{reportId}`

The alternative option is to open the report URL in iframe.

<details>
  <summary>Example</summary>

`treatment-checker/TestOrganization/reports/0000000-1111-0000-0000-000000000001`

</details>

***Also user will be notified on provided email with Apixmed URL to open generated report.***
