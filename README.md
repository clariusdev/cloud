Cloud APIs
===============

# User Setup

Each member of an institution is assigned a token (customer ID) that uniquely identifies a user of your service.
An "institution" can contain one or more users and own one or more ultrasound scanners.

## Requirements

1. Provide us with a URL where we can pass the user token. The URL should allow the user to log in so you can associate the token with their account. For example:  
```
GET https://example.com?token=user_token_goes_here
```
2. Once the user is logged in and the token is successfully stored, notify us by calling the [Customer Confirmation API](#customer-confirmation-api).

## User workflow

Once a subscription is enabled on the Clarius Cloud, institution users pass their token to your service as follows:

1. Login to Clarius Cloud
1. Go to **Profile**
1. Choose **Services** from the menu
1. Find their institution's service subscription and click **Connect**
1. Proceed to the service login page
1. Log into their service account

# Notifications

If you wish to receive real-time notifications whenever one of your customers uploads an exam, you can [register a notification webhook](#api-key-authentication).

## Webhook requirements

* Accepts a POST request
* Does not require authentication

## Notification payload

Notification payload contains the uploaded exam's UUID, ID of the customer who uploaded the exam, a URL where this exam's data can be retrieved, a UUID of this submission (request) and a timestamp when the user submitted this exam.

### Example:

```json
{
    "request_uuid": "cf1b649c-428a-4cc2-b726-066f8a8c31a7",
    "exam_uuid": "418c1f0f-4dd5-4c65-ae68-1fe8eccf46e5",
    "customer_id": "eYENW6HO4xdM0KlGO2kBXtTbD5XX1LTfdV-vYslapMY",
    "download_url": "https://cloud.clarius.com/api/public/v0/download-requests/cf1b649c-428a-4cc2-b726-066f8a8c31a7/data/",
    "submitted_at": "2022-10-22T03:10:47.231755Z",
}
```

# Poll API

[***Requires API key***](#api-key-authentication)

```
GET https://cloud.clarius.com/api/public/v0/exams/accessible/
```

The poll API returns all accessible exam downloads in the [notification payload format](#notification-payload).

### Filters

Results can be filtered by query parameters:

* `customer_id` exact match

### Pagination

#### Parameters

* `limit` (default: 100) - maximum number of results per page
* `offset` - pagination offset

#### Response

* "count" - total number of items in the response
* "next" - URL to the next page or `null` if this is the last page
* "previous" - URL to the previous page or `null` if this is the first page
* "results" - paginated data

### Example response

```json
{
    "count": 2,
    "next": null,
    "previous": null,
    "results": [
        {
            "request_uuid": "5ac5e0e3-92bd-4b55-8250-c03296a12872",
            "exam_uuid": "440648a3-435f-4053-942b-17509c0787c1",
            "customer_id": "eYENW6HO4xdM0KlGO2kBXtTbD5XX1LTfdV-vYslapMY",
            "download_url": "https://cloud.clarius.com/api/public/v0/exams/download-requests/5ac5e0e3-92bd-4b55-8250-c03296a12872/data/",
            "submitted_at": "2022-10-22T03:10:47.231755Z",
        }
        {
            "request_uuid": "8cc97ac4-ad8d-4fed-a70e-967703ebe521",
            "exam_uuid": "440648a3-435f-4053-942b-17509c0787c1",
            "customer_id": "eYENW6HO4xdM0KlGO2kBXtTbD5XX1LTfdV-vYslapMY",
            "download_url": "https://cloud.clarius.com/api/public/v0/exams/download-requests/8cc97ac4-ad8d-4fed-a70e-967703ebe521/data/",
            "submitted_at": "2022-10-22T01:35:22.157454Z"
        }
    ]
}
```


# Download API

[***Requires API key***](#api-key-authentication)

```
GET https://cloud.clarius.com/api/public/v0/exams/download-requests/[request uuid]/data/
```

TBD

### Current response

* "request_uuid" - UUID of the customer's request
* "patient" - patient data, possible fields:
  * "identifier"
  * "date_of_birth"
  * "gender"
  * "first_name" (human patient only)
  * "middle_name" (human patient only)
  * "last_name" (human patient only)
  * "animal_name" (animal patient only)
  * "animal_type" (animal patient only)
  * "owner_name" (animal patient only)
* "captures" - a list of ultrasound captures with each item containing a "url" field where the capture media (image or video) can be downloaded.

```json
{
    "request_uuid": "f037679c-6ed2-46cd-b2dc-f4abe52e5b52",
    "patient": {
        "identifier": "12345",
        "date_of_birth": "1970-01-01"
    },
    "captures": [
        {
            "url": "..."
        }
    ]
}
```


# Customer Confirmation API

[***Requires API key***](#api-key-authentication)

```
POST https://cloud.clarius.com/api/public/v0/services/customer-confirmations/
{
    "token": "my customer token"
}
```
Used for notifying us that you've successfully associated our token with an account.

### Example Payload
```json
{
    "token": "eYENW6HO4xdM0KlGO2kBXtTbD5XX1LTfdV-vYslapMY"
}
```

### Response
Status codes
* 200 if successfully confirmed integration.
* 404 if a customer with this token does not exist.

# Active Tokens API

[***Requires API key***](#api-key-authentication)

```
GET https://cloud.clarius.com/api/public/v0/services/confirmed-tokens/
```
Returns a paginated list of tokens for all confirmed users.

### Pagination

#### Parameters

* `limit` (default: 10000) - maximum number of results per page
* `offset` - pagination offset

#### Response

* "count" - total number of items in the response
* "next" - URL to the next page or `null` if this is the last page
* "previous" - URL to the previous page or `null` if this is the first page
* "results" - paginated data

### Example response

```json
{
    "count": 2,
    "next": null,
    "previous": null,
    "results": [
        "eYENW6HO4xdM0KlGO2kBXtTbD5XX1LTfdV-vYslapMY",
        "GEMoSRSC5nQTjicaQHE2L_0_HTaPwyjsIs86R6khqHc"
    ]
}
```

# Report Upload API

[***Requires API key***](#api-key-authentication)

```
PUT https://cloud.clarius.com/api/public/v0/services/requests/[request uuid]/report/
```
Optionally, upload a report file associated with a user request. The report will be viewable in the "Services" modal on the exam page.

#### Parameters

* `file` - the file to upload (note: your Content-Type should be `multipart/form-data`)

# API key authentication

Include your API key in the [Authorization header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization) with the `Service-API-Key` keyword.

For example:
```
Authorization: Service-API-Key Q4nmtzH5.mvIzWOlklcrFJ3iWaJTFESYCzCe8j6TQ
```

# Managing API keys and notification webhooks

As there's currently no UI for managing your Cloud API usage, to obtain API keys or register a notification webhook, you have to contact us.
