# Research API

[← All Clarius Cloud APIs](README.md)

> [!NOTE]
> The Raw Data and DICOM APIs only work for exams acquired with a probe that has the Research Package licensed and is owned by the exam's institution. Your account must belong to that institution, and the DICOM API also requires permission to view patient information.

## Table of Contents

- [Token Based Authentication](#token-based-authentication)
- [Auth Token API](#auth-token-api)
  - [Payload](#payload)
  - [Response](#response)
- [Raw Data API](#raw-data-api)
  - [Payload](#payload-1)
  - [Response](#response-1)
- [DICOM API](#dicom-api)
  - [Parameters](#parameters)
  - [Payload](#payload-2)
  - [Response](#response-2)

## Token Based Authentication
Include your token in the [Authorization header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization) with the Token keyword.

For example:
```
Authorization: Token e11f11a11111d11bdca7e49e517534257d9da8801e539885eaacc1111111cdc3

```

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
