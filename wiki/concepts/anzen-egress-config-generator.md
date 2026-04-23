# Anzen Egress — Config Generator

> Dynamic RouterOS config generation for CPEs. No templates — everything is built programmatically from wizard selections.

## Router Modes

The config generator supports two fundamentally different router modes:

### Anzen Passthrough (`anzen_passthrough`)

The original mode. Customer gets a **public /30 directly on their LAN**, no NAT.

- Public IP assigned to CPE's LAN interface
- CPE runs DHCP server on the customer /30 (tiny range — 1 address)
- Traffic from the public /30 is policy-routed through the WG tunnel via `src-address` routing rule
- No masquerade — true routed public IP
- Use case: Customers who need real public IPs on their equipment (servers, VoIP, etc.)

### Standard Router (`standard_router`)

Private LAN with NAT, DHCP, WiFi, bridge — a traditional SOHO router.

- CPE has private subnet on a bridge (e.g., 192.168.88.0/24)
- NAT masquerade on the WG interface
- All traffic goes through tunnel (no split routing — `routing-rule` matches all)
- Full WiFi, DHCP server for private range
- Use case: Standard office/retail customers who just need internet

---

## WAN Configuration

Three WAN types, each generating complete RouterOS config:

### DHCP (default)

```routeros
/ip/dhcp-client/add interface=ether1 use-peer-dns=no use-peer-ntp=yes \
  add-default-route=no comment="WAN via DHCP (Anzen will provide default route)"
```

Note: `add-default-route=no` because Anzen provides the default route via the WG tunnel routing table. In standard_router mode, the WAN default is a backup (lower priority).

### Static IP

```routeros
/ip/address/add address=203.0.113.50/24 interface=ether1 comment="WAN static"
/ip/route/add dst-address=0.0.0.0/0 gateway=203.0.113.1 comment="WAN gateway (backup, lower priority)"
```

For passthrough mode, the WAN default route is the primary route (used for WG endpoint reachability). For standard_router mode, it's a lower-priority backup.

### PPPoE

```routeros
/interface/pppoe-client/add interface=ether1 name=pppoe-wan user=<username> \
  password=<password> service-name=<service> \
  add-default-route=no use-peer-dns=no
```

PPPoE credentials are stored in the `wanPppoeConfig` JSONB column on `cpe_devices`.

---

## LAN Configuration

### Passthrough Mode LAN

Minimal — just the public /30 on the LAN interface:
```routeros
/ip/address/add address=170.205.18.241/30 interface=ether2 comment="Anzen public LAN"
```

Optional DHCP server on the /30 (1 IP in the range):
```routeros
/ip/pool/add name=pool-anzen ranges=170.205.18.242
/ip/dhcp-server/network/add address=170.205.18.240/30 gateway=170.205.18.241 dns-server=1.1.1.1,8.8.8.8
/ip/dhcp-server/add name=dhcp-anzen interface=ether2 address-pool=pool-anzen lease-time=1d
```

### Standard Router Mode LAN

Full bridge + private subnet:

**Bridge:**
```routeros
/interface/bridge/add name=bridge-lan comment="LAN Bridge"
/interface/bridge/port/add bridge=bridge-lan interface=ether2 comment="ether2 LAN port"
/interface/bridge/port/add bridge=bridge-lan interface=ether3 comment="ether3 LAN port"
/interface/bridge/port/add bridge=bridge-lan interface=wifi-2ghz comment="WiFi 2.4GHz"
/interface/bridge/port/add bridge=bridge-lan interface=wifi-5ghz comment="WiFi 5GHz"
```

Bridge ports are configurable in the wizard (`bridgeConfig.ports` array — defaults to `["ether2","ether3","ether4","ether5"]`). WiFi interfaces are optionally included (`bridgeConfig.includeWiFi`, default true).

**Address & DHCP:**
```routeros
/ip/address/add address=192.168.88.1/24 interface=bridge-lan comment="LAN Gateway"
/ip/pool/add name=pool-lan ranges=192.168.88.100-192.168.88.200
/ip/dhcp-server/network/add address=192.168.88.0/24 gateway=192.168.88.1 dns-server=1.1.1.1,8.8.8.8
/ip/dhcp-server/add name=dhcp-lan interface=bridge-lan address-pool=pool-lan lease-time=1d
```

All values (subnet, gateway, DHCP range, DNS) come from the wizard's `privateLanConfig` object.

---

## WiFi Configuration

Only available in `standard_router` mode. Uses RouterOS 7 `/interface/wifi` API (not the legacy `/interface/wireless`).

### Band Options

| Band | Behavior |
|------|----------|
| `2.4ghz` | Single 2.4 GHz radio, one SSID |
| `5ghz` | Single 5 GHz radio, one SSID |
| `2.4ghz_5ghz` | Both radios, **same SSID** (roaming) |
| `separate` | Both radios, **different SSIDs** per band |

### Security Options

| Profile | RouterOS Mode |
|---------|--------------|
| `wpa2` | `wpa2-psk` |
| `wpa3` | `wpa3-psk` |
| `wpa2_wpa3` | `wpa2-wpa3-psk` (transition mode) |

### Generated Config Example (dual-band, same SSID, WPA3)

