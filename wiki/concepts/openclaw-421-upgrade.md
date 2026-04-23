# OpenClaw .21 Upgrade (2026.4.21)

**Date:** 2026-04-22
**Context:** Upgrading Pund-IT fleet from 2026.4.8 to 2026.4.21 revealed two breaking changes

## Overview

Upgrading fleet bots from .4.8 → .4.21 introduces required fixes post-update:

1. **Google Chat JWT patch** — The dist file hash changes every build, so the patch must be reapplied to the new file
2. **`plugins.entries.googlechat.enabled: true`** — .21 tightened plugin loading; the plugin must now be explicitly enabled in entries
3. **`models.providers.ollama-local.api: "openai-completions"`** — Required if the bot uses lossless-claw with ollama-local provider (pi-ai library only knows OpenAI-compatible API types)

## Breaking Changes

### 1. plugins.entries required

**Before .21 (.4.8 behavior):** A plugin in `plugins.allow` would start without an explicit `plugins.entries` entry.

**After .21 (.4.21 behavior):** Google Chat shows `disabled` unless explicitly in entries:
```json
"plugins": {
  "entries": {
    "ollama": { "enabled": true },
    "googlechat": { "enabled": true }
  },
  "allow": ["ollama", "telegram", "googlechat", "memory-core"]
}
```

### 2. ollama provider api value

**Before .21:** `api: "ollama"` worked for both OpenClaw and lossless-claw.

**After .21:** `lossless-claw` uses pi-ai which only registers OpenAI-compatible API types. `api: "ollama"` causes `"No API provider registered for api: ollama"`.

**Fix:** Change to `api: "openai-completions"` — OpenClaw's Ollama plugin doesn't care about the value.

## Upgrade Procedure

### Step 1: Run update
```bash
sudo openclaw update
```
Wait for daemon restart to complete.

### Step 2: Find new API file (JWT patch)
```bash
API_FILE=$(grep -l 'verifyGoogleChatRequest' /usr/lib/node_modules/openclaw/dist/api-*.js | head -1)
echo "Current file: $API_FILE"
```

### Step 3: Apply JWT bypass
```bash
sudo sed -i 's/async function verifyGoogleChatRequest(params) {/async function verifyGoogleChatRequest(params) { return { ok: true }; } async function verifyGoogleChatRequest_bypass(params) {/' "$API_FILE"
```

### Step 4: Fix plugins.entries
```bash
sudo jq '.plugins.entries.googlechat = {"enabled": true}' /root/.openclaw/openclaw.json > /tmp/tmp.json && sudo mv /tmp/tmp.json /root/.openclaw/openclaw.json
```

Verify result:
```json
"plugins": {
  "entries": {
    "ollama": { "enabled": true },
    "googlechat": { "enabled": true }
  }
}
```

### Step 5: Fix ollama-local api value (if using lossless-claw)
```bash
jq '.models.providers["ollama-local"].api = "openai-completions"' /root/.openclaw/openclaw.json | sudo tee /root/.openclaw/openclaw.json.new && sudo mv /root/.openclaw/openclaw.json.new /root/.openclaw/openclaw.json
```
> Full: [openclaw-421-ollama-provider-fix](./concepts/openclaw-421-ollama-provider-fix.md)

### Step 6: Restart gateway
```bash
sudo systemctl restart openclaw-gateway
```

### Step 7: Verify
```bash
# Check plugin status
openclaw plugins list | grep -E 'googlechat|ollama'

# Check webhook
curl -s --max-time 5 -k https://<bot-ip>/googlechat -H "Content-Type: application/json" -d '{"type":"message"}'
# Expected: invalid payload (not 502)

# Test LCM call
openclaw api call lcm status
```

## Complete One-Liner (steps 2-6)
```bash
API_FILE=$(grep -l 'verifyGoogleChatRequest' /usr/lib/node_modules/openclaw/dist/api-*.js | head -1) && \
  sudo sed -i 's/async function verifyGoogleChatRequest(params) {/async function verifyGoogleChatRequest(params) { return { ok: true }; } async function verifyGoogleChatRequest_bypass(params) {/' $API_FILE && \
  sudo jq '.plugins.entries.googlechat = {"enabled": true}' /root/.openclaw/openclaw.json > /tmp/tmp.json && sudo mv /tmp/tmp.json /root/.openclaw/openclaw.json && \
  jq '.models.providers["ollama-local"].api = "openai-completions"' /root/.openclaw/openclaw.json | sudo tee /root/.openclaw/openclaw.json.new && sudo mv /root/.openclaw/openclaw.json.new /root/.openclaw/openclaw.json && \
  sudo systemctl restart openclaw-gateway
```

## Fleet Status (2026-04-22)

| Bot | IP | Version | Status |
|-----|-----|---------|--------|
| Brad (Colin) | .171 | .4.21 | ✅ Fixed |
| Piotr (Eve) | .169 | .4.21 | ✅ Fixed |
| ROB | .163 | .4.8 | Pending upgrade |
| CEDRIC | .164 | .4.8 | Pending upgrade |
| Wayne | .167 | .4.8 | Pending upgrade |
| Brett | .168 | .4.8 | Pending upgrade |
| Mai | .180 | .4.8 | Pending upgrade |
| Hino | .192 | .4.8 | Pending upgrade |
| Rosita | .188 | .4.8 | Pending upgrade |

## Related

- [google-chat-jwt-verification-patch](./concepts/google-chat-jwt-verification-patch.md)
- [openclaw-421-ollama-provider-fix](./concepts/openclaw-421-ollama-provider-fix.md)
- [log](./log.md)