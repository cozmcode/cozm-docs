# 🧾 Cozm Travels API Documentation

---

## 1. Introduction

The **Cozm Applications API** streamlines the submission and management of A1, COC and Immigration certificate applications for cross-border employment. It is built to:

- Authenticate authorized clients securely
- Retrieve dynamic form fields based on country and compliance type
- Submit completed applications
- Manage documentation and workflows

This API is ideal for organizations handling international work assignments and automating compliance tasks from initiation to submission.

---

## 2. API Base URL

```
https://api.development.cozmtravels.democozm.com/
```

---

## 3. Authentication

The API requires a **Bearer token** and a custom **Version header**.

### Request Headers

```http
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
Version: **redacted**
```

---

## 4. API Endpoints

### a. 🔐 Authentication

`POST /api/users/token/`

Retrieves three tokens:

- `access_token` – *(not needed)*
- `id_token` – **Used as Bearer token**
- `refresh_token` – *(not needed)*

#### Request Fields

| Key        | Type     | Required | Description   | Value             |
|------------|----------|----------|---------------|-------------------|
| `email`    | `string` | Yes      | User email    | **redacted**      |
| `password` | `string` | Yes      | User password | **redacted**      |

#### Example (cURL)

```
curl --location 'https://api.development.cozmtravels.democozm.com/api/users/token/' \
     --header 'Version: **redacted**' \
     --form 'email="**redacted**"' \
     --form 'password="**redacted**"'
```

The returned `id_token` needs to be used as the Bearer token in further requests.

---

### b. 📄 Form Fields

`GET /api/compliance/fields`

Retrieves dynamic fields for A1 applications based on:

- `country`: Country for which the form is requested (e.g. `US`)
- `form_type`: Form type (e.g. `COC`)

#### Query Parameters

| Key            | Type     | Required | Description                                          |
|----------------|----------|----------|------------------------------------------------------|
| `country`      | `string` | Yes      | 2-letter ISO code of the country (e.g. `US`)         |
| `form_type`    | `string` | Yes      | Form/Compliance type (e.g. `COC`, `MSW-A1`, `A1`)    |

#### Example (cURL)

```
curl --location 'https://api.development.cozmtravels.democozm.com/api/compliance/fields?country=US&form_type=COC' \
     --header 'Authorization: Bearer YOUR_ACCESS_TOKEN' \
     --header 'Version: **redacted**'
```

### Compliance Types

The API supports several types of compliance forms that can be used with the `/api/compliance/fields` endpoint:

| Type    | Description                                                                                                                                                |
|---------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `A1`    | Standard A1 certificate for EU social security coverage during temporary work assignments within the EU/EEA/Switzerland. Required for single-country postings. |
| `MSW-A1`| Multi-State Worker A1 certificate for employees regularly working in multiple EU countries. Covers social security compliance across multiple jurisdictions.   |
| `COC`   | Certificate of Coverage for social security agreements with non-EU countries. Maintains social security coverage during overseas assignments.            |
| `ETA`   | Electronic Travel Authorization for temporary entry into specific countries. Required for business travel to certain destinations.                            |
| `BV`    | Business Visa application for temporary business activities in foreign countries. Required for specific business-related activities abroad.                   |

These compliance types can be used as the `form_type` parameter when calling the `/api/compliance/fields` endpoint to retrieve the appropriate form fields. For example:

```
GET /api/compliance/fields?country=US&host_country=DE&form_type=COC
```

The response will contain all required fields specific to that compliance type and country.

### Field Attributes Explained

The main key returned by the API call above is the `fields` array.
Each element of this array roughly represents one question asked on the corresponding questionnaire
and contains a number of attributes which are described below.

The set of fields in the `fields` array is different for each country and form type.

| Attribute            | Description                                                                                  |
|----------------------|----------------------------------------------------------------------------------------------|
| `name`               | Internal field identifier used as key (e.g. `employee_first_name`)                           |
| `type`               | Data format or UI control, see below for more information (e.g. `string`, `date`, `boolean`) |
| `group`              | Logical grouping, mostly used on the UI (e.g. `Personal details`)                            |
| `description`        | The actual question in human-readable format (e.g. "First name")                             |
| `persona`            | Role the field is meant for (see below)                                                      |
| `required`           | `true` or `false` – indicates if field is mandatory                                          |
| `max_length`         | Maximum number characters allowed (only relevant for `string` inputs)                        |
| `choices`            | Predefined dropdown label-value pairs (one of which must be used as the value when sending a request). The label has to be displayed to the user, and the corresponding value has to be sent in the API request payload.   |
| `parent_value`       | Used in conditional logic. The field only needs to be specified in the payload, if the parent field's value matches this.                    |
| `conditional_fields` | Nested fields, relevant depending on the current field's value                               |
| `placeholder`        | Placeholder text shown to user (e.g. "John", only used on the UI)                            |
| `extra_validations`  | Regex or additional validations, the value for this field needs to pass the validation       |

