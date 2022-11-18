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

If you wish to receive real-time notifications whenever one of your customers uploads an exam, you can [register a notification webhook](#managing-api-keys-and-notification-webhooks).

## Webhook requirements

* Accepts a POST request
* Does not require authentication

## Notification payload

Notification payload contains the uploaded exam's UUID, ID of the customer who uploaded the exam, a URL where this exam's data can be retrieved and a timestamp when the user submitted this exam.

### Example:

```json
{
    "exam_uuid": "00000000-0000-0000-0000-000000000000",
    "customer_id": "eYENW6HO4xdM0KlGO2kBXtTbD5XX1LTfdV-vYslapMY",
    "download_url": "https://cloud.clarius.com/api/public/v0/exams/00000000-0000-0000-0000-000000000000/customers/eYENW6HO4xdM0KlGO2kBXtTbD5XX1LTfdV-vYslapMY/data/",
    "submitted_at": "2022-10-22T03:10:47.231755Z",
}
```

# Poll API

[***Requires API key***](#managing-api-keys-and-notification-webhooks)

```
GET https://cloud.clarius.com/api/public/v0/exams/accessible/
```

The poll API returns all accessible exams in the [notification payload format](#notification-payload).

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
            "exam_uuid": "00000000-0000-0000-0000-000000000000",
            "customer_id": "eYENW6HO4xdM0KlGO2kBXtTbD5XX1LTfdV-vYslapMY",
            "download_url": "https://cloud.clarius.com/api/public/v0/exams/00000000-0000-0000-0000-000000000000/customers/eYENW6HO4xdM0KlGO2kBXtTbD5XX1LTfdV-vYslapMY/data/",
            "submitted_at": "2022-10-22T03:10:47.231755Z",
        }
        {
            "exam_uuid": "00000000-0000-0000-0000-000000000001",
            "customer_id": "eYENW6HO4xdM0KlGO2kBXtTbD5XX1LTfdV-vYslapMY",
            "download_url": "https://cloud.clarius.com/api/public/v0/exams/00000000-0000-0000-0000-000000000001/customers/eYENW6HO4xdM0KlGO2kBXtTbD5XX1LTfdV-vYslapMY/data/",
            "submitted_at": "2022-10-22T01:35:22.157454Z"
        }
    ]
}
```


# Download API

[***Requires API key***](#managing-api-keys-and-notification-webhooks)

```
GET https://cloud.clarius.com/api/public/v0/exams/[exam uuid]/customers/[customer id]/data/
```

TBD

### Current response

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

[***Requires API key***](#managing-api-keys-and-notification-webhooks)

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

# Managing API keys and notification webhooks

As there's currently no UI for managing your Cloud API usage, to obtain API keys or register a notification webhook, you have to contact us.
