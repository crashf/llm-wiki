# Anzen Egress — MVP Status

> What's built, what's not, key decisions, and lessons learned as of April 2026.

## What's Built ✅

| Feature | Details |
|---------|---------|
| **Pre-Stage Page** | 3-step wizard (CPE info → Gateway → Download), generates staging WG keypair, produces downloadable `.rsc` bootstrap config, adds edge peer via REST API |
| **Bootstrap .rsc Generation** | `src/lib/bootstrap-config.ts` — complete staging tunnel config (WG, DHCP WAN, staging routing table, minimal firewall, provisioning user, service hardening) |
| **USB Bundle** | `autorun.rsc` + `bootstrap.rsc` + `README.txt` for factory-reset auto-import |
| **Deployment Wizard** | 9-step flow at `/deployments/new` — CPE, Customer, Gateway, IP Allocation, WAN Config, LAN Config, Security, Management, Review & Deploy |
| **Router Mode Selector** | `anzen_passthrough` vs `standard_router` — determines which config generator runs |
| **Private LAN + WiFi + Bridge Config** | Full standard_router mode config generation: bridge ports, private subnet, DHCP server, WiFi (2.4/5/separate, WPA2/WPA3, guest), NAT masquerade |
| **MikroTik REST API Client** | Custom HTTP client for RouterOS 7+ `/rest/` endpoints — used for edge peer addition and CPE config pushes |
| **Gateway Health Checks** | API reachability, WG endpoint status verification |
| **IPAM** | IP pool CRUD, /30 auto-allocation from /28 pools, subnet utilization tracking |
| **Transit Allocations** | /30 transit pair allocation for both production (10.99.0.0/24) and staging (10.99.1.0/24) transit networks |
| **Usage Metrics** | Prometheus exporter at `/api/metrics` — per-tunnel status, latency, rx/tx bytes, handshake timestamps |
| **Post-deploy Validation** | WG handshake detection, ping test, DNS/HTTP connectivity, MTU/MSS verification |
| **Full DB Schema** | Drizzle ORM schema with all tables deployed and migrated |

## What's Not Built Yet ❌

| Feature | Status |
|---------|--------|
| **ConnectWise Integration** | Schema ready (`cw_config`, `cw_agreement_mappings`), no API client or wizard integration |
| **Grafana Dashboards** | Prometheus metrics endpoint exists, no Grafana datasource provisioning or dashboards |
| **Alert Engine** | Schema ready (`alert_rules`, `alerts`), no evaluation/notifications logic |
| **Billing Reports** | Monthly rollups schema ready, no billing calculation or CW addition sync |
| **Azure AD SSO** | NextAuth + `sso_config` table ready, no Azure provider configured |
| **User Management** | `users` table exists, no admin UI for role assignment |
| **Dashboard Home** | No fleet overview, activity feed, or quick actions page |
| **Day-2 Ops** | No remote config push, firmware management, backup/restore, or diagnostics UI |
| **Decommission Flow** | No clean removal of WG peers, IP release, or archive workflow |

---

## Key Decisions

### Staging / Pre-Production Split

Separate WireGuard tunnels (`wg-anzen` port 51820, `wg-staging` port 51821) on separate transit networks. This means:
- Pre-staged CPEs phone home over staging without polluting production
- Portal can push full production config over the staging tunnel
- Production and staging are fully isolated — a staging failure doesn't affect live customers
- Both tunnels have independent routing tables (`anzen` and `staging` FIBs)

### Model Dropdown — Not Template Lock-In

The CPE model dropdown is **informational only** — it doesn't determine config:
- hAP ac3, hAP ax3, RB5009, CCR2004-1G-2S+, Chateau, Other
- All configs are generated dynamically from interface/type selections
- No per-model templates exist. New models work by simply entering their interface names

### REST API Over SSH for CPE Config Pushes

Chose MikroTik REST API (`/rest/`) instead of SSH exec for configuration pushes:
- Structured JSON responses vs parsing CLI output
- Idempotent — read-before-write is natural
- Better error handling (HTTP status codes)
- No need for SSH key management on CPEs
- Caveat: REST API only available on RouterOS 7+

---

## Lessons Learned

### proxy-ARP is Essential for /30 Handoff

Without proxy-ARP on the edge router's `ether1`, the upstream provider won't route customer public IPs to the edge. The edge router must respond to ARP requests for customer IPs on its WAN interface. This is configured per-gateway (`proxyArpInterface` column, default `ether1`).

### tweetnacl Not Needed — Node crypto Handles X25519

Originally planned to use the `tweetnacl` library for Curve25519 key generation. Node.js `crypto.generateKeyPairSync("x25519")` handles X25519 natively with proper clamping. The raw 32-byte private key export is directly compatible with WireGuard. No third-party crypto dependencies needed.

### MikroTik SSH Exec Path

The MikroTik SSH exec path is `/tool/ssh-exec`, not `/system/ssh-exec` as some documentation suggests. This matters if using SSH-based exec for any operations, though the portal primarily uses REST API.

### DER Header Stripping for Key Export

Node's X25519 key export in DER format includes ASN.1 headers that WireGuard doesn't use:
- Public key: 12-byte SPKI header → skip, take last 32 bytes
- Private key: 16-byte PKCS8 header → skip, take last 32 bytes
- The stripped bytes, base64-encoded, produce identical output to `wg genkey`/`wg pubkey`

### RouterOS 7 Routing Table Requirement

`/routing/table/add name=anzen fib` must be created **before** any routes reference it. RouterOS 7 enforces this — routes referencing non-existent tables will fail silently or error.

### WG IP Address /32 with Network Parameter

Assigning transit IPs as `/32` with a `network` parameter is the correct RouterOS pattern for WireGuard point-to-point links:
```routeros
/ip/address/add address=10.99.0.6/32 interface=wg-anzen network=10.99.0.5
```
This avoids needing a separate route for the edge peer and is cleaner than /30 interface addresses.

---

*Related: [Portal Overview](../concepts/anzen-egress-portal.md) · [WireGuard Architecture](../concepts/anzen-egress-wireguard.md) · [Config Generator](../concepts/anzen-egress-config-generator.md)*