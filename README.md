# Shortcuty V1 Signing API Documentation

## Overview

Welcome to the Shortcuty V1 Signing API documentation. This API allows
developers to upload an unsigned Apple Shortcut and receive a signed version.

### What You Can Do

With the Signing API, you can:

- **Upload an unsigned Shortcut** using a multipart form request
- **Download the signed Shortcut** directly as a `.shortcut` file
- **Request the signed Shortcut as JSON** with Base64-encoded content
- **Verify the signed output** using the `AEA1` file marker

### Signing flow

Shortcuts follow a simple signing flow:

1. **Upload** → Send an unsigned Shortcut in the `file` form field
2. **Validate** → The service checks that the upload is an Apple workflow
3. **Sign** → The macOS Shortcut signer creates the signed file
4. **Return** → The API returns the signed file or a JSON representation

### Getting Started

**Base URL:** `https://sign.shortcuty.app`

No API key is required. All signing requests must use `multipart/form-data` and
must include a file part named `file`.

## Sign a Shortcut

### Download Signed Shortcut

**POST** `/api/v1/sign`

Signs an unsigned Apple Shortcut and returns the signed file as a download.

**Content-Type:** `multipart/form-data`

**Form Data:**

- `file` (required): An unsigned `.shortcut` workflow containing
  `WFWorkflowActions`

**Request Example:**

```sh
curl --fail --show-error --silent \
  -F 'file=@My Shortcut.shortcut' \
  'https://sign.shortcuty.app/api/v1/sign' \
  -o 'My Shortcut — Signed.shortcut'
```

**Response (200 OK):**

Returns the signed `.shortcut` file as a download. The file begins with the
`AEA1` marker.

---

### Return Signed Shortcut as JSON

**POST** `/api/v1/sign?response=json`

Signs an unsigned Apple Shortcut and returns the signed file as JSON.

**Content-Type:** `multipart/form-data`

**Form Data:**

- `file` (required): An unsigned `.shortcut` workflow containing
  `WFWorkflowActions`

**Request Example:**

```sh
curl --fail --show-error --silent \
  -F 'file=@My Shortcut.shortcut' \
  'https://sign.shortcuty.app/api/v1/sign?response=json'
```

**Response (200 OK):**

```json
{
  "success": true,
  "file": {
    "filename": "My Shortcut — Signed.shortcut",
    "content_base64": "<Base64-encoded signed Shortcut>",
    "size": 21424,
    "format": "AEA1",
    "signing_mode": "anyone",
    "input_sha256": "<sha256 of uploaded bytes>",
    "signed_sha256": "<sha256 of decoded content>"
  }
}
```

Decode `content_base64` to obtain the signed `.shortcut` file. Confirm that the
decoded bytes begin with `AEA1` before saving or sharing the file.

## Response Fields

### File Object

```json
{
  "filename": "string",
  "content_base64": "string",
  "content_type": "string",
  "size": "integer",
  "format": "AEA1",
  "signing_mode": "anyone",
  "input_sha256": "string",
  "signed_sha256": "string"
}
```

| Field | Type | Description |
|---|---|---|
| `filename` | string | Name of the signed Shortcut file |
| `content_base64` | string | Signed Shortcut encoded as Base64 |
| `size` | integer | Size of the signed file in bytes |
| `format` | string | Signed file format, currently `AEA1` |
| `signing_mode` | string | Signing mode, currently `anyone` |
| `input_sha256` | string | SHA-256 hash of the uploaded file |
| `signed_sha256` | string | SHA-256 hash of the signed file |

## Error Responses

Errors use this format:

```json
{
  "success": false,
  "error": {
    "code": "error_code",
    "message": "Human-readable explanation"
  }
}
```

**400 Bad Request:**

- `missing_file` — The multipart request does not contain `file`.
- `invalid_response_format` — `response` must be `download` or `json`.

**422 Unprocessable Entity:**

- `invalid_workflow` — The upload is not a valid unsigned Shortcut workflow.
- `already_signed` — The upload is already signed.

**503 Service Unavailable:**

- `signer_unavailable` — The signing service is temporarily unavailable.
- `signing_queue_full` — Retry the request after a short delay.
