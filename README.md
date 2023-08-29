# Treatment checker integration guide (ver. 0.1.0)

Here, you'll find step-by-step documentation and clear instructions for integrating your applications with Apixmed treatment checker.

Sandbox
- API URL: https://dev.api.apixmed.com
- WEB APP URL: https://dev.apixmed.com

Production
- API URL: https://api.apixmed.com
- WEB APP URL: https://apixmed.com

> [IMPORTANT]
> 
> If your organization intends to utilize the sandbox as well, it is necessary to follow the same process.
> 
> Also there is test organization on samdbox (but its usage is limited); see its parameters provided in this guide.

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
  - Id = `"test"`
  - Name = `"TestOrganizationName"`
  - Description = `"Test Organization Description: Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed eleifend nisl quis nisl egestas, at rhoncus tortor eleifend. Fusce finibus eros justo, nec vehicula sem elementum eu. Aliquam at ligula id dolor pretium tempus. Donec malesuada est non arcu consectetur aliquam. Curabitur ornare vitae justo vel ultricies. Nulla facilisi. Sed aliquet nulla enim, tristique consequat diam tincidunt at. Proin convallis venenatis mi in tincidunt. Aenean mi magna, molestie quis mauris id, aliquam finibus felis. Donec sed turpis a metus placerat ullamcorper. Mauris dapibus porta nibh, ac sodales eros fermentum et."`

- **Account**
  - **Id** = `"testpremium"`
  - **Name** = `"Test organization primary account"`
  - **Secret** = `"secret12312312312"`
  - **ApplyPayloadEncryption** = `false`
  - **PublicKey** = `null`
  - **UseFhirServer** = `false`
  - **FhirServerUrl** = `null`
  - **FhirServerCreds** = `null`

</details>

## 2. Collect user data for treatment checker

Collect user data via your application and let the user to define the treatment.

**Required data:**
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

`GET` `api/test/testpremium/tags/diseases?searchPattern=heart`

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

`GET` `api/test/testpremium/tags/medications?searchPattern=heart`

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
    "requestId": string,
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
    "userCorrelationId": string
    // Other fields of this object depends on data that your organization can provide. Contact our team to agree its format.
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

Each payload that contains checksum must also contain request protection fields:

- **Date** - represents the date (yyyyMMddhhmmss) of the request payload's creation. It serves the purpose of setting a boundary for the object's lifetime, with a maximum allowable lifetime of 24 hours on production.
- **RequestId** - is an identifier of operation on client side and must be unique for each request (if you do not track these requests on your side, just generate GUID/UID for every new payload). It functions as a distinctive and exclusive identifier, ensuring the uniqueness of each request.

Checksum should be calculated based on request protection fields ('date' and 'requestId'), values of PatientData object and your organization account secret by concatenating all field to a single string:

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

### Report (short) as JSON

Also you are able to get the report as `json` via Apixmed API endpoint:

- HTTP method: `GET`
- URL: `api/treatment-checker/{organizationId}/reports/{reportId}`
- Language can be defined by header: `Accept-Language` with allowed values `en` `uk`

### Examples

<details>
  <summary>Example for checksum calculation</summary>

- Define date and requestId
```json
{
    "date": "20230829080101",
    "requestId": "00000000-0000-0000-0001-000000000001"
}
```
- Define PatientData object
```
PatientData object
{
    "email": "support@apixmed.com",
    "gender": "male",
    "age": 30, 
    "weight": 85,
    "height": 182,
    "diseaseIds": [ "7b42cffc-d931-4604-82e8-5e505ca70d68" ],
    "medicationIds": [ "a8c39e64-b5b8-41f6-9a2f-0974a0f8ccc4" ]
}
```
- Concatenate fields: `2023082908010100000000-0000-0000-0001-000000000001support@apixmed.commale30851827b42cffc-d931-4604-82e8-5e505ca70d68a8c39e64-b5b8-41f6-9a2f-0974a0f8ccc4`
- Attach secret (here secret value is `secret12312312312`): `2023082908010100000000-0000-0000-0001-000000000001support@apixmed.commale30851827b42cffc-d931-4604-82e8-5e505ca70d68a8c39e64-b5b8-41f6-9a2f-0974a0f8ccc4secret12312312312`
- Hash with SHA-512: `11dcbb170c7455a9445e445ffd92070bd4163715192bfb36456ed531ab9297814faba03edd6b1b88d6b0219dfe75cf8a023f04c05e4825dde2dbe009a8d7b4dd`

</details>

<details>
  <summary>Example of full payload</summary>

`POST` `api/treatment-checker/generate/test/testpremium`

Header:
  - `Accept-Language`: `en`

```json
{
    "date": "20230829080101",
    "requestId": "00000000-0000-0000-0001-000000000001",
    "patientData": {
        "email": "support@apixmed.com",
        "gender": "male",
        "age": 30, 
        "weight": 85,
        "height": 182,
        "diseaseIds": [ "7b42cffc-d931-4604-82e8-5e505ca70d68" ],
        "medicationIds": [ "a8c39e64-b5b8-41f6-9a2f-0974a0f8ccc4" ]
    },
    "patientDataCustom": {
        "userCorrelationId": "TestUser"
    },
    "checksum": "11dcbb170c7455a9445e445ffd92070bd4163715192bfb36456ed531ab9297814faba03edd6b1b88d6b0219dfe75cf8a023f04c05e4825dde2dbe009a8d7b4dd"
}
```

