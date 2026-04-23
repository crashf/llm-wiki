# Anzen Egress — WireGuard Architecture

> How WireGuard tunnels are structured, keyed, and managed across the Anzen Egress platform.

## Production vs Staging Tunnels

Two completely separate WireGuard interfaces on each edge router:

| Property | Production (`wg-anzen`) | Staging (`wg-staging`) |
|----------|----------------------|----------------------|
| Port | 51820 | 51821 |
| Transit network | 10.99.0.0/24 | 10.99.1.0/24 |
| Purpose | Live customer traffic | Pre-stage phone-home, API access |
| Keys | Generated at deployment time | Generated at pre-stage time |
| Routing table | `anzen` | `staging` |

The staging tunnel is the control plane. CPEs phone home to it, and the portal pushes full production configs over it. Once production is live, the staging tunnel remains available as a management fallback.

---

## Key Generation

Keys are generated server-side using Node.js native crypto — no `wg` binary or native dependencies required:

```typescript
import { generateKeyPairSync } from "crypto";

export function generateWireGuardKeyPair(): { publicKey: string; privateKey: string } {
  const { publicKey, privateKey } = generateKeyPairSync("x25519", {
    publicKeyEncoding:  { type: "spki",  format: "der" },
    privateKeyEncoding: { type: "pkcs8", format: "der" },
  });

  // DER-encoded X25519 public key: 12-byte SPKI header + 32-byte raw key
  const rawPublic = publicKey.subarray(publicKey.length - 32);

  // DER-encoded X25519 private key: 16-byte PKCS8 header + 32-byte raw key
  const rawPrivate = privateKey.subarray(privateKey.length - 32);

  return {
    publicKey: rawPublic.toString("base64"),
    privateKey: rawPrivate.toString("base64"),
  };
}
```

**Why this works:** WireGuard private keys are clamped Curve25519 scalars. Node's X25519 implementation (`crypto.generateKeyPairSync` / `crypto.diffieHellman`) handles clamping internally, so the raw 32-byte export is directly compatible with `wg genkey` output and MikroTik's WireGuard implementation.

The DER stripping pattern:
- **Public key:** SPKI DER = 12-byte header + 32 raw bytes → take last 32 bytes
- **Private key:** PKCS8 DER = 16-byte header + 32 raw bytes → take last 32 bytes

Both result in base64 strings identical to what `wg genkey` / `wg pubkey` produce.

> **Lesson learned:** Originally planned to use `tweetnacl` for Curve25519. Turned out Node.js `crypto` handles X25519 natively with proper clamping — no third-party crypto library needed.

---

## Transit Networks

Transit networks are internal /30 pairs used for WG tunnel endpoints:

| Network | Allocation | Edge IP | CPE IP (example) |
|---------|-----------|---------|-------------------|
| 10.99.0.0/24 | Production | .1, .5, .9, ... | .2, .6, .10, ... |
| 10.99.1.0/24 | Staging | .1, .5, .9, ... | .2, .6, .10, ... |

Each /30 provides:
- Network address (unused)
- Edge IP — assigned to the WG interface on the edge router
- CPE IP — assigned to the WG interface on the customer CPE
- Broadcast (unused)

Transit allocations are tracked in the `transit_allocations` table with per-deployment FK.

---

## Customer /30 Handoff

Customers receive routed public IPs — no NAT, full identity:

- **Pool:** 170.205.18.240/28 (16 addresses, 4 usable /30s — extensible via admin GUI)
- **Per-customer /30:** network addr, gateway IP (assigned to customer LAN), customer IP (first usable), broadcast
- **Proxy-ARP:** Edge router does proxy-ARP on `ether1` for all customer public IPs — this is **essential** for the /30 handoff to work. Without it, the upstream provider won't route customer IPs to the edge.

Example:
```
Subnet:   170.205.18.240/30
Gateway:  170.205.18.241  (CPE LAN interface)
Customer: 170.205.18.242  (customer device)
```

The edge router adds:
1. WG peer with customer's public key and `allowed-address=170.205.18.240/30`
2. Route: `170.205.18.240/30 → wg-anzen`
3. Proxy-ARP on `ether1` for the customer IP

---

## Tunnel Lifecycle

### Bootstrap (Staging Keys)

