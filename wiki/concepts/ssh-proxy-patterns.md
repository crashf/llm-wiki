---
title: SSH Proxy Patterns for Fleet Key Management
type: concept
tags: [ssh, fleet-key, proxy, security, openclaw, automation]
sources: 1
created: 2026-04-07
updated: 2026-04-07
---

## Summary

Pattern for centralized SSH access to distributed infrastructure using a single shared fleet key. The PunClaw Dashboard uses this to proxy through to all claws without managing individual SSH credentials per instance.

## Key Points

- **Fleet Key:** Single Ed25519 key (`punclaw_fleet`) authorized in `authorized_keys` on all managed hosts
- **Dashboard Proxy:** Central server (PunClaw Dashboard) uses fleet key to SSH into claws
- **Key Location:** `/home/openclaw/.ssh/punclaw_fleet` on dashboard host
- **Authorized Key:** `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINSngv+IZ+vZbFmunGc0vXmugGQJ0jVM7Tgg3voBvDss punclaw-fleet@pund-it.ca`
- **Per-User SSH:** Dashboard SSHes as root on claws (fleet key in root's `authorized_keys`)
- **Passwordless Access:** No password prompts — key-based auth only
- **Security Implication:** Compromise of fleet key grants access to ALL claws

## Connections

- Related to [[openclaw-fleet-management]] — relies on SSH fleet key for centralized operations
- Related to [[punclaw-dashboard]] — uses fleet key for proxy, terminal, monitoring
- See also [[anzenegress-portal]] — separate security product, different key architecture (WireGuard)

## Details

### The Pattern

```
┌─────────────────────────────────────────────────────────────┐
│                    PunClaw Dashboard                        │
│  10.255.245.162                                            │
│                                                             │
│  /home/openclaw/.ssh/punclaw_fleet  ────────────────────────┼──→ SSH to claws
│  (Ed25519 private key)                                     │      │
│                                                              │      │
└──────────────────────────────────────────────────────────────┘
                              │
                              ▼
        ┌───────────────────────────────────────────────┐
        │            Root Authorized Keys               │
        │  /root/.ssh/authorized_keys                   │
        │                                                │
        │  ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA...     │
        │  punclaw-fleet@pund-it.ca                     │
        └───────────────────────────────────────────────┘
                              │
        ┌──────────┬──────────┬──────────┬──────────┐
        ▼          ▼          ▼          ▼          ▼
      Claw .163  Claw .164  Claw .167  Claw .168  Claw .169
      (ROB)     (CEDRIC)    (Wayne)   (Brett)     (Eve)
```

### Implementation in Dashboard

```typescript
// src/lib/vm-proxy.ts (simplified)
import { execSync } from 'child_process';

export async function sshToClaw(clawIp: string, command: string) {
  const fleetKey = '/home/openclaw/.ssh/punclaw_fleet';
  const result = execSync(command, {
    stdio: 'pipe',
    encoding: 'utf-8',
    ssh: `-i ${fleetKey} root@${clawIp}`
  });
  return result;
}
```

### Key Operations Enabled

1. **Remote File Reading:** `src/lib/remote-fs.ts` reads claw workspace files
2. **System Monitoring:** SSH-based CPU/memory/disk/uptime collection
3. **WebSocket Terminal:** PTY server SSHes in with fleet key
4. **Skills Deployment:** tar + base64 push to `/root/.openclaw/workspace/skills/`
5. **Configuration Updates:** Write channel configs to claws

### Security Considerations

| Concern | Mitigation |
|---------|------------|
| Key compromise | Fleet key only, not user's personal key; isolate to dashboard host |
| Root access | Fleet key authorized for root — necessary for gateway management |
| No audit trail | SSH commands logged at dashboard level, not on claw |
| Revocation | Remove from all claws' authorized_keys to revoke fleet-wide |

### Why This Pattern?

- **Simplicity:** One key to manage vs N keys for N claws
- **Consistency:** Same access model for all claws
- **Centralization:** Dashboard is the only point needing fleet key
- **Automation-friendly:** No interactive prompts in scripts

### Alternative Approaches

- **Per-claw keys:** More granular, harder to rotate
- **Jump host:** Single entry point, adds hop latency
- **VPN + private CA:** More robust but heavier infrastructure