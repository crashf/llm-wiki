# Anzen Egress — Management Portal

> Zero-CLI deployment portal for Anzen Egress WireGuard tunnels. Turns a manual, SSH-heavy MikroTik provisioning process into a wizard-driven web interface any Pund-IT tech can use.

## The 3-Phase Model

All CPE deployments follow three phases — zero MikroTik CLI required at any stage.

### Phase 1: Pre-Stage (Office/Warehouse)

1. Tech opens portal → **"Pre-Stage New CPE"**
2. Portal generates a staging WireGuard keypair and a bootstrap `.rsc` script containing:
   - WAN config (DHCP client on selected interface)
   - WG tunnel to staging endpoint (`wg-staging`, port 51821) on selected gateway
   - Minimal firewall (API access over WG only, deny everything else)
   - NTP, DNS basics, provisioning user
3. Tech applies bootstrap via **USB auto-import** (preferred — MikroTik autoruns `autorun.rsc` from USB after factory reset) or **Netinstall**
4. CPE is marked **"Pre-Staged"** — sitting on shelf, ready to ship

> **Key insight:** Bootstrap is intentionally minimal. It just phones home. All real configuration happens in Phase 2 over the staging tunnel.

### Phase 2: Deploy (On-Site or Remote — From Anywhere)

1. Pre-staged CPE plugs into any internet connection at target site
2. CPE boots → WAN gets DHCP → WG staging tunnel auto-connects
3. Portal detects CPE online: **"CPE XXXX — Online, Awaiting Assignment"**
4. Tech opens portal → **Deployment Wizard** (9 steps):

| Step | Selection |
|------|-----------|
| 1 | Select CPE from "online, unassigned" list |
| 2 | Customer (search, select) |
| 3 | Gateway (edge router) |
| 4 | IP Allocation (auto-pick /30 or manual) |
| 5 | WAN Configuration (interface, DHCP/static/PPPoE) |
| 6 | LAN Configuration (bridge ports, DHCP, VLAN, DNS) |
| 7 | Security Policy (firewall profile) |
| 8 | Management Access (allowed IPs, SSH users) |
| 9 | Review & Deploy |

5. On **Deploy**, the portal:
   - Allocates IPs from pool (marks as used)
   - Generates full production config dynamically from wizard selections
   - **Pushes to Edge Router** via MikroTik REST API: adds WG peer, routes, verifies proxy-ARP
   - **Pushes to CPE** via MikroTik REST API over staging WG tunnel: production keys, WAN/LAN/firewall/mgmt config
   - Reboots CPE → comes up on production tunnel
6. Post-deployment validation: WG handshake, ping, DNS/HTTP, MTU/MSS
7. Deployment marked **"Active"**

### Phase 3: Day-2 Operations

- Live tunnel status (up/down, latency, last handshake)
- Remote config changes without site visit
- Firmware management, backup/restore
- Troubleshooting (ping, traceroute, torch, packet sniff via API)
- Usage tracking, billing reports
- Clean decommission (free IPs, remove WG peer, archive)

---

## Tech Stack

| Component | Technology | Rationale |
|-----------|-----------|-----------|
| **Frontend** | Next.js 15 (App Router) | Wayne's team knows it |
| **UI Library** | shadcn/ui + Tailwind CSS | Fast, professional, dark mode |
| **Backend** | Next.js API Routes + Server Actions | Single codebase |
| **Database** | PostgreSQL | Relational + JSONB flexibility |
| **ORM** | Drizzle ORM | Type-safe, lightweight, Drizzle Kit migrations |
| **Auth** | NextAuth.js v5 + Azure AD | SSO, configurable tenant |
| **MikroTik API** | Custom REST client (RouterOS 7+) | Direct HTTP to routers, no SSH |
| **CW Manage API** | Custom REST client | Companies, agreements, additions |
| **WG Key Gen** | Node.js `crypto.generateKeyPairSync("x25519")` | Native X25519, no native deps |
| **Encryption** | AES-256-GCM | WG private keys + API secrets at rest |
| **Metrics** | Built-in poller + Prometheus `/api/metrics` | Portal dashboards + Grafana |
| **Hosting** | Docker on punsvnfdocker01/02 | Low overhead, existing infra |

---

## Key Integrations

### MikroTik Edge Routers (REST API)

- RouterOS 7+ REST API (`/rest/...`) for all config pushes
- Edge router operations: add WG peer, add routes, verify proxy-ARP status
- CPE operations over staging tunnel: push full production config, reboot
- Health checks: API reachability, WG endpoint status, interface stats

### ConnectWise Manage (Planned)

