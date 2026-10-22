# OEM API

[← All Clarius Cloud APIs](README.md)

## Table of Contents

- [OEM API key authentication](#oem-api-key-authentication)
- [Claim Scanners API](#claim-scanners-api)
  - [Payload](#payload)
  - [Response](#response)

## OEM API key authentication

Include your OEM API key in the [Authorization header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization) with the `OEM-API-Key` keyword.

For example:
```
Authorization: OEM-API-Key Ab12Cd34.efGHij56KLmn78OPqr90STuv12WXyz34
```

Each OEM API key belongs to one institution. An admin of the institution creates it in Clarius Cloud under **Settings › Policies › OEM API Keys**, and can revoke it there. A revoked key is refused from then on.

## Claim Scanners API

[***Requires OEM API key***](#oem-api-key-authentication)

Claims scanners into the institution that owns the API key. The institution always comes from the key: the payload names none, and an `inst_id` sent in it is ignored.

A scanner can be claimed once its previous owner has released it, as long as it isn't deactivated or reported lost and it has a Solum license that hasn't expired. Claiming it ends the previous owner's ownership.

### Payload

```bash
POST https://cloud.clarius.com/api/public/v0/devices/solum/claim/
{
    "serial_numbers": ["SERIAL-0001", "SERIAL-0002"]
}
```

- `serial_numbers` (required) - a non-empty list of scanner serial numbers. Surrounding whitespace is ignored, and a serial listed twice is claimed once.

The claim is all or nothing: if any scanner in the list can't be claimed, none of them are.

### Response

```json
[
    {
        "serial": "SERIAL-0001",
        "name": "SERIAL-0001",
        "institution": 123
    },
    {
        "serial": "SERIAL-0002",
        "name": "SERIAL-0002",
        "institution": 123
    }
]
```

One entry per claimed scanner, sorted by serial number: the serial, the scanner's name in the institution (its serial until someone renames it), and the institution's ID.

Status codes

- 201 if success.
- 400 if `serial_numbers` is missing, empty or not a list.
- 401 if the API key is missing, revoked, or not an OEM API key.
- 404 if any scanner can't be claimed. Nothing is claimed.

Every 404 has the same wording, whatever the reason (unknown serial, never released, already claimed, deactivated, lost, or no active Solum license), and lists the serials that couldn't be claimed:

```json
{
    "detail": "No scanner available to claim: SERIAL-0002"
}
```

Each claimed scanner is recorded in the institution's audit log under the API key's name.
