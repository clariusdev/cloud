# Clarius Cloud APIs

Clarius Cloud offers three public REST APIs, each for a different kind of integration and each with its own authentication.

| API | For | Authentication |
| --- | --- | --- |
| [Service API](service-api.md) | Services that receive exams from Clarius Cloud users: webhook notifications, polling and downloading exams, uploading reports, and managing API keys and application customizations | Service API key |
| [Research API](research-api.md) | Researchers with the Research Package: raw data and DICOM for their exams | Token from the Auth Token API |
| [OEM API](oem-api.md) | OEM partners: claiming scanners into their institution | OEM API key |

All endpoints are under:

```
https://cloud.clarius.com/api/public/v0/
```
