# 🧾 Cozm Assessments API Documentation

## 1. Introduction

The **Cozm Assessments API** provides compliance assessments for international business travel and work assignments. It evaluates requirements for:
- Business Visas
- Social Security
- Posted Worker Notifications
- Income Tax
- Corporate Tax

---

## 2. Authentication

The API requires a **Bearer token** and a custom **Version header**.

### Request Headers

```http
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json
Version: YOUR_VERSION
```

---

## 3. API Endpoints

### a. 📋 List User Trips

`GET /api/assessments/user-trips/`

Retrieves a paginated list of user trips with their compliance assessments.

#### Request Headers

| Header          | Value                      | Required | Description                |
|-----------------|----------------------------|----------|----------------------------|
| `Version`       | `YOUR_VERSION`             | Yes      | API version identifier     |
| `Authorization` | `Bearer YOUR_ACCESS_TOKEN` | Yes      | Authentication token       |

#### Example Request (cURL)

```bash
curl --location '<base_url>/api/assessments/user-trips/' \
     --header 'Authorization: Bearer YOUR_ACCESS_TOKEN' \
     --header 'Version: **redacted**'
```

#### Response Format

The response includes pagination information and an array of trip objects:

| Field         | Type    | Description                                    |
|---------------|---------|------------------------------------------------|
| `page_number` | integer | Current page number                            |
| `page_size`   | integer | Number of items per page                       |
| `page_count`  | integer | Total number of pages                          |
| `count`       | integer | Total number of items                          |
| `next`        | string  | URL for next page (null if last page)          |
| `previous`    | string  | URL for previous page (null if first page)     |
| `results`     | array   | List of trip objects                           |

Each trip object in the `results` array contains:

| Field                     | Type    | Description                                    |
|---------------------------|---------|------------------------------------------------|
| `uuid`                    | string  | Unique identifier for the trip                 |
| `home_country`            | string  | Country code of origin                         |
| `host_country`            | string  | Destination country code                       |
| `passport_country`        | string  | Country code of passport                       |
| `nationality`             | string  | Country code of nationality                    |
| `residence_country`       | string  | Country code of residence                      |
| `birth_country`           | string  | Country code of birth                          |
| `days`                    | integer | Duration of stay in days                       |
| `travelling_from`         | string  | Trip start date (YYYY-MM-DD)                   |
| `travelling_to`           | string  | Trip end date (YYYY-MM-DD)                     |
| `travel_type`             | string  | Type of travel (e.g., "Corporate Travel")      |
| `min_monthly_salary_eur`  | string  | Minimum monthly salary in EUR                  |
| `is_same_company_activity`| boolean | Whether activity is for same company           |
| `event_start_date`        | string  | Event start date (YYYY-MM-DD)                  |
| `event_end_date`          | string  | Event end date (YYYY-MM-DD)                    |
| `assessment`              | object  | Compliance assessment results                  |
| `trip_documents`          | array   | List of associated documents                   |

The `assessment` object contains compliance evaluations for:

| Field                          | Description                                           |
|--------------------------------|-------------------------------------------------------|
| `business_visa`                | Business visa requirements and regulations            |
| `social_security`              | Social security compliance requirements               |
| `posted_worker_notification`   | Posted worker notification requirements               |
| `income_tax`                   | Income tax implications                               |
| `corporate_tax`                | Corporate tax implications                            |

Each assessment field contains:
- `is_required`: "YES" or "NO"
- `messages`: Array of requirement details or notifications
- `regulations`: Object containing specific regulatory requirements (for business_visa)

#### Example Response

```json
{
    "page_number": 1,
    "page_size": 20,
    "page_count": 1,
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "uuid": "84625706-9af7-48a5-8e91-86243b1cd9bc",
            "assessment": {
                "business_visa": {
                    "is_required": "YES",
                    "messages": [
                        {
                            "name": "Travel Document Requirements",
                            "text": "",
                            "regulations": {}
                        },
                        {
                            "name": "Travel Document Validity",
                            "text": "<p>Your travel document must be valid for at least 3 months upon arrival. </p>",
                            "regulations": {}
                        },
                        {
                            "name": "Visa Requirements",
                            "text": "<p>You must have a valid visa issued by Albania.</p>",
                            "regulations": {
                                "E-visa": [
                                    "<p>You can obtain an e-visa before departure at <a href=\"https://e-visa.al/\">https://e-visa.al/</a>. You must have a printed e-visa confirmation which can be verified by the SQsign App by scanning its QR code.</p>"
                                ],
                                "Exemptions Country Specific": [
                                    "<p>You can enter without a visa if are of Albanian ethnicity and you leave Albania on or before 2025-08-12. You can stay for a maximum of 90 days in a period of 180 days. You can apply to extend your stay.</p>"
                                ],
                                "Exemptions for 3rd Country Visa": [
                                    "<p>You can enter with a multiple entry D visa issued by a Schengen Member State, if you have used the visa to enter the Schengen Area at least once for a maximum of 90 days in a period of 180 days.</p>",
                                    "<p>You can enter with a multiple entry visa issued by the United Kingdom or the USA, if you have already used the visa to enter the country of issue at least once for a maximum of 90 days.</p>",
                                    "<p>You can enter with a multiple entry C visa issued by a Schengen Member State, if you used the visa to enter the Schengen Area at least once for a maximum of 90 days.</p>"
                                ]
                            }
                        }
                    ]
                },
                "social_security": {
                    "is_required": "NO",
                    "messages": [
                        "No additional action is required."
                    ]
                },
                "posted_worker_notification": {
                    "is_required": "NO",
                    "messages": [
                        "No additional action is required."
                    ]
                },
                "income_tax": {
                    "is_required": "NO",
                    "messages": [
                        "No additional action is required."
                    ]
                },
                "corporate_tax": {
                    "is_required": "NO",
                    "messages": [
                        "No additional action is required."
                    ]
                }
            },
            "trip_documents": [],
            "home_country": "AF",
            "host_country": "AL",
            "activity": "e72d66e7-b710-46c7-be5a-5587ccb403c6",
            "passport_country": "AF",
            "nationality": "AF",
            "residence_country": "AF",
            "birth_country": "AF",
            "days": 18,
            "created_at": "2025-05-12T08:50:06.095113Z",
            "updated_at": "2025-05-12T08:50:06.095426Z",
            "travelling_from": "2025-05-13",
            "travelling_to": "2025-05-31",
            "project": "",
            "travel_type": "Corporate Travel",
            "min_monthly_salary_eur": "12345.00",
            "is_same_company_activity": true,
            "event_start_date": "2025-05-13",
            "event_end_date": "2025-05-31"
        }
    ]
}
```

---

## 4. ⚠️ Error Handling

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
