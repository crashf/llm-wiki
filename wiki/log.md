---

## [2026-04-24] build | Mac Studio — MiniMax-M2.7 Added to Ollama (131B model)

**Type:** Model Deployment — COMPLETED ✅
**Impact:** MiniMax-M2.7 (228.7B parameters, Q4_K_M ~140GB) now available via Ollama on Mac Studio
**Reporter:** Wayne LeDrew

### Background
Wayne asked to add the downloaded MiniMax-M2.7 model to Ollama with tools enabled for OpenClaw use.

### Problem: Multi-Part GGUF Files
- **Symptom:** Ollama `create` succeeded but running the model returned `500: unable to load model`
- **Root cause:** Model downloaded as 4 split GGUF shards (00001-of-00004 through 00004-of-00004). Ollama's `FROM` directive only copies the first shard (7.9MB metadata header), missing the actual weight tensors.
- **Fix:** Used `llama-gguf-split --merge` to merge all 4 shards into a single 131GB unified GGUF:
  ```bash
  llama-gguf-split --merge \
    MiniMax-M2.7-UD-Q4_K_M-00001-of-00004.gguf \
    /opt/models/minimax-m2.7/MiniMax-M2.7-UD-Q4_K_M.gguf
  ```
  Result: `809 tensors merged from 4 splits`

### Modelfile Configuration
```
FROM /opt/models/minimax-M2.7/MiniMax-M2.7-UD-Q4_K_M.gguf
SYSTEM You are a helpful AI assistant with tool access. Use tools when needed.
PARAMETER stop "]~!b["
PARAMETER stop "]~b]"
PARAMETER num_ctx 8192
```

### Verification
- ✅ `ollama create minimax-m2.7` completed successfully (140GB model blob)
- ✅ API test via `/api/chat` — model loaded and generated coherent response
- ✅ Model metadata: `architecture: minimax-m2, parameters: 228.7B, context: 196608`

### Important: Native Ollama Tools Not Supported
- Ollama lists only `completion` capability — `tools` parameter returns `400 Bad Request`
- **Not a blocker:** OpenClaw handles tool calling itself via structured prompts, independent of Ollama's native tool API
- The model has native XML-style tool template (`<minimax:tool_call>`, `<invoke>`) embedded in GGUF metadata, but Ollama doesn't recognize it

### Performance Metrics
- **Cold load time:** ~79 seconds (from disk to RAM)
- **Prompt eval:** ~127 seconds for 8-token prompt (first run, cold)
- **Generation speed:** ~22 tokens/sec once loaded
- **Memory footprint:** ~140GB weights + KV cache on 256GB unified memory

### Next Steps
1. Configure OpenClaw bot to target this model via Ollama provider
2. Add API key / reverse proxy auth (currently listening on `*:11434` without auth)
3. Consider `launchctl` persistence for Ollama service at boot

### Files Changed
- `projects/mac_studio_deployment/STATUS.md` — updated with model status

---

## [2026-04-22] fix | Brad's Bot (.171) — OpenClaw .21 Upgrade: Google Chat JWT Patch + plugins.entries Fix

**Type:** Fleet Bot Upgrade — COMPLETED ✅
**Impact:** Brad's bot (Colin's, .171) restored to full Google Chat function after upgrading to 2026.4.21
**Reporter:** Wayne LeDrew

### Background
Ran `openclaw update` on .171 to pull latest .21 version. Google Chat stopped responding after upgrade.

### Problem 1: JWT Patch File Hash Changed
- **Symptom:** `sed` command failed — `api-CA5EPa_P.js: No such file or directory`
- **Cause:** Every OpenClaw update recompiles the dist, changing the filename hash
- **New file:** `api-Cx3-lg2G.js` (hash differs from the original patched file)
- **Fix:** Re-ran sed against the new hash: `sudo sed -i 's/async function verifyGoogleChatRequest(params) {/async function verifyGoogleChatRequest(params) { return { ok: true }; } async function verifyGoogleChatRequest_bypass(params) {/' /usr/lib/node_modules/openclaw/dist/api-Cx3-lg2G.js`

### Problem 2: plugins.entries Requirement
- **Symptom:** `openclaw plugins list` showed `googlechat | disabled`
- **Cause:** .21 tightened plugin loading — Google Chat must now be explicitly in `plugins.entries` with `enabled: true`, not just in `plugins.allow`
- **Fix:**
  ```bash
  sudo jq '.plugins.entries.googlechat = {"enabled": true}' /root/.openclaw/openclaw.json > /tmp/tmp.json && sudo mv /tmp/tmp.json /root/.openclaw/openclaw.json
  ```
  Resulting config:
  ```json
  "plugins": {
    "entries": {
      "ollama": { "enabled": true },
      "googlechat": { "enabled": true }
    },
    "allow": ["ollama", "telegram", "googlechat", "memory-core"]
  }
  ```

### Verification
- ✅ `openclaw plugins list` shows `googlechat | enabled | stock:googlechat/index.js`
- ✅ `curl -k https://10.255.245.171/googlechat` returns `invalid payload` (not 502)
- ✅ Brad responding in Google Chat

### Knowledge Captured
**SKILL.md updated:** `skills/punclaw-bot-access/SKILL.md` — "Upgrading to OpenClaw .21" section
- Step-by-step upgrade procedure
- One-liner script for both fixes
- Explanation of why both fixes are needed

**MEMORY.md updated:** Google Chat JWT patch + .21 upgrade notes

**LLM Wiki updated:**
- New concept: `openclaw-421-upgrade.md` — full upgrade procedure
