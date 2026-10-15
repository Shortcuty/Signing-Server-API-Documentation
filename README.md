# Shortcuty V1 Signing API Documentation

Upload an unsigned Apple Shortcut and receive a signed `.shortcut` file.

[Quickstart](#quickstart) · [Endpoints](#endpoints) · [Request reference](#request-reference) · [Responses](#response-fields) · [Troubleshooting](#error-responses)

## Getting Started

**Base URL:** `https://sign.shortcuty.app`

**Authentication:** None.

## Quickstart

Send an unsigned workflow as a multipart file and save the result to a different path:

```sh
curl --fail --show-error --silent \
  --form 'file=@unsigned.shortcut' \
  --output 'signed.shortcut' \
  'https://sign.shortcuty.app/api/v1/sign'
```

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/v1/sign` | [Sign a Shortcut](#sign-a-shortcut) |
| GET | `/api/v1` | [Get capabilities](#get-capabilities) |
| GET | `/healthz` | [Check health](#check-health) |

## Sign a Shortcut

**POST** `/api/v1/sign`

**Body:** An unsigned workflow, using one of the [upload formats](#upload-formats).

**Options:** See [Request options](#request-options).

**Response:** `200`, either a binary download or JSON, selected by the `response` query parameter.

### Download Signed Shortcut

The default is `response=download`. Use the [quickstart](#quickstart) for a multipart upload, or send raw bytes:

```sh
curl --fail --show-error --silent \
  --header 'Content-Type: application/octet-stream' \
  --header 'X-Filename: Example.shortcut' \
  --data-binary '@unsigned.shortcut' \
  --output 'signed.shortcut' \
  'https://sign.shortcuty.app/api/v1/sign'
```

### Return Signed Shortcut as JSON

Add `?response=json` to the same endpoint:

```sh
curl --fail --show-error --silent \
  --form 'file=@unsigned.shortcut' \
  'https://sign.shortcuty.app/api/v1/sign?response=json'
```

```json
{
  "success": true,
  "file": {
    "filename": "Example — Signed.shortcut",
    "content_base64": "<base64-encoded signed bytes>",
    "content_type": "application/octet-stream",
    "size": 21424,
    "format": "AEA1",
    "signing_mode": "anyone",
    "input_sha256": "<SHA-256 of uploaded bytes>",
    "signed_sha256": "<SHA-256 of decoded signed bytes>"
  }
}
```

Decode `file.content_base64` to obtain the signed file.

### Apple Shortcuts setup

In **Get Contents of URL**:

1. Set **URL → `https://sign.shortcuty.app/api/v1/sign`**.
2. Set **Method → POST** and **Request Body → Form**.
3. Add a **File** field named `file`, using the unsigned shortcut file as its value.
4. Save the returned file with **Save File**.

## Request reference

### Upload formats

| Format | Content-Type | Required input |
|---|---|---|
| Multipart | `multipart/form-data` | A file part named `file` |
| Raw bytes | `application/octet-stream` | The workflow bytes as the entire request body |

Let the client generate the multipart `Content-Type` header and boundary. The input must be an unsigned XML or binary Apple property list with a `WFWorkflowActions` array.

### Request options

| Name | Location | Default | Meaning |
|---|---|---|---|
| `response` | Query | `download` | `download` for binary output, or `json` for Base64 and metadata |
| `mode` | Multipart text field or query | `anyone` | Only `anyone` is supported. A nonempty Form value takes precedence over the query. |
| `filename` | Multipart text field | `workflow.shortcut` | Fallback name only when the uploaded file has no filename |
| `X-Filename` | Header for raw uploads | `workflow.shortcut` | Input filename used to construct the suggested download name |

A normal multipart upload uses the filename attached to the `file` part. To set it explicitly with curl, use `--form 'file=@unsigned.shortcut;filename=Example.shortcut'`.

### Download filenames

The suggested output name is `<name> — Signed.shortcut`; for example, `Example.shortcut` becomes `Example — Signed.shortcut`.

The service removes path components, slashes, and nonprintable characters, adds the `.shortcut` extension when needed, and normalizes common trailing signed/unsigned suffixes. Use `Content-Disposition` for binary responses or `file.filename` for JSON. For Unicode names, HTTP clients should support the `filename*` parameter.

## Response Fields

Both successful formats set `Cache-Control: no-store`. The signed bytes begin with the `AEA1` container marker.

### Binary download

| Header | Value |
|---|---|
| `Content-Type` | `application/octet-stream` |
| `Content-Disposition` | Attachment with the suggested download filename |
| `X-Shortcut-Signed-Format` | `AEA1` |
| `X-Shortcut-Signing-Mode` | `anyone` |
| `X-Shortcut-Input-SHA256` | SHA-256 of the uploaded bytes |
| `X-Shortcut-Signed-SHA256` | SHA-256 of the returned signed bytes |

### File Object

JSON success responses contain `success: true` and this `file` object:

| Field | Type | Meaning |
|---|---|---|
| `filename` | String | Suggested download name |
| `content_base64` | String | Base64-encoded signed bytes |
| `content_type` | String | `application/octet-stream` |
| `size` | Integer | Signed file size in bytes |
| `format` | String | `AEA1` |
| `signing_mode` | String | `anyone` |
| `input_sha256` | String | SHA-256 of the uploaded bytes |
| `signed_sha256` | String | SHA-256 of the decoded signed bytes |

## Get capabilities

**GET** `/api/v1`

**Response:** `200`, describing the endpoint, signing mode, and configured queue capacity.

```json
{
  "service": "shortcuty-signing-api",
  "version": "1",
  "sign_endpoint": "/api/v1/sign",
  "methods": ["multipart/form-data"],
  "file_field": "file",
  "signing_modes": ["anyone"],
  "default_signing_mode": "anyone",
  "max_concurrent_signings": 2,
  "max_queued_signings": 32,
  "returns": "AEA1 signed .shortcut download"
}
```

Capacity values are deployment settings; read the endpoint for their current values.

## Check health

**GET** `/healthz`

**Response:** `200` when the web service is responding.

```json
{"status":"ok","service":"shortcuty-signing-api"}
```

## Limits and timing

| Topic | Behavior |
|---|---|
| Upload size | Upload limits depend on hosting and proxy configuration. |
| Concurrency and queue | Configured capacity is published by [Get capabilities](#get-capabilities). A full queue returns `503`. |
| Rate limiting | Excess requests can return `429`. |
| Timeouts | Allow time for upload, queueing, and signing. The signing command has a separate timeout; exceeding it returns `504`. |

## Error Responses

Application signing errors use this JSON shape:

```json
{
  "success": false,
  "error": {
    "code": "invalid_workflow",
    "message": "The uploaded file is not a valid Apple property-list workflow."
  }
}
```

Check the HTTP status and `Content-Type` before decoding a response. Rate-limit and proxy errors may have a different body format.

| Status | Code | Meaning / action |
|---|---|---|
| `400` | `missing_file` | No workflow bytes received. Check the upload field or raw body. |
| `400` | `invalid_response_format` | Use `download` or `json`. |
| `400` | `invalid_signing_mode` | Use the supported mode from [Request options](#request-options). |
| `422` | `invalid_workflow` | The content is not a valid unsigned workflow. |
| `422` | `already_signed` | The input already starts with `AEA1`; upload an unsigned workflow. |
| `429` | Varies | Rate limit reached. |
| `500` | `internal_error` | Unexpected server error. |
| `502` | `signing_failed` | Apple's signer failed or produced no signed file. |
| `502` | `invalid_signed_output` | The signer produced an unexpected output format. |
| `503` | `feature_unavailable` | New signing jobs are temporarily disabled. |
| `503` | `signer_unavailable` | The signing command cannot be started. |
| `503` | `signing_queue_full` | Queue admission failed; the response includes `Retry-After: 1`. |
| `504` | `signer_timeout` | The signing command exceeded its timeout. |

### Troubleshooting

| Problem | Check |
|---|---|
| The download has an unexpected local name | `--output` controls the local path; the server's suggestion is in the [filename metadata](#download-filenames). |
| The connection timed out | Signing may have started; see [Limits and timing](#limits-and-timing). |

Correct invalid-input `400` or `422` responses before retrying. For transient failures, respect `Retry-After` when supplied and keep retries bounded.
