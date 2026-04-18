---
title: Anzen Egress
type: concept
tags: [wireguard, vpn, networking, pund-it, service]
sources: 2
created: 2026-04-07
updated: 2026-04-07
---

## Summary

Anzen Egress is Pund-IT's managed WireGuard VPN service that provides customers with true routed public IPs over WireGuard tunnels — no NAT, full identity. The service uses MikroTik edge routers as WireGuard hubs and CPE (Customer Premises Equipment) devices as spokes, creating a routed /30 handoff where customers receive a public IP directly.

## Key Points

- **Service Model**: Routed public IP over WireGuard — customer gets `170.205.18.x` directly, not behind NAT
- **Edge Routers**: MikroTik devices acting as WireGuard hubs (current: Netflash DC at `170.205.18.9`)
- **Two Tunnel Types**: Production (`wg-anzen`, port 51820) and Staging (`wg-staging`, port 51821) for phone-home pre-staging
- **Zero-Touch**: Pre-stage CPEs in office with bootstrap config, ship ready-to-go, deploy remotely via portal
- **IP Pool**: Starts with `170.205.18.240/28`, admin adds more via GUI
- **Transit Network**: `10.99.0.0/24` for edge-to-CPE communication (e.g., `10.99.0.1` → `10.99.0.2`)
- **Billing Models**: Flat (fixed monthly), Usage (per-GB), Tiered (included GB + overage)

## How It Works

### Pre-Stage Phase
1. Tech generates WireGuard keypair + bootstrap `.rsc` in portal
2. Bootstrap includes WAN (DHCP), staging WG tunnel, minimal firewall
3. Apply via USB auto-import or Netinstall
4. CPE phones home on staging WG — ready to ship

### Deploy Phase
1. CPE plugged in at site, gets internet from local ISP
2. Staging tunnel connects → portal shows "Online, Awaiting Assignment"
2. Tech runs 10-step wizard (customer → agreement → gateway → IPs → WAN → LAN → firewall → review)
3. Portal pushes production WG config + full router config over staging tunnel
4. CPE reboots on production tunnel → customer live

### Day-2 Operations
- Monitoring: tunnel status, latency, bandwidth per customer
- Alerts: tunnel down, high latency, bandwidth thresholds
- Config updates: push changes remotely without site visit
- Billing: usage rolled up monthly, synced to CW agreement additions

## Connections

- Managed by [[anzen-egress-portal]] — the Next.js portal that automates deployment
- Uses [[mikrotik-edge-router]] — MikroTik EdgeRouter as the WireGuard hub
- Integrates with [[connectwise-manage]] — for customer lookup and billing
- Related to [[wireguard]] concept — underlying VPN technology
