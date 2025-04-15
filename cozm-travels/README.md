# 🧾 Cozm Travels API Documentation

---

## 1. Introduction

The **Cozm Applications API** streamlines the submission and management of A1 and COC certificate applications for cross-border employment. It is built to:

- Authenticate authorized clients securely
- Retrieve dynamic form fields based on country and compliance type
- Submit completed applications
- Manage documentation and workflows

This API is ideal for organizations handling international work assignments and automating social security compliance tasks from initiation to submission.

---

## 2. API Base URL

```
https://api.development.cozmtravels.democozm.com/
```

---

## 3. Authentication

The API requires a **Bearer token** and a custom **Version header**.

### Example Request Headers

```http
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
Version: FRAGOMEN
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
     --header 'Version: FRAGOMEN' \
     --form 'email="**redacted**"' \
     --form 'password="**redacted**"'
```

The returned `id_token` needs to be used as the Bearer token in further requests.

---

### b. 📄 Form Fields

`GET /api/compliance/fields`

Retrieves dynamic fields for A1 applications based on:

- `country`: Origin country (e.g. `US`)
- `host_country`: Destination work country (e.g. `DE`)
- `form_type`: Form type (e.g. `COC`)

#### Query Parameters

| Key            | Type     | Required | Description                                          |
|----------------|----------|----------|------------------------------------------------------|
| `country`      | `string` | Yes      | 2-letter ISO code of the home country (e.g. `US`)    |
| `host_country` | `string` | Yes      | 2-letter ISO code of destination country (e.g. `DE`) |
| `form_type`    | `string` | Yes      | Compliance type (e.g. `COC`, `MSW-A1`, `A1`)         |

#### Example (cURL)

```
curl --location 'https://api.development.cozmtravels.democozm.com/api/compliance/fields?country=US&host_country=DE&form_type=COC' \
     --header 'Authorization: Bearer YOUR_ACCESS_TOKEN' \
     --header 'Version: FRAGOMEN'
```

### Field Attributes Explained

The main key returned by the API call above is the `fields` array.
Each element of this array roughly represents one question asked on the corresponding questionnaire
and contains a number of attributes which are described below.

The set of fields in the `fields` array is different for each home country and form type.

| Attribute            | Description                                                                                  |
|----------------------|----------------------------------------------------------------------------------------------|
| `name`               | Internal field identifier used as key (e.g. `employee_first_name`)                           |
| `type`               | Data format or UI control, see below for more information (e.g. `string`, `date`, `boolean`) |
| `group`              | Logical grouping, mostly used on the UI (e.g. `Personal details`)                            |
| `description`        | The actual question in human-readable format (e.g. "First name")                             |
| `persona`            | Role the field is meant for (see below)                                                      |
| `required`           | `true` or `false` – indicates if field is mandatory                                          |
| `max_length`         | Maximum number characters allowed (only relevant for `string` inputs)                        |
| `choices`            | Predefined dropdown values (one of which must be used as the value when sending a request)   |
| `parent_value`       | Used in conditional logic. The field only needs to be specified in the payload, if the parent field’s value matches this.                    |
| `conditional_fields` | Nested fields, relevant depending on the current field's value                               |
| `placeholder`        | Placeholder text shown to user (e.g. "John", only used on the UI)                            |
| `extra_validations`  | Regex or additional validations, the value for this field needs to pass the validation       |

### Persona Types

| Persona      | Description                                                                  |
|--------------|------------------------------------------------------------------------------|
| `Employee`   | Field to be filled by the employee or traveler                               |
| `Employer`   | Field to be completed by the employer                                        |
| `Assumption` | Pre-filled data that can be changed (e.g. nationality, country of residence) |

---

### c. 📨 Submit A1 Application

`POST /api/compliance/requests/create`

Submits a new A1 application with metadata and field data. Use the dynamic fields retrieved via `/api/compliance/fields`.

#### Request Payload

| Key                         | Type                | Required | Description                                          |
|-----------------------------|---------------------|----------|------------------------------------------------------|
| `home_country`              | string              | Yes      | 2-letter ISO code of origin country (e.g. `US`)      |
| `nationality`               | string              | Yes      | Applicant's nationality                              |
| `persona`                   | string (UUID)       | Yes      | UUID of the employer entity (this is hard-coded for now, it will be removed in the future.)       |
| `compliance_type`           | string              | Yes      | Same as `form_type` in the `/api/compliance/fields` endpoint. One of: `COC`, `MSW-A1`, `A1`.                        |
| `is_complete`               | boolean             | Yes      | Indicates the form is ready for submission           |
| `is_submitted_by_assistant` | boolean             | No       | Indicates the from was submitted by an assistant     |
| `uploaded_files`            | array               | No       | Files attached with submission                       |
| `host_countries`            | array of strings    | Yes      | List of destination countries (e.g., `["UK", "AT"]`) |
| `start_date`                | string (YYYY-MM-DD) | Yes      | Assignment start date                                |
| `expiry_date`               | string (YYYY-MM-DD) | Yes      | Assignment end date                                  |
| `fields`                    | object              | Yes      | Key-value pairs of form fields                       |

#### Signature Format

If the form includes signature fields, they must be **Base64-encoded**:

```json
"employee_signature": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAUA..."
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
   - Click *Send* to get the token

2. **US COC Form**
   - Retrieve form fields for the U.S.

3. **Submit US COC Application**
   - Submits the form
   - Sends confirmation email to `**redacted**`

### Email Inbox Credentials

- **Email:** `**redacted**`
- **Password:** `**redacted**`

---
