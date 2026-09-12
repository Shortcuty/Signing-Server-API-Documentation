```markdown
# Shortcuty Signer API

Sign an Apple Shortcut with one simple request.

## API URL

```text
https://sign.shortcuty.app
```

## Sign a Shortcut

Upload the original Shortcut file:

```sh
curl -fS \
  -F 'file=@My Shortcut.shortcut' \
  https://sign.shortcuty.app/api/v1/sign \
  -o 'My Shortcut — Signed.shortcut'
```

The uploaded file is `My Shortcut.shortcut`. The signed file is returned separately as `My Shortcut — Signed.shortcut`.

## JSON response

```sh
curl -sS \
  -F 'file=@My Shortcut.shortcut' \
  'https://sign.shortcuty.app/api/v1/sign?response=json'
```

```json
{
  "success": true,
  "file": {
    "filename": "My Shortcut — Signed.shortcut",
    "content_base64": "..."
  }
}
```

## Errors

```json
{
  "success": false,
  "error": {
    "code": "invalid_workflow",
    "message": "The uploaded file could not be signed."
  }
}
```
```
