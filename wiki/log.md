---

## [2026-04-25] ops | Anzen Egress Portal Review + CPE Power-Outage Recovery

**Type:** Operations — COMPLETED ✅
**Reporter:** Adrianna

### Tasks Completed
1. **Anzen Egress Portal Review**
   - Read `PLAN.md`, `TRACKER.md`, `DEPLOYMENT-NOTES.md`
   - Status: Phase 8 complete, deployed at `https://anzenegress.pund-it.ca`
   - Confirmed all 8 build phases done: Foundation → IPAM → ConnectWise → CPE Pre-Stage → Deployment Wizard → Monitoring → Polish → Live Deployment
   - Noted open gaps: deploy button UI intermittency, rollup automation cron missing, detail action stubs, decommission incomplete, admin placeholders

2. **CPE Troubleshooting — Wayne Trailer Power Outage**
   - **Symptom:** Tunnel last handshake ~8:46 AM; no transit reachability from edge after morning power outage
   - **Diagnosis (edge-only, no DC changes):**
     - Edge WG interfaces running, firewall open
     - CPE clock showed 10:04 UTC vs actual ~22:25 UTC (12h drift after power loss)
     - NTP client enabled but **no servers configured**
   - **Root cause:** WG crypto handshake sequence counters failed due to clock drift post-power-loss. NTP wasn't configured with any servers, so the CPE never resynced.
   - **Fix:** Wayne added `0.pool.ntp.org`, `1.pool.ntp.org`, `2.pool.ntp.org` to CPE NTP client; clock corrected; tunnels re-established within minutes.
   - **Verification:**
     - Transit `10.99.0.2`: 0% loss, ~40–75ms RTT from edge
     - Customer IP `170.205.18.225`: 0% loss, ~50–64ms RTT via `anzen` routing-table
     - New WAN IP from Starlink CGNAT rotation (129.222.193.42 → 143.105.15.97) — WG handled seamlessly

### Lessons
- Starlink + CGNAT + WG is resilient to IP changes; clock drift is the real enemy post-outage.
- All CPE configs should include explicit NTP servers and consider `persistent-keepalive` as a health indicator but not a substitute for stable time.
- The `.rsc` bootstrap config should include NTP server assignment (currently does not).

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


## [2026-04-25] investigate | Club Sixty Six Transcode Timeout Fix

**Type:** Investigation & Fix — COMPLETED ✅

**Context:** Olivia reported video uploads stuck in "awaiting transcode" state. 3 videos in `pending` status.

**Root cause:** `TRANSCODE_TIMEOUT_MS` hardcoded to 10 minutes (`600000ms`). 20-25 minute videos on loaded VM (load avg 20) take ~15-20 min for 1080p. Sequential transcoding → 1080p always hits the 10-min wall.

**Fix deployed:**
- `src/lib/transcode.ts` line 16: `10 * 60 * 1000` → `60 * 60 * 1000` (1 hour)
- Built and deployed via `deploy.sh` — immediate HTTP 200

**Post-fix — manual re-transcoding:**
- Wrote `retranscode.sh` using direct ffmpeg calls (no Node.js/Prisma client issues)
- All 3 videos processed sequentially (360p → 720p → 1080p + master playlist each)
- DB updated: `transcodeStatus` = `completed`, `storagePath` = HLS master.m3u8
- All 3 completed successfully

**Related:** OOM transcode incident (2026-04-13), sequential transcoding fix, PM2 OOM restart loop

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

---

## [2026-04-25] build | Anker SOLIX F3000 — Grafana Dashboards Created

**Type:** Monitoring Setup — IN PROGRESS 🔄
**Reporter:** Adrianna

### Tasks Completed
1. **Database Setup**
   - PostgreSQL container (`anker-postgres:15-alpine`) running with `anker` db
   - Table `f3000_metrics` with 39 columns for all confirmed live MQTT fields
   - Index on `(device_sn)` and `(timestamp DESC, device_sn)`

2. **Poller Script**
   - `f3000_to_postgres.py` subscribes to Anker MQTT broker via `SolixMqttDevicePps`
   - Writes confirmed live fields only: `output_power_total`, `temperature`, `battery_soc`, `ac_input_limit`, `ac_output_power_switch`, etc.
   - Systemd timer runs every 60 seconds (`f3000-monitor.timer`)

3. **Grafana Dashboards**
   - Datasource `anker-postgres` configured (UID: `efk5jsmhf4t8gd`)
   - Created `flux-capacitor-v2` dashboard with 11 panels (gauges, stats, time series, table)
   - Created `HARDCODED DB CHECK` dashboard with explicit device SN in queries (no variables)
   - All API queries return correct data (e.g., ~1867W output power, 21°C)

### Blocker / Issue
- Grafana browser render shows dramatically different values from API/backend:
  - API returns output_power_total ~1867W, but dashboard gauge shows ~93W
  - Temperature history graph shows 62–85°C range instead of constant ~21–25°C
  - Boolean fields show numeric values (31.8, 72.5) instead of ON/OFF
  - Troubleshooting: no other databases, datasources, or tables found; no transformations configured; hardcoded queries also affected
  - **Status:** Root cause unidentified. Grafana Explore testing requested to isolate browser vs backend issue.
