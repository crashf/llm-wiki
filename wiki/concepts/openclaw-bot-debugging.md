---
title: OpenClaw Bot Debugging
type: concept
tags: [openclaw, debugging, troubleshooting, configuration, ssh, linux]
sources: 1
created: 2026-04-08
updated: 2026-04-13
---

## Summary

Patterns for diagnosing and fixing unresponsive or looping OpenClaw gateway instances on managed claws. Covers config validation errors, systemd service management, log analysis, and common root causes.

## Key Points

- **Config validation errors cause startup loops** — keys from newer OpenClaw versions rejected by older binaries; remove unknown keys from `openclaw.json`
- **Minimax configs cause startup failures on 2026.3.13+** — OpenClaw rejects `plugins.entries.minimax` and `minimax-portal` provider configs; symptoms: bot appears offline, gateway won't start
- **Minimax fix pattern** — Remove all minimax entries (auth, provider, fallbacks), set default to `ollama/kimi-k2.5:cloud`, keep ollama plugin, restart gateway
- **Systemd service** (`openclaw-gateway.service`) runs as root on port 3578; managed with `systemctl start/stop/restart openclaw-gateway`
- **Startup CPU spike** — LCM compaction of large conversation DB (50-100MB) causes temporary high CPU that settles within ~2 minutes
- **Gateway log location** — `/tmp/openclaw/openclaw-YYYY-MM-DD.log` on the bot; also via `journalctl` for systemd units
- **Port conflict** — "Port 3578 is already in use" means a previous gateway instance is still running; kill it first
- **Telegram not connecting** — check if `channels.telegram.enabled` is `true` in config; gateway requires restart on channel toggle
- **Gateway already running lock** — `pid {N} root: openclaw-gateway (*:3578)`; use `openclaw gateway stop` or kill the PID

- **Feishu/Lark SDK missing** — `@larksuiteoapi/node-sdk` not installed; causes probe errors on every HTTP request to `__openclaw__/canvas/`
- **Google Chat not responding** — Check for `CHAT_API_BASE is not defined` error in logs; indicates patch removal of constants. See [[google-chat-patch-2026-04-09]] for fix.
- **LCM plugin disabled but logging errors** — `lossless-claw` plugin is `enabled: false` in config but logs ERROR-level messages about LCM compaction model being unconfigured
- **Plugins disabled but present** — acpx, anthropic, minimax, openai, xai, slack, lossless-claw all show config warnings (disabled but config exists); not a crash-level issue but fills the log
- **Config is owned by rightclaw user** — `/home/rightclaw/.openclaw/openclaw.json` is readable by adrianna sudo, but not writable without elevated privileges
- **Restart strategy** — graceful restart via `systemctl restart` picks up newly installed npm packages (like `@larksuiteoapi/node-sdk`)
- **Gateway health probe** — `curl http://<ip>:18789/__openclaw__/health` works but the feishu probe errors spam the gateway log every time it's hit
- **Agent model config** — agents.defaults.model.primary is correctly set to `minimax-portal/MiniMax-M2.5` even though minimax plugin entry itself is `enabled: false` — the plugin and the model are separate

## SSH Access Pattern

All fleet claws are accessible via SSH from the dashboard host:

```bash
sshpass -p '@dr1anna@12' ssh -o StrictHostKeyChecking=no adrianna@<claw-ip>
```

Or with fleet key (dashboard .162 only):
```bash
ssh -i ~/.ssh/punclaw_fleet openclaw@<claw-ip>
```

## Diagnostic Checklist

When a claw is unresponsive:

1. **Ping** — `curl -s --max-time 3 http://<ip>:3578/__openclaw__/health`
2. **Process check** — `ps aux | grep openclaw | grep -v grep`
3. **Systemd status** — `sudo systemctl status openclaw-gateway`
4. **Recent logs** — `sudo tail -50 /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log`
5. **Config validity** — `openclaw gateway doctor` (non-interactive: `openclaw gateway run --verbose`)
6. **Port conflict** — `ss -tlnp | grep 3578`

## Config Cleanup

Known problematic keys to remove when upgrading from newer OC versions or when bots won't start:

```json
"plugins": {
  "lcm": { ... },     // ← remove if present (causes startup loops)
  "minimax": { ... }  // ← remove if present (OC 2026.3.13+ rejects)
},
"runtime": "..."       // ← remove if present
"minimax-portal"       // ← remove entire provider config block
```

Validate with: `openclaw gateway doctor`

## Fleet Config Fix (2026-04-13)

When minimax configs cause bot startup failures, apply this fix:

### Symptoms
- Bot unresponsive to Telegram/Google Chat
- `systemctl status openclaw-gateway` shows frequent restarts or failure loops
- May see config validation errors in logs

### Fix Sequence
```bash
# SSH to the claw
sshpass -p '@dr1anna@12' ssh -o StrictHostKeyChecking=no adrianna@<claw-ip>

# Edit config to remove all minimax entries, default to ollama
sudo nano /root/.openclaw/openclaw.json

# Changes to make:
# 1. In auth.profiles: delete "minimax-portal:default"
# 2. In models.providers: delete entire "minimax-portal" block
# 3. In agents.defaults.model: change primary to "ollama/kimi-k2.5:cloud"
# 4. In agents.defaults.model.fallbacks: remove all minimax-portal entries
# 5. In plugins.entries: delete "minimax" entry (keep only "ollama")
# 6. In plugins.allow: remove "minimax"

# Restart gateway
sudo systemctl restart openclaw-gateway

# Verify
sudo systemctl status openclaw-gateway --no-pager
```

### Verification
- Gateway should show as "active (running)" without frequent restarts
- Bot should respond to messages within 30 seconds
- Dashboard should show "online" status with recent heartbeat

## Connections

- Related to [[openclaw-fleet-management]] — fleet operations depend on all claws being healthy
- Related to [[ssh-proxy-patterns]] — SSH is the access layer for all debugging
