Cloud APIs
===============

# User Setup

Each member of an institution is assigned a token (customer ID) that uniquely identifies a user of your service.
An "institution" can contain one or more users and own one or more ultrasound scanners.

## Requirements

1. Using the [settings update API](#settings-viewupdate-api), provide us with a URL where we can pass the user token. The URL should allow the user to log in so you can associate the token with their account. For example:  
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

If you wish to receive real-time notifications whenever one of your customers uploads an exam, you can register a notification webhook using the [settings update API](#settings-viewupdate-api).

## Webhook requirements

* Accepts a POST request
* Does not require authentication

Upon receiving the notification, make sure to acknowledge it by responding with a 2xx status code. Later, asynchronously retrieve the data by following the download URL.

⚠️ If you process the data synchronosly before responding to the notification, it can take a lot of time and the request might time out on our end. So, even if you've successfully received the notification, from our side it would look like a failure. Currently, we do not implement notification retries but, in the future, we might.

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
POST https://cloud.clarius.com/api/public/v0/services/requests/[request uuid]/reports/
```
Optionally, upload one or more report files associated with a user request. Reports will be viewable in the "Services" modal on the exam page.

#### Parameters

* `name` - user-friendly name describing the report. The name will be shown to the user in the "Services" modal.
* `file` - the file to upload (note: your Content-Type should be `multipart/form-data`)


# Settings View/Update API

[***Requires API key***](#api-key-authentication)

```
GET https://cloud.clarius.com/api/public/v0/services/settings/
```
Returns your settings

```
PATCH https://cloud.clarius.com/api/public/v0/services/settings/
```
Allows updating your settings

### Allowed fields
* "notification_url" - a valid URL or an empty string
* "token_transfer_url" - a valid URL containing the string `{token}` anywhere in the URL, or an empty string

### Example Payload
```json
{
    "token_transfer_url": "https://example.com/{token}/"
}
```

# List active API keys

[***Requires API key***](#api-key-authentication)

```
GET https://cloud.clarius.com/api/public/v0/services/api-keys/
```

### Example response

```json
[
    {
        "prefix": "QUgDR6JB",
        "name": "My key",
        "created": "2022-08-31T23:28:55.587755Z"
    }
]
```

# Create API key

[***Requires API key***](#api-key-authentication)

```
POST https://cloud.clarius.com/api/public/v0/services/api-keys/
{
    "name": "My new api key"
}
```

Creates new API keys. Note: __it is not possible to retrieve the key after it's been created so store it somewhere safe.__

### Example Payload
```json
{
    "name": "My new api key"
}
```

### Example Response

```json
{
    "prefix": "ghhni3x8",
    "name": "My new api key",
    "created": "2023-03-09T01:35:32.639396Z",
    "key": "ghhni3x8.aToALr2FC8A7toD11Cj902BLtTuLSZEr"
}
```

# Revoke API key

[***Requires API key***](#api-key-authentication)

```
DELETE https://cloud.clarius.com/api/public/v0/services/api-keys/[key prefix]/
```

Note: it is not possible to revoke the key you're using to authenticate.

# API key authentication

Include your API key in the [Authorization header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization) with the `Service-API-Key` keyword.

For example:
```
Authorization: Service-API-Key Q4nmtzH5.mvIzWOlklcrFJ3iWaJTFESYCzCe8j6TQ
```

# Managing API keys

Onboarding is currently a manual process. We will provide you with the initial API key.

For rotating API keys, use [Create](#create-api-key) and [Revoke](#revoke-api-key) APIs.
