Shortcuty Signer API

Base URL: https://sign.shortcuty.app

⸻

Sign a Shortcut

Upload a .shortcut file using the file form field.

cURL

curl -fS \
  -F 'file=@My Shortcut.shortcut' \
  https://sign.shortcuty.app/api/v1/sign \
  -o 'My Shortcut.shortcut'

HTTP

POST /api/v1/sign HTTP/1.1
Host: sign.shortcuty.app
Content-Type: multipart/form-data
Content-Disposition: form-data; name="file"; filename="My Shortcut.shortcut"
Content-Type: application/octet-stream
<file contents>

The signed Shortcut is returned directly as the response.

⸻

JSON Response

Add ?response=json to return the signed Shortcut as JSON instead of a file.

cURL

curl -sS \
  -F 'file=@My Shortcut.shortcut' \
  'https://sign.shortcuty.app/api/v1/sign?response=json'

HTTP

POST /api/v1/sign?response=json HTTP/1.1
Host: sign.shortcuty.app
Content-Type: multipart/form-data
Content-Disposition: form-data; name="file"; filename="My Shortcut.shortcut"
Content-Type: application/octet-stream
<file contents>

Response

{
  "success": true,
  "file": {
    "filename": "My Shortcut — Signed.shortcut",
    "content_base64": "..."
  }
}

content_base64 contains the signed .shortcut file encoded as Base64.

⸻

Errors

Errors are returned as JSON.

{
  "success": false,
  "error": {
    "code": "invalid_workflow",
    "message": "The uploaded file could not be signed."
  }
}

* error.code — machine-readable error code
* error.message — human-readable error message
