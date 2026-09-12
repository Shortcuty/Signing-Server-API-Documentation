```markdown
# Shortcuty Signer API Documentation

## Overview

Welcome to the Shortcuty Signer API documentation.

This API allows developers to upload an Apple Shortcut file and receive a signed version in response.

### What You Can Do

With the Signer API, you can:

- Upload an unsigned Shortcut file
- Download the signed Shortcut directly
- Request the signed Shortcut as JSON for use in an application

### Getting Started

**Base URL:** [https://sign.shortcuty.app](https://sign.shortcuty.app)

---

## Sign a Shortcut

**POST** `/api/v1/sign`

Uploads an unsigned `.shortcut` file and returns the signed Shortcut.

### Request

The file must be uploaded using the `file` form field.

**Content-Type:** `multipart/form-data`

### cURL

```sh
curl -fS \
  -F 'file=@My Shortcut.shortcut' \
  https://sign.shortcuty.app/api/v1/sign \
  -o 'My Shortcut — Signed.shortcut'
```

### HTTP

```http
POST /api/v1/sign HTTP/1.1
Host: sign.shortcuty.app
Content-Type: multipart/form-data

Content-Disposition: form-data; name="file"; filename="My Shortcut.shortcut"
Content-Type: application/octet-stream

<file contents>
```

### Response

The signed Shortcut is returned directly as a file download.

---

## Return a JSON Response

**POST** `/api/v1/sign?response=json`

Returns the signed Shortcut as JSON instead of a file download.

### cURL

```sh
curl -sS \
  -F 'file=@My Shortcut.shortcut' \
  'https://sign.shortcuty.app/api/v1/sign?response=json'
```

### Response

```json
{
  "success": true,
  "file": {
    "filename": "My Shortcut — Signed.shortcut",
    "content_base64": "..."
  }
}
```

`content_base64` contains the signed Shortcut encoded as Base64.

---

## Response Fields

### File Object

```json
{
  "filename": "string",
  "content_base64": "string"
}
```

| Field | Type | Description |
|---|---|---|
| `filename` | string | Name of the signed Shortcut file |
| `content_base64` | string | Signed Shortcut encoded as Base64 |

---

## Error Responses

### 400 Bad Request

The request is missing a required file.

```json
{
  "success": false,
  "error": {
    "code": "missing_file",
    "message": "Send an unsigned workflow in the 'file' multipart field or as a raw request body."
  }
}
```

### 422 Unprocessable Entity

The uploaded file cannot be signed.

```json
{
  "success": false,
  "error": {
    "code": "invalid_workflow",
    "message": "The uploaded file could not be signed."
  }
}
```

### 503 Service Unavailable

The signing service is temporarily unavailable.

```json
{
  "success": false,
  "error": {
    "code": "signer_unavailable",
    "message": "The signing service is temporarily unavailable."
  }
}
```
```
