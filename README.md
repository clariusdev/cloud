Cloud APIs
===============

# User Setup

Each institution is assigned a token that is used to uniquely identify a subscription.
An "institution" can contain one or more users and own one or more ultrasound scanners.

Once a subscription is enabled on the Clarius Cloud, institution users can obtain the token as follows:

1. Login to Clarius Cloud with an Administrator account
1. Go to **Institution Settings**
1. Choose **Service Subscriptions** from the menu
1. Copy the token associated with your service

# Notifications

If you wish to receive real-time notifications whenever one of your customers uploads an exam, you can register a notification webhook.

## Webhook requirements

* Accepts a POST request
* Does not require authentication

## Notification payload

Notification payload contains the uploaded exam's UUID, ID of the customer who uploaded the exam and a URL where this exam's data can be retrieved.

### Example:

```json
{
    "exam_uuid": "00000000-0000-0000-0000-000000000000",
    "customer_id": "eYENW6HO4xdM0KlGO2kBXtTbD5XX1LTfdV-vYslapMY",
    "download_url": "https://cloud.clarius.com/api/public/v0/exams/00000000-0000-0000-0000-000000000000/data/",
}
```

# Poll API

***Requires API key***

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
            "download_url": "https://cloud.clarius.com/api/public/v0/exams/00000000-0000-0000-0000-000000000000/data/",
        }
        {
            "exam_uuid": "00000000-0000-0000-0000-000000000001",
            "customer_id": "eYENW6HO4xdM0KlGO2kBXtTbD5XX1LTfdV-vYslapMY",
            "download_url": "https://cloud.clarius.com/api/public/v0/exams/00000000-0000-0000-0000-000000000001/data/",
        }
    ]
}
```


# Download API

***Requires API key***

```
GET https://cloud.clarius.com/api/public/v0/exams/[exam uuid]/data/
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
