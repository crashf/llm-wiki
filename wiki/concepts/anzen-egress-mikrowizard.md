# Anzen Egress + MikroWizard Integration

**Anzen Egress** is the WireGuard egress tunnel provisioning portal for Pund-IT. **MikroWizard** (https://github.com/MikroWizard/mikroman) is a full-featured MikroTik management platform.

## Integration Decision (2026-04-19)

After reviewing MikroWizard's features, six operational capabilities were selected for integration into Anzen Egress *without* turning it into a generic NOC platform.

## Features Added

### 1. Device Health Metrics
- CPU %, memory %, uptime, temperature
- Polled every 5 minutes via MikroTik REST API
- Sparkline charts (24h, 7d) per deployment
- **Where:** Deployment Detail → "Health" tab

### 2. Config Backup & Restore
- Automatic backup before firmware upgrades or config changes
- Manual backup anytime
- One-click restore to previous configuration
- **Where:** Deployment Detail → "Backups" tab

### 3. Firmware Management (Safe Updates)
- Firmware repository with auto-download from MikroTik
- "Safe Update" flow: backup → download → upgrade → verify → rollback if needed
- Per-deployment approval (no fleet-wide auto-deploy)
- **Where:** Deployment Detail → "Firmware" tab

### 4. Syslog Viewer
- Collect syslogs from CPEs via UDP 514
- Filter by severity, topic, search
- Live tail mode with 7-day retention
- Auto-configuration on deployment
- **Where:** Deployment Detail → "Logs" tab

### 5. Scheduled Maintenance Tasks
- Per-deployment recurring tasks (not fleet-wide)
- Pre-built: Daily backup, Weekly firmware check
- Simple cron-like scheduling
- Execution history
- **Where:** Deployment Detail → "Tasks" tab

### 6. Diagnostics Terminal
- Safe command execution (whitelisted only)
- Pre-built: ping, traceroute, interface status, route table
- Terminal-style UI with streaming output
- **Where:** Deployment Detail → "Diagnostics" tab

## Features Intentionally Skipped

| Feature | Reason |
|---------|--------|
| NOC Dashboard (all tunnels) | Anzen Egress is for provisioning, not 24/7 monitoring. Use Grafana. |
| Device Groups & Batch Ops | Egress is 1:1 customer:CPE. No need to run commands on 50 routers. |
| RADIUS Server Integration | Not applicable to egress tunnels (no user PPPoE) |
| Wireless Radio Monitoring | Not applicable (wired CPEs) |
| Complex Snippets Library | Diagnostics tab covers 90% of needs |
| Fleet-wide Auto-Upgrade | Too risky for customer equipment. Per-device only. |

## Timeline

**6 weeks** total vs 10 weeks for full MikroWizard clone.

| Phase | Duration | Deliverable |
|-------|----------|-------------|
| Health Metrics | Week 1 | Health tab with CPU/memory charts |
| Syslog | Week 1 | Log viewer with live tail |
| Backups | Week 2 | Backup/restore workflow |
| Firmware | Week 2 | Safe update with rollback |
| Tasks | Week 1 | Scheduled maintenance |
| Diagnostics | Week 1 | Terminal for safe commands |

## Technical Additions

**New Database Tables:**
- `device_health_metrics` — CPU, memory, uptime snapshots
- `device_syslog` — Syslog messages per deployment
- `device_backups` — Config backups
- `firmware_repository` — Cached firmware versions
- `device_firmware_status` — Current vs available per deployment
- `firmware_upgrade_history` — Upgrade attempts
- `scheduled_tasks` — Recurring tasks
- `task_executions` — Task run history

**New Background Workers:**
- `metrics-poller` — Poll health every 5 minutes
- `syslog-server` — UDP 514 collector
- `task-scheduler` — Cron-like task runner

**New UI Tabs on Deployment Detail:**
```
[Overview] [Health] [Logs] [Backups] [Firmware] [Tasks] [Diagnostics] [History]
```

## References

- **Anzen Egress Repo:** https://github.com/crashf/anzen-egress-portal
- **MikroWizard:** https://github.com/MikroWizard/mikroman
- **Integration Plan:** `/projects/anzen-egress-portal/MIKROWIZARD-INTEGRATION-MINIMAL.md`
- **Full Feature Plan:** `/projects/anzen-egress-portal/MIKROWIZARD-FEATURES-PLAN.md`
- **Project TODO:** `/projects/anzen-egress-portal/TODO.md`

## Tags

anzen-egress, mikrowizard, mikrotik, wireguard, portal, device-management, firmware, backups, syslog, diagnostics
