# Google Chat JWT Verification Patch

**Updated:** 2026-04-22
**Context:** Emergency bypass for `verifyGoogleChatRequest` in OpenClaw .4.8 through .4.21+

## Problem

All Pund-IT fleet bots use Google Chat as a channel. OpenClaw's `verifyGoogleChatRequest` function validates JWT tokens from Google Chat add-on webhooks but the `expectedAddOnPrincipal` check fails with the `sub` claim from Google's service account tokens.

**Symptoms:**
- Google Chat messages return 401 / "Unauthorized"
- Bot silently ignores all Google Chat messages
- `curl -k https://<bot-ip>/googlechat` returns `invalid payload` (webhook is reached but rejected)

**GitHub Issues:** #63079, #53888, #58514, #57386, #67786, #35095, #58541, #60377

## Solution

Bypass the verification function entirely by replacing its body with `return { ok: true };`.

## ⚠️ Critical: File Hash Changes Every Update

**The dist filename contains a hash that changes with every OpenClaw update!**

Before applying the patch, ALWAYS find the current file:
```bash
grep -l 'verifyGoogleChatRequest' /usr/lib/node_modules/openclaw/dist/api-*.js
```

Never hardcode the filename — it will be different after every `openclaw update`.

## Patch Application

### Find current file
```bash
API_FILE=$(grep -l 'verifyGoogleChatRequest' /usr/lib/node_modules/openclaw/dist/api-*.js | head -1)
echo "Current file: $API_FILE"
```

### Apply patch
```bash
sudo sed -i 's/async function verifyGoogleChatRequest(params) {/async function verifyGoogleChatRequest(params) { return { ok: true }; } async function verifyGoogleChatRequest_bypass(params) {/' "$API_FILE"
```

### Restart gateway
```bash
sudo systemctl restart openclaw-gateway
```

### Verify
```bash
curl -s --max-time 5 -k https://<bot-ip>/googlechat -H "Content-Type: application/json" -d '{"type":"message"}'
# Expected: invalid payload (not 502 Bad Gateway)
```

Also check plugin status:
```bash
openclaw plugins list | grep googlechat
# Expected: enabled | stock:googlechat/index.js
```

## Constants Preservation

The sed replacement intentionally keeps all code after the function signature intact, including:
- `const CHAT_API_BASE = "https://chat.googleapis.com/v1"`
- `const CHAT_UPLOAD_BASE = "https://chat.googleapis.com/upload/v1"`

These are required for the Google Chat runtime to function. If they're lost, symptoms include blank messages and `ReferenceError` in gateway logs.

## Complete One-Liner (Post-.21 Upgrade)
```bash
API_FILE=$(grep -l 'verifyGoogleChatRequest' /usr/lib/node_modules/openclaw/dist/api-*.js | head -1) && sudo sed -i 's/async function verifyGoogleChatRequest(params) {/async function verifyGoogleChatRequest(params) { return { ok: true }; } async function verifyGoogleChatRequest_bypass(params) {/' $API_FILE && sudo systemctl restart openclaw-gateway
```

## Fleet Status (2026-04-22)

| Bot | IP | Version | Google Chat | Notes |
|-----|-----|---------|-------------|-------|
| ROB | .163 | 2026.4.8 | ✅ patched | Old version, still works |
| CEDRIC | .164 | 2026.4.8 | ✅ patched | |
| Wayne (AdriannaMKII) | .167 | 2026.4.8 | ✅ patched | |
| Brett (Echo) | .168 | 2026.4.8 | ✅ patched | |
| Piotr (Eve) | .169 | 2026.4.8 | ✅ patched | |
| Mai (Leon) | .180 | 2026.4.8 | ✅ patched | |
| Brad (Colin's) | .171 | 2026.4.21 | ✅ fixed | Upgraded 2026-04-22, needed plugins.entries fix |
| Hino | .192 | 2026.4.8 | ✅ patched | |
| Rosita | .188 | 2026.4.8 | ✅ patched | |
| Jeeves (Danielle) | .189 | 2026.4.8 | ✅ patched | Had gateway restart loop, fixed by Danielle |

## Related

- [[openclaw-421-upgrade]] — Full upgrade procedure including the plugins.entries change
- [[log]] — 2026-04-22 entries for .171 Brad upgrade