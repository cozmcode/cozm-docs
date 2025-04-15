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

**POST** `/api/users/token/`
Retrieves three tokens:

- `access_token` – *(not used)*
- `id_token` – **Used as Bearer token**
- `refresh_token` – *(not used)*

#### Request Fields

| Key       | Type   | Required | Description   | Value             |
|-----------|--------|----------|---------------|-------------------|
| email     | string | Yes      | User email    | **redacted**      |
| password  | string | Yes      | User password | **redacted**      |

#### Example (cURL)

```bash
curl --location 'https://api.development.cozmtravels.democozm.com/api/users/token/' \
     --header 'Version: FRAGOMEN' \
     --form 'email="**redacted"' \
     --form 'password="**redacted**"'
```

---

### b. 📄 Form Fields

**GET** `/api/compliance/fields`

Retrieves dynamic fields for A1 applications based on:

- `country`: Origin country (e.g., `US`)
- `host_country`: Destination work country
- `form_type`: Form type (e.g., `COC`, `MSW-A1`, `A1`)

#### Query Parameters

| Key         | Type   | Required | Description                                |
|-------------|--------|----------|--------------------------------------------|
| country     | string | Yes      | ISO code of the home country (e.g., `US`)  |
| host_country| string | Yes      | ISO code of destination country            |
| form_type   | string | Yes      | Type of A1 form (e.g., `COC`, `MSW-A1`)    |

#### Example (cURL)

```bash
curl --location 'https://api.development.cozmtravels.democozm.com/api/compliance/fields?country=US&form_type=COC' \
     --header 'Authorization: Bearer YOUR_ACCESS_TOKEN' \
     --header 'Version: FRAGOMEN'
```

### Field Attributes Explained

| Attribute          | Description                                                                 |
|--------------------|-----------------------------------------------------------------------------|
| `name`             | Internal field identifier (used as key)                                     |
| `type`             | Data format or UI control (e.g., `string`, `date`, `boolean`)               |
| `group`            | Logical UI grouping (e.g., `Personal details`)                              |
| `description`      | Human-readable label                                                        |
| `persona`          | Role the field is meant for (see below)                                     |
| `required`         | `true` or `false` – indicates if field is mandatory                         |
| `max_length`       | Max characters allowed (for `string` inputs)                                |
| `choices`          | Predefined dropdown values                                                  |
| `parent_value`     | Triggers conditional logic – value of the parent field to show this field   |
| `conditional_fields`| Nested fields shown based on the current field’s value                    |
| `placeholder`      | Placeholder text shown to user                                              |
| `extra_validations`| Regex or additional frontend validations                                    |

### Persona Types

| Persona    | Description                                                                              |
|------------|------------------------------------------------------------------------------------------|
| `Employee` | Field is filled by the employee or traveler                                              |
| `Employer` | Field is to be completed by the HR/employer                                              |
| `Assumption`| Auto-filled data that applies to all users (e.g., nationality, country of residence)     |

---

### c. 📨 Submit A1 Application

**POST** `/api/compliance/requests/create`

Submits a new A1 application with metadata and field data. Use the dynamic fields retrieved via `/fields`.

#### Request Payload

| Key                   | Type              | Required | Description                                          |
|------------------------|-------------------|----------|------------------------------------------------------|
| `home_country`         | string            | Yes      | ISO code of origin country                           |
| `nationality`          | string            | Yes      | Applicant's nationality                              |
| `persona`              | string (UUID)     | Yes      | UUID of the user (employee/employer/assistant)       |
| `compliance_type`      | string            | Yes      | e.g., `COC`, `MSW-A1`, `A1`                          |
| `is_complete`          | boolean           | Yes      | Indicates form is ready for submission               |
| `is_submitted_by_assistant` | boolean    | No       | Was this submitted by an assistant?                  |
| `uploaded_files`       | array             | No       | Files attached with submission                       |
| `host_countries`       | array of strings  | Yes      | List of destination countries (e.g., `["UK", "AT"]`) |
| `start_date`           | string (YYYY-MM-DD)| Yes     | Assignment start date                                |
| `expiry_date`          | string (YYYY-MM-DD)| Yes     | Assignment end date                                  |
| `fields`               | object            | Yes      | Key-value pairs of form fields                       |

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

| Data Type                    | Description                                              | Example Input             |
|-----------------------------|----------------------------------------------------------|---------------------------|
| `application_from`          | Start date of assignment                                 | `2024-01-01`              |
| `application_to`            | End date of assignment                                   | `2024-12-31`              |
| `date_past`                 | Past/historical date (e.g., employment start)            | `2023-01-15`              |
| `date_of_birth`             | Birth date                                               | `1990-05-20`              |
| `string`                    | General text input                                       | `John Doe`                |
| `user_email`                | Email address                                            | `john.doe@company.com`    |
| `employer_name`             | Company name                                             | `XYZ Corporation`         |
| `employer_street_name`      | Employer street                                          | `123 Main St`             |
| `employer_postal_code`      | Postal code                                              | `10001`                   |
| `employer_city`             | City                                                     | `New York`                |
| `employer_house_number`     | Street/building number                                   | `456`                     |
| `nationality`               | Country field (e.g., nationality/residence)              | `United States`           |
| `all_country`               | All countries (e.g., birth country selector)             | `United States`           |
| `host_country`              | Destination country                                      | `France`                  |
| `boolean`                   | true/false toggle                                        | `true`                    |
| `phone`                     | Phone number                                             | `12025550123`             |
| `signature`            | Base64-encoded signature image                           | `data:image/png;base64,...` |

> ℹ️ Other data types may be used. Contact the Cozm team if needed.

---

## 7. ⚠️ Error Handling

### Common Error Codes

| Status Code | Error Message            | Description                                |
|-------------|--------------------------|--------------------------------------------|
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

## 9. 📜 Changelog

- ✅ Implemented core A1 application functionality
- 🔐 Added token-based authentication with versioned headers
- 📄 Enabled dynamic form rendering by country & compliance type
- 🔁 Supported conditional logic for smart fields
- 🧾 Documented metadata, field types, and conditional fields
- ✍️ Supported Base64 digital signatures

---