# Veeam Restore Testing Automation

## Summary
Fully automated backup restore testing — zero human interaction. A PowerShell script on the VBR server (`10.255.71.71`) runs on a schedule, recovers each VM's latest backup to a test ESXi host, verifies the VM is healthy, then cleans up. Results go to a JSON file for n8n/Grafana consumption.

## Architecture
- **Runner:** `automated-runner.ps1` — runs on VBR server as Windows Scheduled Task
- **Method:** Instant VM Recovery via VBR REST API (no space needed on VBR)
- **Target:** ESXi `10.255.71.52` → ResourcePool `TestRestores` → Datastore `PundIT-HDD`
- **Health checks:** Boot state → VMware Tools → ICMP ping → IP assigned → disk visible (15 min timeout)
- **Cleanup:** Auto-unmounts Instant Recovery session after checks
- **Results:** `results/results.json` — n8n reads this for Slack alerting on failure

## Key Design Decisions
- **Fully unmanned** — Wayne chose option 1: no human interaction at all
- **Instant VM Recovery** — chosen over full restore for speed and zero disk usage
- **Per-client scheduling** — n8n cron trigger per client → calls runner with `-ClientId`
- **No GUI** — techs never touch VBR console; everything via REST API

## Tech Stack
- PowerShell 5.1+ (runs on VBR server)
- VBR REST API v1 (HTTPS on port 9419)
- VMware PowerCLI (for health check queries)
- n8n (alerting/scheduling layer)

## Related
- [[Veeam Backup Infrastructure]] — VBR server, repos, backup jobs
- [[Automation Runbooks]] — n8n integration patterns
