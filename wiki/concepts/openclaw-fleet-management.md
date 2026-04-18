---
title: OpenClaw Fleet Management
type: concept
tags: [openclaw, fleet-management, distributed-agents, ssh-proxy, automation]
sources: 1
created: 2026-04-07
updated: 2026-04-07
---

## Summary

Pattern for managing multiple OpenClaw agent instances ("claws") from a central dashboard. Combines SSH proxy via shared fleet key, SQLite-backed inventory, heartbeat monitoring, and fleet-wide operations (skills deployment, system monitoring, usage tracking).

## Key Points

- **Central Dashboard:** Single control plane (PunClaw Dashboard) manages all claws
- **SSH Fleet Key:** One key authorized on all claws enables passwordless proxy from dashboard
- **Heartbeat:** Every 5 minutes, claws report IP/CPU/memory/disk/uptime/gateway status to dashboard
- **Usage Tracking:** Session JSONL parsing with incremental watermark, per-model/provider/agent rollups
- **Per-Claw Isolation:** Each claw maintains its own `openclaw.json`, workspace, and channels
- **Fleet Operations:**
  - Skills push (tar + base64 via SSH)
  - System monitoring (SSH-based CPU/memory/disk/uptime)
  - WebSocket terminal access (PTY server, fleet key auth)
  - Search (per-claw, per-agent scoped grep)
- **Channel Integration:** Central wizards for Telegram/Google Chat/Slack configure claws remotely
- **Provisioning:** Fresh claws (full wizard) vs existing claws (read-only registration)

## Connections

- Related to [[punclaw-dashboard]] — implementation of fleet management pattern
- Related to [[ssh-proxy-patterns]] — SSH fleet key for passwordless claw access
- Related to [[anzenegress-portal]] — Anzen umbrella product, separate but same branding philosophy

## Details

### Architecture

```
Dashboard (10.255.245.162)
    │
    ├── SQLite DB (claws, users, sessions, usage)
    ├── Fleet SSH Key (/home/openclaw/.ssh/punclaw_fleet)
    │
    └── SSH Proxy ───→ Claw .163 (ROB)
           │             Claw .164 (CEDRIC)
           │             Claw .167 (Wayne)
           │             Claw .168 (Brett)
           │             Claw .169 (Eve)
           │             Claw .180 (Mai)
```

### Heartbeat Protocol

1. Claw cron runs `/usr/local/bin/punclaw-heartbeat` every 5 min
2. Script reads system stats, packages with claw key
3. POST to `/api/claws/heartbeat` on dashboard
4. Dashboard updates DB, auto-corrects IP if changed

### Usage Collection Protocol

1. Claw cron runs `scripts/punclaw-usage-collector.sh` every 5 min
2. Parses `{ocHome}/agents/*/sessions/*.jsonl` for assistant messages
3. Extracts model, provider, usage (input/output tokens)
4. Incremental watermark prevents re-processing
5. POST to `/api/usage` on dashboard

### Fleet Skills Deployment

1. Dashboard fetches skill from source claw (via SSH)
2. Creates tarball, base64 encodes
3. POSTs to each target claw's `/api/fleet/skills` endpoint
4. Target decodes, extracts to `/root/.openclaw/workspace/skills/`

### Per-Claw Paths

Not all claws use the same OpenClaw home directory:

| Claw | IP | OpenClaw Home |
|------|-----|---------------|
| ROB | .163 | `/home/openclaw/.openclaw` |
| CEDRIC | .164 | `/home/openclaw/.openclaw` |
| Wayne | .167 | `/root/.openclaw` |
| Brett | .168 | `/root/.openclaw` |
| Eve | .169 | `/root/.openclaw` |
| Mai | .180 | `/root/.openclaw` |
| Danielle | .189 | `/root/.openclaw` |
| Rosita | .188 | `/root/.openclaw` |
| Colin | .171 | `/root/.openclaw` |
| A-Team | .153 | `/root/.openclaw` |
| Support-Escellations | .192 | `/root/.openclaw` |

Dashboard detects this during provisioning/registration.

### Fleet Size Evolution

- **2026-03:** 6 claws (ROB, CEDRIC, Wayne, Brett, Eve, Mai) — MVP
- **2026-04-08:** 11 claws — Danielle, Rosita, Colin, A-Team, Support-Escellations added
- Heartbeat API: `GET /api/grafana/claws` returns all claws with CPU/mem/disk/time

### Startup Behavior

First-run LCM compaction of the conversation database (typically 50-103MB) causes a temporary CPU spike (132%) that settles to ~13% within 1-2 minutes. This is normal — the compaction runs once per process start and does not repeat.

Config key compatibility: OpenClaw version 2026.3.13 rejects `plugins.lcm` and `runtime` keys from newer config schemas. Always validate config against the running version's schema.

### Anzen Brand Context

"OpenClaw Fleet Management" is part of Pund-IT's Anzen umbrella — security products under unified branding:

- **Anzen DNS** — DNS filtering/security
- **Anzen Protect** — Endpoint protection  
- **Anzen Egress** — WireGuard VPN tunnels
- **Anzen Fleet** — OpenClaw fleet management (this concept)

Each product maintains separate branding (product name + Anzen prefix) while sharing Pund-IT's parent brand identity.