### Persona Types

| Persona         | Description                                                                  |
|-----------------|------------------------------------------------------------------------------|
| `Employee`      | Field to be filled by the employee or traveler                               |
| `Employer`      | Field to be completed by the employer and visible to the employee            |
| `Employer only` | Field to be completed by the employer and invisible to the employee          |
| `Assumption`    | Data that can be pre-filled (e.g. nationality, country of residence)         |

---

### c. 📄 Generate Upload File URLs

`POST /api/compliance/document_upload_url/`

Generates pre-signed URLs for secure file uploads to AWS S3. These URLs are valid for 15 minutes and allow direct file uploads to S3.

#### Request Payload

| Key           | Type            | Required | Description                                    |
|---------------|-----------------|----------|------------------------------------------------|
| `file_names`  | array of string | Yes      | List of file names to generate upload URLs for |

#### Example Request (cURL)

```bash
curl --location 'https://api.development.cozmtravels.democozm.com/api/compliance/document_upload_url/' \
     --header 'Authorization: Bearer YOUR_ACCESS_TOKEN' \
     --header 'Version: **redacted**' \
     --header 'Content-Type: application/json' \
     --data '{
         "file_names": [
             "passport_size_photo.jpg",
             "passport_scan.pdf"
         ]
     }'
```

#### Response Format

The API returns an array of objects, each containing:

| Field            | Type   | Description                                               |
|------------------|--------|-----------------------------------------------------------|
| `file_name`      | string | Original file name provided in the request                |
| `object_key`     | string | Unique identifier for the file in S3 (UUID)              |
| `pre_signed_url` | string | Temporary URL for uploading the file directly to S3       |

#### Example Response

```json
[
    {
        "file_name": "passport_size_photo.jpg",
        "object_key": "01d8ead3-cb37-4751-9e4b-0452214d3bfc",
        "pre_signed_url": "https://some-bucket.s3.amazonaws.com/01d8ead3-cb37-4751-9e4b-0452214d3bfc?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=..."
    },
    {
        "file_name": "passport_scan.pdf",
        "object_key": "a68bb5b0-e30f-4d1d-b0e8-798b804781f6",
        "pre_signed_url": "https://some-bucket.s3.amazonaws.com/a68bb5b0-e30f-4d1d-b0e8-798b804781f6?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=..."
    }
]
```

#### Usage Notes

1. The pre-signed URLs expire after 15 minutes
2. Use the returned `object_key` when referencing uploaded files in other API calls, e.g. when submitting the application.
3. Supported file types include JPG, JPEG, and PDF formats
4. Files should be uploaded using an HTTP PUT request to the pre-signed URL
5. Include the appropriate `Content-Type` header when uploading (e.g., `image/jpeg`, `application/pdf`)

---

### d. 📄 Upload Files to S3

`PUT <pre_signed_url>`

After obtaining the pre-signed URL from the previous endpoint, use it to upload the actual file to AWS S3.

#### Request Headers

| Header         | Value                                | Required | Description                                    |
|---------------|--------------------------------------|----------|------------------------------------------------|
| `Content-Type` | `image/jpeg`, `image/jpg`, `application/pdf` | Yes      | The MIME type of the file being uploaded       |

#### Request Body

The file content should be sent as binary data in the request body.

#### Example Request (cURL)

```bash
curl --location --request PUT 'https://some-bucket.s3.amazonaws.com/<object_key>?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=....' \
     --header 'Content-Type: image/jpeg' \
     --data-binary '/path/to/your/file.jpg'
```

#### Example Request (Postman)

1. Create a new request with method `PUT`
2. Enter the pre-signed URL as the request URL
3. Set the appropriate `Content-Type` header
4. In the "Body" tab:
   - Select "binary"
   - Click "Select File" and choose your file
5. Send the request

#### Response

A successful upload will return an HTTP 200 status code with no response body.

#### Usage Notes

1. The upload must be completed before the pre-signed URL expires (15 minutes from generation)
2. The `Content-Type` header must match the actual file type being uploaded
3. The file size and type must match the constraints of your application
4. No additional authentication headers are needed - the pre-signed URL contains the necessary credentials

---

### e. 📨 Submit A1 Application

`POST /api/compliance/requests/create`

Submits a new A1 application with metadata and field data. Use the dynamic fields retrieved via `/api/compliance/fields`.

#### Request Payload

