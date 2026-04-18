---
title: cPanel/WHM Skill
type: concept
tags: [cpanel, whm, api, skill, infrastructure, automation]
sources: 1
created: 2026-04-10
updated: 2026-04-10
---

## Summary

OpenClaw skill for interacting with cPanel/WHM servers via API. Provides comprehensive coverage for account management, email, databases, DNS, SSL certificates, domains, and backups.

## Key Points

- **Location:** `~/.openclaw/workspace/skills/cpanel-whm/SKILL.md`
- **Credentials:** Stored in `~/.openclaw/workspace/.secrets/cpanel-whm-creds.txt`
- **Endpoint:** `https://tech-method.com:2087` (WHM root access)
- **Authentication:** WHM API token (`root:S03M999X8YDGZGA1K8AZKOZFNITCFS16`)
- **Critical Rule:** Human-in-the-Middle — never make changes without explicit user confirmation

## Capabilities

| Category | Operations |
|----------|------------|
| **Accounts** | Create, suspend, terminate, modify quotas/packages |
| **Email** | Accounts, forwarders, filters |
| **Databases** | MySQL create/drop, users, grants |
| **DNS** | Zones, records, transfers |
| **SSL** | Install, AutoSSL management |
| **Domains** | Addon, parked, subdomains |
| **Backups** | Generate, restore, migrate |

## API Pattern

```bash
curl -k -H "Authorization: whm root:TOKEN" \
  "https://tech-method.com:2087/json-api/ENDPOINT"
```

## cPanel Accounts

Current accounts on `tech-method.com`:
- `bithooker` (8bithooker.com)
- `boulevardlimo` (boulevardlimousine.ca)
- `funkautospa` (funkenautospa.com)
- `twelvegaugecoati` (twelvegaugecoatings.com)
- `clubsixty` (clubsixty.ca) — *separate client, no Pund-IT branding*
- `systematik` (systematikhosting.com)
- `linkcomm` (linkcomm.ca)
- `voicefusion` (voicefusion.ca)
- `builtbywordpress` (builtbywordpress.com)
- `alternativesjour` (alternativesjournal.ca)
- `fugitivecustom` (fugitivecustomcars.ca)

## Connections

- Related to [[club-sixty-six]] — uses cPanel for DNS/mail (clubsixty.ca domain)
- See also [[openclaw-fleet-management]] — both use centralized credential storage pattern
