---
title: PunClaw Dashboard
type: summary
tags: [fleet-management, openclaw, dashboard, punclaw, sqlite, ssh-proxy]
sources: 1
created: 2026-04-07
updated: 2026-04-07
---

## Summary

Centralized fleet management platform for OpenClaw agent VMs ("claws"). Built in Next.js 16 with SQLite backend, it provides provisioning, heartbeat monitoring, token usage tracking, channel integration wizards (Telegram, Google Chat), WebSocket terminal access, and fleet-wide skills deployment via SSH proxy using a shared fleet key.

## Key Points

- **Host:** 10.255.245.162 (public: 208.90.101.13), runs Next.js app + PTY server via PM2
- **Database:** SQLite at `/home/openclaw/apps/PunClaw-Dashboard/data/punclaw.db`
- **SSH Proxy:** Fleet key (`/home/openclaw/.ssh/punclaw_fleet`) authorized on all managed claws for passwordless access
- **Managed Claws:** 6 claws (.163 ROB, .164 CEDRIC, .167 Wayne, .168 Brett, .169 Eve, .180 Mai)
- **Authentication:** Cookie-based sessions with `crypto.scrypt` password hashing, roles: admin/tech
- **Heartbeat:** Every 5 minutes via cron, reports IP/CPU/memory/disk/uptime/gateway status
- **Usage Tracking:** Parses session JSONL files for model/provider/usage metrics, incremental watermark
- **Channel Wizards:** Telegram (6-step), Google Chat (5-step with auto-DNS/nginx/config)
- **WebSocket Terminal:** PTY server on port 3001, proxied via nginx `/ws/terminal`
- **Fleet Skills:** Push workspace skills tarballs across all claws via SSH
- **Branding:** Part of Anzen umbrella — Pund-IT security product line

## Connections

- Related to [[openclaw-fleet-management]] — core concept of managing distributed agents
- Related to [[ssh-proxy-patterns]] — SSH fleet key architecture enabling dashboard access
- Related to [[anzen-egress-portal]] — another Anzen-branded product under Pund-IT security umbrella

## Details

### Architecture

The dashboard acts as a central control plane for a fleet of OpenClaw instances. It uses SSH proxying through a shared fleet key to execute commands and read files on remote claws without individual SSH credentials.

**Two-hop nginx routing:**
1. Cloudflare → .162 nginx (SSL termination, wildcard `*.pund-it.ca`)
2. .162 nginx → per-subdomain server block → claw nginx (HTTP)
3. Claw nginx → gateway (loopback)

### Provisioning

- **Fresh claws:** 5-step wizard (Connect → Configure → Channels → Providers → Launch)
- **Existing claws:** 2-step read-only registration (Connect → Register)
- Auto-installs heartbeat script, usage collector, static IP netplan config

### Key Files

- `src/lib/db.ts` — SQLite: users, claws, sessions, usage
- `src/lib/vm-proxy.ts` — SSH proxy via fleet key
- `src/app/api/claws/heartbeat/route.ts` — Heartbeat ingestion (X-Claw-Key auth)
- `src/app/api/usage/route.ts` — Usage ingestion + query
- `src/app/api/integrations/googlechat-setup/route.ts` — Auto-DNS/nginx/SA upload
- `pty/pty-server.js` — WebSocket terminal server
- `scripts/punclaw-heartbeat.sh`, `scripts/punclaw-usage-collector.sh` — Deployed to claws

### Future Work

Slack wizard, Fleet Overview page, User Management UI, Reports page, N8N integration, Cost calculation (pricing data needed).

---

**Repo:** https://github.com/crashf/PunClaw-Dashboard.git  
**MVP Tag:** `v1.0.0-mvp` (2026-03-15)