| Key                         | Type                | Required | Description                                          |
|-----------------------------|---------------------|----------|------------------------------------------------------|
| `home_country`              | string              | Yes      | 2-letter ISO code of origin country (e.g. `US`)      |
| `nationality`               | string              | Yes      | Applicant's nationality                              |
| `persona`                   | string (UUID)       | Yes      | UUID of the employer entity (this is hard-coded for now, it will be removed in the future.)       |
| `compliance_type`           | string              | Yes      | Same as `form_type` in the `/api/compliance/fields` endpoint. One of: `COC`, `MSW-A1`, `A1`, `ETA`, `BV`.      |
| `is_complete`               | boolean             | Yes      | Indicates the form is ready for submission, set this to `true`           |
| `is_submitted_by_assistant` | boolean             | No       | Indicates the from was submitted by an assistant     |
| `uploaded_files`            | array               | No       | Files attached with submission                       |
| `host_countries`            | array of strings    | Yes      | List of destination countries (e.g., `["UK", "AT"]`) |
| `start_date`                | string (YYYY-MM-DD) | Yes      | Assignment start date                                |
| `expiry_date`               | string (YYYY-MM-DD) | Yes      | Assignment end date                                  |
| `fields`                    | object              | Yes      | Key-value pairs of form fields                       |

#### Signature Format

If the form includes signature fields, they must drawn on a canvas and the image must be **Base64-encoded**:

```json
"employee_signature": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAUA..."
```

#### Uploaded Files

Any uploaded files must be included in the `uploaded_files` array.

The `uploaded_files` array contains objects with the name and `aws_object_key` of the uploaded files.

```json
"uploaded_files": [
    {
        "name": "blank.pdf",
        "aws_object_key": "64599c9c-72cd-466c-be79-7ae59171e2bf"
    },
    {
        "name": "blank.pdf",
        "aws_object_key": "64599c9c-72cd-466c-be79-7ae59171e2bf"
    }
]
```

### f. 📋 List Submitted Applications

`GET /api/compliance/compliance-requests/`

Retrieves a paginated list of all submitted applications.

#### Request Headers

| Header          | Value                      | Required | Description                |
|-----------------|----------------------------|----------|----------------------------|
| `Version`       | `YOUR_VERSION`             | Yes      | API version identifier     |
| `Authorization` | `Bearer YOUR_ACCESS_TOKEN` | Yes      | Authentication token       |

#### Example Request (cURL)

```bash
curl --location 'https://api.development.cozmtravels.democozm.com/api/compliance/compliance-requests/' \
     --header 'Authorization: Bearer YOUR_ACCESS_TOKEN' \
     --header 'Version: **redacted**'
```

#### Response Format

The response includes pagination information and an array of application objects:

| Field         | Type    | Description                                    |
|--------------|---------|------------------------------------------------|
| `page_number` | integer | Current page number                            |
| `page_size`   | integer | Number of items per page                       |
| `page_count`  | integer | Total number of pages                          |
| `count`       | integer | Total number of items                          |
| `next`        | string  | URL for next page (null if last page)         |
| `previous`    | string  | URL for previous page (null if first page)    |
| `results`     | array   | List of application objects                    |

Each application object in the `results` array contains:

| Field                         | Type    | Description                                    |
|-------------------------------|---------|------------------------------------------------|
| `uuid`                        | string  | Unique identifier for the application          |
| `home_country`                | string  | Country code of origin                         |
| `host_countries`              | array   | List of destination country codes              |
| `compliance_type`             | string  | Type of application (e.g., `COC`, `BV`)        |
| `status`                      | string  | Current status (e.g., `DRAFTED`, `FILED`)      |
| `first_name`                  | string  | Employee's first name                          |
| `last_name`                   | string  | Employee's last name                           |
| `email`                       | string  | Employee's email address                       |
| `compliance_type_description` | string  | Human-readable compliance type                 |
| `persona_name`                | string  | Name of the employer entity                    |
| `start_date`                  | string  | Assignment start date (YYYY-MM-DD)             |
| `expiry_date`                 | string  | Assignment end date (YYYY-MM-DD)               |
| `issue_date`                  | string  | Date when application was submitted            |
| `days_to_expiry`              | integer | Days remaining until expiry                    |
| `created_at`                  | string  | Application creation timestamp                 |
| `reference_details`           | object  | Additional reference information               |
| `comment`                     | string  | Optional comments on the application           |
| `case_manager`                | string  | Assigned case manager                          |
| `reference_id`                | string  | External reference ID                          |
| `application_link`            | string  | Not implemented yet                            |

#### Example Response

```json
{
    "page_number": 1,
    "page_size": 20,
    "page_count": 1,
    "count": 12,
    "next": null,
    "previous": null,
    "results": [
        {
            "uuid": "b312b8ee-0336-438c-b5e7-d7b2dbcf7ga3",
            "home_country": "AT",
            "host_countries": ["UK"],
            "compliance_type": "BV",
            "status": "DRAFTED",
            "first_name": "Sandbox",
            "last_name": "Cozm",
            "email": "sandbox@thecozm.com",
            "compliance_type_description": "Business Visa",
            "persona_name": "The Cozm -  UK",
            "start_date": "2025-05-01",
            "expiry_date": "2025-05-23",
            "created_at": "2025-05-01T13:42:54.468333Z",
            "days_to_expiry": 22
        }
    ]
}
```