```routeros
# 2.4 GHz
/interface/wifi/security/add name=wifi-main-secpol mode=wpa3-psk \
  authentication-types=wpa3-psk passphrase="<password>"
/interface/wifi/configuration/add name=wifi-main-config mode=ap band=2ghz-g/n \
  ssid="<ssid>" security=wifi-main-secpol
/interface/wifi/add name=wifi-2ghz configuration=wifi-main-config disabled=no

# 5 GHz (same SSID, same security profile)
/interface/wifi/configuration/add name=wifi-main-config mode=ap band=5ghz-a/n/ac/ax \
  ssid="<ssid>" security=wifi-main-secpol
/interface/wifi/add name=wifi-5ghz configuration=wifi-main-config disabled=no
```

### Guest WiFi

Optional guest network on separate VLAN:
```routeros
/interface/wifi/security/add name=wifi-guest-secpol mode=wpa2-psk \
  authentication-types=wpa2-psk passphrase="<guest_password>"
/interface/wifi/configuration/add name=wifi-guest-config mode=ap band=2ghz-g/n \
  ssid="<guest_ssid>" security=wifi-guest-secpol
/interface/wifi/add name=wifi-guest configuration=wifi-guest-config disabled=no
```

Guest VLAN is set via `guestVlan` integer — the guest WiFi interface is added to the bridge with VLAN tagging.

---

## Firewall Profiles

### Standard

Allow all outbound, block WAN input, allow established/related + ICMP + management IPs + WG:

```routeros
# Input: accept established/related, drop invalid, allow ICMP, allow management, allow WG, drop WAN
# Forward: accept established/related, drop invalid, accept all
```

In standard_router mode, NAT provides outbound security. In passthrough mode, the customer handles their own inbound since they have a public IP.

### Strict

Standard + outbound filtering on the `output` chain:
```routeros
/ip/firewall/filter/add chain=output connection-state=established,related action=accept
/ip/firewall/filter/add chain=output protocol=dns action=accept
/ip/firewall/filter/add chain=output action=drop
```

Only allows established outbound and DNS. Good for lockdown scenarios.

### Minimal

Basic connection tracking only — customer manages their own firewall:
```routeros
# Input: accept established/related, drop invalid, allow ICMP
# Forward: accept established/related, drop invalid, accept all
```

### Custom

User provides raw RouterOS filter rules as string array in `customFirewallRules`. Injected verbatim into the generated config.

---

## Management Access

Standard management IP ranges (pre-populated in wizard):

| Range | Description |
|-------|-------------|
| 10.99.0.0/24 | WG transit network |
| 10.255.71.0/24 | DC management VLAN |
| 208.90.101.0/28 | Pund-IT office |

All become address-list entries:
```routeros
/ip/firewall/address-list/add address=10.99.0.0/24 list=management comment="management access"
/ip/firewall/address-list/add address=10.255.71.0/24 list=management comment="management access"
/ip/firewall/address-list/add address=208.90.101.0/28 list=management comment="management access"
```

Custom IPs can be added. SSH users are created with full group:
```routeros
/user/add name=<username> group=full password="<password>"
```

---

## MSS Clamp Tuning

Default 1360, configurable per deployment. Applied as mangle rules in both directions:

```routeros
/ip/firewall/mangle/add chain=forward protocol=tcp tcp-flags=syn \
  in-interface=wg-anzen action=change-mss new-mss=1360 passthrough=yes
/ip/firewall/mangle/add chain=forward protocol=tcp tcp-flags=syn \
  out-interface=wg-anzen action=change-mss new-mss=1360 passthrough=yes
```

---

## Dynamic Generation — No Templates

The config generator (`src/lib/config-generator.ts`) is purely programmatic. There are zero `.rsc` template files. Every line of RouterOS config is constructed from TypeScript function calls that read the `ProductionConfigOptions` interface:

```typescript
export interface ProductionConfigOptions {
  cpeLabel: string;
  routerMode: "anzen_passthrough" | "standard_router";
  productionWgPrivkey: string;
  productionWgPubkey: string;
  gatewayWanIp: string;
  gatewayWgListenPort: number;
  gatewayWgPubkey: string;
  transitSubnet: string;
  cpeTransitIp: string;
  edgeTransitIp: string;
  customerSubnet: string;
  customerGatewayIp: string;
  customerIp: string;
  wan: WanConfig;
  lan: LanConfig;
  privateLanConfig?: PrivateLanConfig;
  wifiConfig?: WiFiConfig;
  bridgeConfig?: BridgeConfig;
  firewall: FirewallProfile;
  management: ManagementAccess;
  mssClamp: number;
}
```

The `generateProductionConfig()` function dispatches to `generatePassthroughConfig()` or `generateStandardRouterConfig()` based on `routerMode`. Each builds system identity, WG interface, IP addresses, WAN, routing tables, bridge/WiFi (if applicable), DHCP, NAT (if applicable), firewall, MSS clamp, DNS, NTP, management users, and service hardening.

This means:
- Any combination of WAN/LAN/WiFi/firewall is valid
- New CPE models require **no template changes** — just select the right interfaces
- Config is always consistent and auditable (same input → same output)

---

*Related: [Portal Overview](./anzen-egress-portal.md) · [WireGuard Architecture](./anzen-egress-wireguard.md)*