- **Company search** in deployment wizard — search CW companies by name
- **Agreement mapping** — select CW agreement to map usage/billing
- **Addition tracking** — create/update CW agreement additions for Anzen line items
- **Ticket creation** — optionally create CW tickets on critical alerts
- Billing models per customer: flat, usage (per-GB), tiered (included + overage)

### Grafana (Planned)

- Portal exposes `/api/metrics` in Prometheus format
- Auto-provisioned Grafana datasource
- Pre-built dashboards: Fleet Overview, Per-Customer Detail, Billing Summary
- Deployed to existing Grafana at `10.255.71.48:3000`

Prometheus metric examples:
```
anzen_tunnel_status{deployment_id="...", customer="...", gateway="..."} 1
anzen_tunnel_latency_ms{deployment_id="...", customer="..."} 12.5
anzen_tunnel_rx_bytes_total{deployment_id="...", customer="..."} 1234567890
anzen_tunnel_tx_bytes_total{deployment_id="...", customer="..."} 987654321
anzen_active_tunnels_total 15
```

---

## Hosting

Docker deployment on **punsvnfdocker01** (10.255.71.46) or **punsvnfdocker02** (10.255.71.48) — whichever has lowest usage at deploy time. Served behind Nginx Proxy Manager at `anzen.pund-it.ca`.

---

## Database Schema Overview

The PostgreSQL schema uses Drizzle ORM with JSONB for flexible config storage. Core tables:

| Table | Purpose |
|-------|---------|
| `gateways` | Edge routers (name, IPs, WG keys, staging keys, API creds, proxy-ARP interface, status) |
| `ip_pools` | Public IP pools per gateway (CIDR, allocation size) |
| `ip_allocations` | Per-/30 allocations from pools (subnet, gateway_ip, customer_ip, status, deployment FK) |
| `transit_pools` | Transit network pools per gateway (10.99.0.0/24, 10.99.1.0/24) |
| `transit_allocations` | Per-/30 transit pairs (edge_ip, cpe_ip, deployment FK) |
| `customers` | Cached from CW + local enrichment (contacts, site address, CW company ID) |
| `cpe_devices` | CPE hardware (model, serial, staging/production WG keys, WAN config, status lifecycle: new → pre_staged → online_unassigned → deployed → offline → decommissioned) |
| `deployments` | The core record — customer, CPE, gateway, IP allocations, router_mode, LAN/WiFi/bridge config (JSONB), firewall profile, MSS clamp, config versioning, status lifecycle: provisioning → active → degraded → offline → decommissioned |
| `tunnel_metrics` | Raw 5-min samples from edge router (status, latency, rx/tx bytes, last handshake) |
| `tunnel_metrics_hourly` |Hourly rollups (uptime %, avg latency, bytes delta) |
| `tunnel_metrics_daily` | Daily rollups + peak hour bytes |
| `tunnel_metrics_monthly` | Monthly rollups + billing (p95 daily, CW agreement FK, billed flag) |
| `alert_rules` | Conditions (tunnel_down, high_latency, bandwidth_threshold, config_drift, firmware_outdated), thresholds, severity, notification channels |
| `alerts` | Fired alerts (deployment FK, rule FK, severity, message, details JSONB, acknowledged/resolved tracking) |
| `users` | Azure AD OID, email, role (admin/tech/viewer) |
| `sso_config` | Azure AD tenant config (admin-editable in GUI) |
| `cw_config` | CW Manage API credentials |
| `cw_agreement_mappings` | Deployment → CW agreement/addition mapping with billing type + config |
| `audit_log` | Every action (user, action, entity, details JSONB, IP address) |

Key design notes:
- WireGuard private keys stored AES-256-GCM encrypted at rest
- `router_mode` enum: `anzen_passthrough` (public /30 on LAN, no NAT) or `standard_router` (private LAN with NAT, DHCP, WiFi)
- Config versioning: `config_version` counter + `last_config_backup` JSONB snapshot
- All CPE/WAN/LAN config in JSONB columns — no rigid per-column schema for dynamic router configs

---

## Current Status

**MVP complete.** The staging/pre-stage workflow is live — bootstrap keypair generation, `.rsc` download, edge peer addition, and the 9-step deployment wizard are functional. Private LAN + WiFi + bridge config, MikroTik REST client, gateway health checks, IPAM with transit allocations, and usage metrics with Prometheus exporter are built.

Not yet built: ConnectWise integration, Grafana dashboards, Alert engine, Billing reports, Azure AD SSO, User management.

---

*Related: [WireGuard Architecture](./anzen-egress-wireguard.md) · [Config Generator](./anzen-egress-config-generator.md) · [MVP Status](../summaries/anzen-egress-mvp.md)*