---

## 5. 🔄 Conditional Fields Logic

The form uses **conditional fields** to dynamically display form inputs based on user responses. Each conditional field is linked to a parent field and only appears when the parent has a specific value (`parent_value`).

The form also supports recursive conditional logic to allow the addition of multiple host countries for Multi-state (MSW-A1) applications.

This logic ensures that users only see fields relevant to their situation, making the form cleaner and easier to complete.

### Structure Example

```json
{
  "name": "parent_field",
  "type": "string",
  "conditional_fields": [
    {
      "name": "child_field",
      "type": "string",
      "parent_value": "Expected Value"
    }
  ]
}
```

This logic helps:
- Show only relevant fields
- Support dynamic forms
- Handle nested, multi-step inputs (e.g., MSW-A1)

---

## 6. 🧩 Field Data Types

| Data Type                          | Description                                                   | Example Input               |
|------------------------------------|---------------------------------------------------------------|-----------------------------|
| `application_from`                 | Start date of assignment (can be in the past or future)       | `2024-01-01`                |
| `application_to`                   | End date of assignment (must be after `application_from`)     | `2024-12-31`                |
| `date_past`                        | Past date (e.g. employment start)                             | `2023-01-15`                |
| `date_of_birth`                    | Date of birth (must be at least 18 years before current date) | `1990-05-20`                |
| `string`                           | General text input                                            | `John Doe`                  |
| `user_email`                       | Email address                                                 | `john.doe@company.com`      |
| `employer_name` (`string`)         | Company name                                                  | `XYZ Corporation`           |
| `employer_street_name` (`string`)  | Company's street name                                         | `Main St`                   |
| `employer_house_number` (`string`) | Company's house number                                        | `456`                       |
| `employer_postal_code` (`string`)  | Company's postal code                                         | `10001`                     |
| `employer_city` (`string`)         | Company's city/town                                           | `London`                    |
| `nationality`                      | Nationality (must be a country name)                          | `United States`             |
| `all_country`                      | Country field (must be a country name)                        | `Germany`                   |
| `host_country`                     | Country field (must be a supported destination country)       | `France`                    |
| `boolean`                          | true/false toggle                                             | `true`                      |
| `phone`                            | Phone number                                                  | `+12025550123`              |
| `attachment_jpg_jpeg`              | JPG/JPEG image (use `object_key` from Upload File URLs API)   | `01d8ead3-cb37-4751-9e4b-0452214d3bfc` |
| `passport_attachment`              | Passport image in JPG/JPEG or PDF format (use `object_key` from Upload File URLs API) | `01d8ead3-cb37-4751-9e4b-0452214d3bfc` |
| `passport_attachment_jpg_jpeg`     | Passport JPG/JPEG image (use `object_key` from Upload File URLs API) | `01d8ead3-cb37-4751-9e4b-0452214d3bfc` |
| `signature`                        | Base64-encoded signature image                                | `data:image/png;base64,...` |

> ℹ️ Other data types may be used. Contact the Cozm team if needed.

---

## 7. ⚠️ Error Handling

### Common Error Codes

| Status Code | Error Message             | Description                                |
|-------------|---------------------------|--------------------------------------------|
| 400         | Bad Request               | Invalid parameters                         |
| 401         | Unauthorized              | Missing/invalid token                      |
| 403         | Forbidden                 | Insufficient permissions                   |
| 404         | Not Found                 | Resource not found                         |
| 500         | Internal Server Error     | Something went wrong on the server         |

### Example Error Response

```json
{
  "detail": "Incorrect authentication credentials."
}
```

---

## 8. 🧪 Postman Collection

To simplify testing, please request the prebuilt Postman collection and access credentials from the Cozm team.

### Authentication Credentials

- **Email:** `**redacted**`
- **Password:** `**redacted**`

### Included Requests

1. **Set Token**
   - Go to Body > `form-data`
   - Enter email and password
   - Click *Send* to set the token automatically in the environment variables.

2. **A1/MSW/COC Applications**
   - US COC Form
   - Submit US COC Application

3. **VISA Applications**
   - UK ETA Form
   - Submit UK ETA Application
   - UK BV Form
   - Submit UK BV Application

4. **File Management**
   - Generate File Upload URLs
   - Upload File to S3

5. **Application Management**
   - Submitted Applications List

### Email Inbox Credentials

- **Email:** `**redacted**`
- **Password:** `**redacted**`

Please contact the Cozm team to obtain the above credentials.

---