1. Portal generates staging keypair for CPE
2. Bootstrap `.rsc` configures `wg-staging` with staging private key
3. Edge router's staging public key is hardcoded in the bootstrap config
4. Portal adds CPE's staging public key as a peer on `wg-staging` via REST API
5. CPE phones home → appears as **"Online, Unassigned"**

### Deployment (Production Keys)

1. Portal generates production keypair for CPE
2. Portal pushes full production config to CPE over staging tunnel via REST API
3. Production config creates `wg-anzen` with production private key
4. Portal adds CPE's production public key as peer on edge's `wg-anzen`
5. CPE reboots → connects on production tunnel → staging remains available

### Day-2 (Key Rotation)

1. Portal generates new production keypair
2. New keys pushed to CPE (old tunnel still active)
3. Edge peer updated atomically (RouterOS supports key swap on existing peer)
4. Old keys invalidated

---

## MikroTik-Specific WireGuard Config

Key MikroTik settings used in generated configs:

```routeros
# Interface
/interface/wireguard/add name=wg-anzen mtu=1420 listen-port=51821 \
  private-key="<BASE64_PRIVKEY>"

# Peer
/interface/wireguard/peers/add interface=wg-anzen \
  public-key="<EDGE_PUBKEY>" \
  endpoint-address=170.205.18.9 endpoint-port=51820 \
  allowed-address=0.0.0.0/0 \
  persistent-keepalive=25 \
  comment="anzen-edge-170.205.18.9"
```

Critical settings:
- **MTU 1420** — WireGuard adds 60 bytes overhead (20 IPv4 + 20 UDP + 8 WG header + 4 AEAD), so 1420 avoids fragmentation on standard 1500 MTU links
- **persistent-keepalive=25** — Keeps NAT/firewall state alive, ensures tunnel recovery after idle. 25 seconds is aggressive but reliable for CPEs behind carrier NAT
- **allowed-address=0.0.0.0/0** — Full tunnel (all traffic routes through WG). Policy routing decides what actually goes through

### IP Address Pattern

Transit IPs are assigned as /32 on the WG interface with a `network` parameter pointing to the edge:

```routeros
/ip/address/add address=10.99.0.6/32 interface=wg-anzen network=10.99.0.5 \
  comment="Anzen WG transit"
```

This tells RouterOS that 10.99.0.5 (the edge) is directly reachable via this interface without needing a separate route entry for the peer.

### Routing Table Pattern

Production and staging each get their own FIB table:

```routeros
/routing/table/add name=anzen fib
/ip/route/add dst-address=0.0.0.0/0 gateway=10.99.0.5 routing-table=anzen
```

For passthrough mode, a routing rule scopes traffic from the customer subnet:
```routeros
/routing/rule/add src-address=170.205.18.240/30 action=lookup-only-in-table table=anzen
```

For standard router mode, all traffic goes through the tunnel:
```routeros
/routing/rule/add action=lookup-only-in-table table=anzen
```

### WireGuard Health Check Scheduler

Every CPE gets a 10-minute health check that restarts the peer if the tunnel goes dead:

```routeros
/system/scheduler/add name=wg-keepalive interval=10m on-event="...restart peer..." start-time=startup
```

---

## MSS Clamping

Default: **1360 bytes**

```routeros
/ip/firewall/mangle/add chain=forward protocol=tcp tcp-flags=syn \
  in-interface=wg-anzen action=change-mss new-mss=1360 \
  passthrough=yes comment="MSS clamp WG inbound"

/ip/firewall/mangle/add chain=forward protocol=tcp tcp-flags=syn \
  out-interface=wg-anzen action=change-mss new-mss=1360 \
  passthrough=yes comment="MSS clamp WG outbound"
```

Why 1360: WireGuard overhead (60 bytes) + typical PPPOE overhead (8 bytes) + IP/TCP headers (40 bytes) = default 1500 - 60 - 8 = 1432, but 1360 gives comfortable margin for nested tunnels and avoids Path MTU Discovery black holes.

The MSS clamp is applied in both directions (inbound and outbound on the WG interface) and is configurable per deployment in the wizard.

---

*Related: [Portal Overview](./anzen-egress-portal.md) · [Config Generator](./anzen-egress-config-generator.md)*