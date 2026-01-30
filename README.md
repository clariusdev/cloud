Cloud APIs

## Table of Contents

- [Service API](#service-api)
  - [User Setup](#user-setup)
      - [Requirements](#requirements)
      - [User workflow](#user-workflow)
  - [Notifications](#notifications)
      - [Webhook requirements](#webhook-requirements)
      - [Notification payload](#notification-payload)
          - [Example:](#example)
  - [API](#api)
      - [Poll API](#poll-api)
          - [Filters](#filters)
          - [Pagination](#pagination)
          - [Example response](#example-response)
      - [Download API](#download-api)
          - [Response](#response)
      - [Customer Confirmation API](#customer-confirmation-api)
          - [Example Payload](#example-payload)
          - [Response](#response-1)
      - [Active Tokens API](#active-tokens-api)
          - [Pagination](#pagination-1)
          - [Example response](#example-response-1)
      - [Report Upload API](#report-upload-api)
      - [Settings View/Update API](#settings-viewupdate-api)
          - [Allowed fields](#allowed-fields)
          - [Example Payload](#example-payload-1)
      - [Notification Records API](#notification-records-api)
          - [Pagination](#pagination-2)
          - [Example response](#example-response-2)
      - [List Active API keys](#list-active-api-keys)
          - [Example response](#example-response-3)
      - [Create API key](#create-api-key)
          - [Example Payload](#example-payload-2)
          - [Example Response](#example-response-4)
      - [Revoke API key](#revoke-api-key)
      - [View Clarius Cloud Applications](#view-clarius-cloud-applications)
          - [Example response](#example-response-5)
      - [Customize Applications](#customize-applications)
          - [List customized applications](#list-customized-applications)
          - [Retrieve customized application](#retrieve-customized-application)
          - [Create/Update application customizations](#createupdate-application-customizations)
          - [Remove application customizations](#remove-application-customizations)
  - [API key authentication](#api-key-authentication)
  - [Managing API keys](#managing-api-keys)
- [Research API](#research-api)
  - [Auth Token API](#auth-token-api)
    - [Payload](#payload)
    - [Response](#response-2)
  - [Raw Data API](#raw-data-api)
    - [Payload](#payload-1)
    - [Response](#response-3)
  - [DICOM API](#dicom-api)
    - [Parameters](#parameters)
    - [Payload](#payload-2)
    - [Response](#response-4)
  - [Token Based Authentication](#token-based-authentication)

# Service API

## User Setup

Each member of an institution is assigned a token (customer ID) that uniquely identifies a user of your service.
An "institution" can contain one or more users and own one or more ultrasound scanners.

#### Requirements

1. Using the [settings update API](##settings-viewupdate-api), provide us with a URL where we can pass the user token. The URL should allow the user to log in so you can associate the token with their account. For example:
```
GET https://example.com?token=user_token_goes_here
```
2. Once the user is logged in and the token is successfully stored, notify us by calling the [Customer Confirmation API](##customer-confirmation-api).

#### User workflow

Once a subscription is enabled on the Clarius Cloud, institution users pass their token to your service as follows:

1. Login to Clarius Cloud
1. Go to **Profile**
1. Choose **Services** from the menu
1. Find their institution's service subscription and click **Connect**
1. Proceed to the service login page
1. Log into their service account

## Notifications

If you wish to receive real-time notifications whenever one of your customers uploads an exam, you can register a notification webhook using the [settings update API](##settings-viewupdate-api).

To assist with integration and debugging, we log every time we call your webhook. You can view these records at the [Notification Records API](##notification-records-api).

#### Webhook requirements

* Accepts a POST request
* Does not require authentication

Upon receiving the notification, make sure to acknowledge it by responding with a 2xx status code. Later, asynchronously retrieve the data by following the download URL.

⚠️ If you process the data synchronosly before responding to the notification, it can take a lot of time and the request might time out on our end. So, even if you've successfully received the notification, from our side it would look like a failure. Currently, we do not implement notification retries but, in the future, we might.

#### Notification payload

Notification payload contains the uploaded exam's UUID, ID of the customer who uploaded the exam, a URL where this exam's data can be retrieved, a UUID of this submission (request) and a timestamp when the user submitted this exam.

###### Example:

```json
{
    "request_uuid": "cf1b649c-428a-4cc2-b726-066f8a8c31a7",
    "exam_uuid": "418c1f0f-4dd5-4c65-ae68-1fe8eccf46e5",
    "customer_id": "eYENW6HO4xdM0KlGO2kBXtTbD5XX1LTfdV-vYslapMY",
    "download_url": "https://cloud.clarius.com/api/public/v0/download-requests/cf1b649c-428a-4cc2-b726-066f8a8c31a7/data/",
    "submitted_at": "2022-10-22T03:10:47.231755Z",
}
```

## API

#### Poll API

[***Requires API key***](##api-key-authentication)

```
GET https://cloud.clarius.com/api/public/v0/exams/accessible/
```

The poll API returns all accessible exam downloads in the [notification payload format](##notification-payload).

###### Filters

Results can be filtered by query parameters:

* `customer_id` exact match

###### Pagination

######## Parameters

* `limit` (default: 100) - maximum number of results per page
* `offset` - pagination offset

######## Response

* "count" - total number of items in the response
* "next" - URL to the next page or `null` if this is the last page
* "previous" - URL to the previous page or `null` if this is the first page
* "results" - paginated data

###### Example response

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


#### Download API

[***Requires API key***](##api-key-authentication)

```
GET https://cloud.clarius.com/api/public/v0/exams/download-requests/[request uuid]/data/
```

⚠️ Download API is meant to be accessed through "download_url" and not directly. It is not recommended to build the download URL manually as the format may change.

###### Response

Response may vary based on data you need from us and can be reconfigured on request. This is an example of what it may look like:

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


#### Customer Confirmation API

[***Requires API key***](##api-key-authentication)

```
POST https://cloud.clarius.com/api/public/v0/services/customer-confirmations/
{
    "token": "my customer token"
}
```
Used for notifying us that you've successfully associated our token with an account.

###### Example Payload
```json
{
    "token": "eYENW6HO4xdM0KlGO2kBXtTbD5XX1LTfdV-vYslapMY"
}
```

###### Response
Status codes
* 200 if successfully confirmed integration.
* 404 if a customer with this token does not exist.

#### Active Tokens API

[***Requires API key***](##api-key-authentication)

```
GET https://cloud.clarius.com/api/public/v0/services/confirmed-tokens/
```
Returns a paginated list of tokens for all confirmed users.

###### Pagination

######## Parameters

* `limit` (default: 10000) - maximum number of results per page
* `offset` - pagination offset

######## Response

* "count" - total number of items in the response
* "next" - URL to the next page or `null` if this is the last page
* "previous" - URL to the previous page or `null` if this is the first page
* "results" - paginated data

###### Example response

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

#### Report Upload API

[***Requires API key***](##api-key-authentication)

```
POST https://cloud.clarius.com/api/public/v0/services/requests/[request uuid]/reports/
```
Optionally, upload one or more report files associated with a user request. Reports will be viewable in the "Services" modal on the exam page.

######## Parameters

* `name` - user-friendly name describing the report. The name will be shown to the user in the "Services" modal.
* `file` - the file to upload (note: your Content-Type should be `multipart/form-data`)


#### Settings View/Update API

[***Requires API key***](##api-key-authentication)

```
GET https://cloud.clarius.com/api/public/v0/services/settings/
```
Returns your settings

```
PATCH https://cloud.clarius.com/api/public/v0/services/settings/
```
Allows updating your settings

###### Allowed fields
* "notification_url" - a valid URL or an empty string
* "token_transfer_url" - a valid URL containing the string `{token}` anywhere in the URL, or an empty string

###### Example Payload
```json
{
    "token_transfer_url": "https://example.com/{token}/"
}
```

#### Notification Records API

[***Requires API key***](##api-key-authentication)

```
GET https://cloud.clarius.com/api/public/v0/services/notifications/records/
```
Returns a paginated list of notification webhook records.

⚠️ Records are deleted after 14 days.

###### Pagination

######## Parameters

* `limit` (default: 100) - maximum number of results per page
* `offset` - pagination offset

######## Response

* "count" - total number of items in the response
* "next" - URL to the next page or `null` if this is the last page
* "previous" - URL to the previous page or `null` if this is the first page
* "results" - paginated data

###### Example response

```json
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "logged_at": "2023-06-01T02:15:28.331086Z",
            "event": {
                "sent_at": "2023-06-01T02:15:23.435871Z",
                "request": {
                    "url": "https://example.com",
                    "method": "POST",
                    "content": "{}",
                },
                "response": null,
                "error": "timed out"
            }
        }
    ]
}
```

#### List Active API keys

[***Requires API key***](##api-key-authentication)

```
GET https://cloud.clarius.com/api/public/v0/services/api-keys/
```

###### Example response

```json
[
    {
        "prefix": "QUgDR6JB",
        "name": "My key",
        "created": "2022-08-31T23:28:55.587755Z"
    }
]
```

#### Create API key

[***Requires API key***](##api-key-authentication)

```
POST https://cloud.clarius.com/api/public/v0/services/api-keys/
{
    "name": "My new api key"
}
```

Creates new API keys.

⚠️ __It is not possible to retrieve the key after it's been created so store it somewhere safe.__

###### Example Payload
```json
{
    "name": "My new api key"
}
```

###### Example Response

```json
{
    "prefix": "ghhni3x8",
    "name": "My new api key",
    "created": "2023-03-09T01:35:32.639396Z",
    "key": "ghhni3x8.aToALr2FC8A7toD11Cj902BLtTuLSZEr"
}
```

#### Revoke API key

[***Requires API key***](##api-key-authentication)

```
DELETE https://cloud.clarius.com/api/public/v0/services/api-keys/[key prefix]/
```

Note: it is not possible to revoke the key you're using to authenticate.

#### View Clarius Cloud Applications

[***Requires API key***](##api-key-authentication)

```
GET https://cloud.clarius.com/api/public/v0/exams/applications/
```

Retrieve a list of applications we have on Clarius Cloud.

###### Example response

```json
[
    {
        "uuid": "9f9531da-84a7-4149-8d59-c4c86c0afaa3",
        "name": "Abdomen"
    },
    {
        "uuid": "2e79155f-811c-4d29-9b49-807b20013518",
        "name": "Cardiac"
    },
    {
        "uuid": "643395dc-6786-4c90-9012-565fd23ae71a",
        "name": "Bladder"
    }
]
```

#### Customize Applications

A set of APIs to manage customizations of exam applications for your users.

###### List customized applications

[***Requires API key***](##api-key-authentication)

```
GET https://cloud.clarius.com/api/public/v0/services/applications/
```

######## Example response

```json
[
    {
        "uuid": "2e79155f-811c-4d29-9b49-807b20013518",
        "name": "Cardiac",
        "annotations": [
            {
                "title": "Our special annotations",
                "labels": [
                    "LABEL1",
                    "LABEL2"
                ],
                "linked": false
            }
        ]
    }
]
```

###### Retrieve customized application

[***Requires API key***](##api-key-authentication)

```
GET https://cloud.clarius.com/api/public/v0/services/applications/[application uuid]/
```

######## Example response

```json
{
    "uuid": "2e79155f-811c-4d29-9b49-807b20013518",
    "name": "Cardiac",
    "annotations": [
        {
            "title": "Our special annotations",
            "labels": [
                "LABEL1",
                "LABEL2"
            ],
            "linked": false
        }
    ]
}
```

###### Create/Update application customizations

[***Requires API key***](##api-key-authentication)

```
PUT https://cloud.clarius.com/api/public/v0/services/applications/[application uuid]/
```

######## Parameters
* `annotations` - a list of annotations you would like your users to use in the Clarius Ultrasound app. Each item on the list must contain the following fields:
  * `title` - string
  * `labels` - list of strings
  * `linked` - true/false boolean. Labels in linked categories are combined together in a unique label.

######## Example payload

```json
{
    "annotations": [
        {
            "title": "Our special annotations",
            "labels": [
                "LABEL1",
                "LABEL2"
            ],
            "linked": false
        }
    ]
}
```

###### Remove application customizations

[***Requires API key***](##api-key-authentication)

```
DELETE https://cloud.clarius.com/api/public/v0/services/applications/[application uuid]/
```

## API key authentication

Include your API key in the [Authorization header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization) with the `Service-API-Key` keyword.

For example:
```
Authorization: Service-API-Key Q4nmtzH5.mvIzWOlklcrFJ3iWaJTFESYCzCe8j6TQ
```

## Managing API keys

Onboarding is currently a manual process. We will provide you with the initial API key.

For rotating API keys, use [Create](##create-api-key) and [Revoke](##revoke-api-key) APIs.


# Research API

## Auth Token API

### Payload

```bash
POST https://cloud.clarius.com/api/public/v0/token-auth/
{
    "username": "<your username>"
    "password": "<your password>"
}
```

### Response

```json
{
    "token": "e11f11a11111d11bdca7e49e517534257d9da8801e539885eaacc1111111cdc3"
}
```

Status codes

- 200 if success.
- 400 if credentials are invalid.


## Raw Data API

Requires Research Bundle

⚠️ Links are valid for 7 days.

### Payload

```bash
GET https://cloud.clarius.com/api/public/v0/exams/rawdata/1
```

### Response

```python
[
    {
        "capture_uuid": "a821fa45-783f-48af-a497-869f1951a2f1",
        "blob": '{"id": "{a821fa45-783f-48af-a497-869f1951a2f1}", "date and time": "2021-09-15T13:43:00.943-08:00", "exam id": "{46805f86-35f8-4779-a8b9-7faddb658712}", "probe serial": "PROBE-SERIAL", "probe model": "EXAMPLE-MODELNUMBER", "sections": [{"files": [{"filename": "31C7F82685580.F.0.jpg", "hardware id": "15", "timestamp": "875758904432000", "stream id": 0.0}], "streams": [{"stream id": 0.0, "info": {"size": {"samples per line": 1078.0, "number of lines": 192.0, "sample size": "1 bytes"}, "type": "B pre-scan", "compression": "jpeg", "sampling rate": "7.5438596491228074 MHz", "delay samples": 7.5438596491228065, "lines": [{"rx element": 0.0, "tx element": 0.0, "angle": "0 °"}, {"rx element": 191.0, "tx element": 191.0, "angle": "0 °"}], "compound": {"low scale": 0.0}, "roi": {"start line": 0.0, "end line": 191.0, "start sample": 0.0, "end sample": 587.0}}}], "acoustic": {"mi": 1.001, "tib": 0.266, "tis": 0.2350692}}]}',
        "raw_image": "link-to-AWS-to-download-from"
    }
]
```

Status codes

- 200 if success.
- 403 if the user lacks permission to view the exam.
- 404 if no exam with matching id found.
- 400 if the exam is still being processed.

## DICOM API

Requires Research Bundle

⚠️ Links are valid for 7 days.

### Parameters

- `syntax` (optional) - DICOM transfer syntax; default = `jpeg lossless nonheirarchical process 14`; choices = [`jpeg lossless nonheirarchical process 14`, `jpeg ls lossless`]
- `b_image_only` (optional) - a boolean flag to disable all overlays on the image; default = `false`; overrides other parameters controlling overlays.
- `resolution` (optional) - a string representing image resolution in format `<number>x<number>`; default = `1024x768`
- `include_annotations` (optional) - a boolean flag to display annotations on the image; default = `false`
- `include_demographics` (optional) - a boolean flag to display demographics on the image; default = `false`
- `include_measurements` (optional) - a boolean flag to display measurements on the image; default = `false`
- `include_scan_parameters` (optional) - a boolean flag to display scan parameters on the image; default = `false`
- `demographics` (optional) - list of demographics to display on the image if `include_demographics` = `true`; default = `all demographic options`; choices = [`PATIENT_NAME`, `PATIENT_ID`, `PATIENT_GENDER`, `PATIENT_DATE_OF_BIRTH`, `INSTITUTION_NAME`, `INSTITUTION_ADDRESS`, `OPERATOR_ID`, `EXAM_DATE`, `PROBE_MODEL`, `APPLICATION`, `ACOUSTIC_INDICES`]
- `tz` (optional) - timezone for locale aware data formatting; default = UTC

### Payload

```bash
POST https://cloud.clarius.com/api/public/v0/exams/dicom/1
{
    "syntax": "jpeg ls lossless", 
    "b_image_only": false, 
    "resolution": "800x600",
    "include_annotations": true,
    "include_demographics": true,
    "include_measurements": false,
    "include_scan_parameters": false,
    "demographics": ["PATIENT_NAME", "EXAM_DATE"]
    "tz": "America/Toronto"
}
```

### Response

```json
{
    "download_url": "link-to-AWS-to-download-from"
}
```

Status codes

- 200 if success.
- 403 if the user lacks permission to view the exam.
- 404 if no exam with matching id found.
- 400 if parameters are invalid


## Token Based Authentication
Include your token in the [Authorization header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization) with the Token keyword.

For example:
```
Authorization: Token e11f11a11111d11bdca7e49e517534257d9da8801e539885eaacc1111111cdc3

```