Full request code
```
POST /api/treatment-checker/generate/test/testpremium HTTP/1.1
Host: dev.api.apixmed.com
Accept-Language: en
Content-Type: application/json
Cookie: ARRAffinity=89efd111daa527202afd96706276879a655817e73c32546fe91ee34fcedb8b04; ARRAffinitySameSite=89efd111daa527202afd96706276879a655817e73c32546fe91ee34fcedb8b04
Content-Length: 620

{
    "date": "20230829080101",
    "requestId": "00000000-0000-0000-0001-000000000001",
    "patientData": {
        "email": "support@apixmed.com",
        "gender": "male",
        "age": 30, 
        "weight": 85,
        "height": 182,
        "diseaseIds": [ "7b42cffc-d931-4604-82e8-5e505ca70d68" ],
        "medicationIds": [ "a8c39e64-b5b8-41f6-9a2f-0974a0f8ccc4" ]
    },
    "patientDataCustom": {
        "userCorrelationId": "TestUser"
    },
    "checksum": "11dcbb170c7455a9445e445ffd92070bd4163715192bfb36456ed531ab9297814faba03edd6b1b88d6b0219dfe75cf8a023f04c05e4825dde2dbe009a8d7b4dd"
}
```

</details>

<details>
  <summary>Example of response</summary>

```json
{
    "reportId": "3a903870-9991-482d-551f-08dba868e05e",
    "reportUrl": "https://dev.apixmed.com/treatment-checker/test/reports/3a903870-9991-482d-551f-08dba868e05e"
}
```

</details>

## 4. Show the report to the user

### Show by direct URL

The most simple flow is to redirect user to the Apixmed Web App URL where he will be able to check his report by report Id:

`treatment-checker/{organizationId}/reports/{reportId}`

The alternative option is to open the report URL in iframe.

<details>
  <summary>Example</summary>

`https://dev.apixmed.com/treatment-checker/test/reports/3a903870-9991-482d-551f-08dba868e05e`

</details>

***Also user will be notified on provided email with Apixmed URL to open generated report.***

### Show reports generated by user

There are two ways to show treatment checker history for your user:
1. Pass `userCorrelationId` field in `PatientDataCustom` object for each report generated by your organization. Use array of items from response to get reportId and then show concrete report as described below.
2. Implement history of reports on your side (based on requestId and reportId).

### Endpoint information: retrieve treatment cheker history for user

- HTTP method: `POST`
- URL: `/api/treatment-checker/getbulk/{organizationId}/{organizationAccountId}`
- Language can be defined by header: `Accept-Language` with allowed values `en` `uk`

```
Payload
{
    "date": string
    "requestId": string,
    "indexFrom": number,
    "pageIndex": number,
    "pageSize": number,
    "filter": {
        "userCorrelationIds": string[]
    },
    "checksum": string
}
```

Checksum should be calculated based on request protection fields ('date' and 'requestId') and your organization account secret by concatenating all field to a single string:

`{date}{requestId}{secret}`

**arrays must be sorted alphabetically and only then concatenated to a single string*

This string should be hashed with SHA512 (see [wikipedia](https://en.wikipedia.org/wiki/SHA-2)).

<details>
  <summary>Example of full request code</summary>

```
POST /api/treatment-checker/getbulk/test/testpremium HTTP/1.1
Host: dev.api.apixmed.com
Accept-Language: en
Content-Type: application/json
Cookie: ARRAffinity=89efd111daa527202afd96706276879a655817e73c32546fe91ee34fcedb8b04; ARRAffinitySameSite=89efd111daa527202afd96706276879a655817e73c32546fe91ee34fcedb8b04
Content-Length: 377

{
    "date":"20230829094130",
    "requestId": "00000000-0000-0000-0002-000000000001",
    "indexFrom": null,
    "pageIndex": null,
    "pageSize": 50,
    "filter": {
        "UserCorrelationIds": ["TestUser"]
    },
    "checksum": "7b6d3d28e588f10f1210323c12ced0319d95aa4702eeada482b343c77663d943a08e745bfbe603f464723309e1cbc96b349451b8a32c1e6b34d1c6155e14e6c2"
}
```

</details>

<details>
  <summary>Example of response</summary>

```
{
    "indexFrom": 1,
    "pageIndex": 1,
    "pageSize": 50,
    "sorting": "Id",
    "filter": {
        "userCorrelationIds": [
            "TestUser"
        ],
        "organizationAccountId": "testpremium",
        "organizationId": "test"
    },
    "items": [
        {
            "id": "3a903870-9991-482d-551f-08dba868e05e",
            "createdOn": "2023-08-29T08:46:29.0566667+00:00",
            "treatmentInfo": {
                "age": 30,
                "height": 182.0,
                "weight": 85.0,
                "diseases": [
                    {
                        "id": "7b42cffc-d931-4604-82e8-5e505ca70d68",
                        "displayText": "Heart failure, unspecified",
                        "type": "circulatory"
                    }
                ],
                "medications": [
                    {
                        "id": "a8c39e64-b5b8-41f6-9a2f-0974a0f8ccc4",
                        "displayText": "Warfarin sodium",
                        "type": "bloodAndBloodFormingOrgans"
                    }
                ]
            },
            "cautions": [
                {
                    "type": "isNotPartOfAnyDiseaseTreatment",
                    "severity": "warning",
                    "parameters": [
                        "Warfarin sodium"
                    ]
                },
                {
                    "type": "bMIAdultOverweight",
                    "severity": "info",
                    "parameters": [
                        "25.66",
                        "18",
                        "25"
                    ]
                },
                {
                    "type": "treatmentGuidelinesAvailable",
                    "severity": "info",
                    "parameters": [
                        "Heart failure",
                        "Cardiovascular disease",
                        "Heart disease"
                    ]
                }
            ]
        }
    ],
    "hasNextPage": false,
    "hasPreviousPage": false,
    "totalCount": 1,
    "totalPages": 1
}
```

</details>
