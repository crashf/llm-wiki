# Veeam Restore Testing Automation

## Summary
Fully automated VBR backup restore testing — no human interaction. Runs on VBR server as a scheduled task, uses Veeam Instant VM Recovery via REST API, performs health checks, cleans up automatically, logs results to JSON.

## Key Facts
- **Repo:** https://github.com/crashf/veeam-restore-automation
- **Runs on:** VBR server `10.255.71.71` (punsvnfmsrv)
- **Target:** ESXi `10.255.71.52`, ResourcePool=TestRestores, Datastore=PundIT-HDD
- **REST API:** `https://10.255.71.71:9419`
- **Method:** Instant VM Recovery (no space needed, fastest)
- **Auth:** OAuth2 via VBR REST API

## Architecture
1. PowerShell runner script on VBR server
2. n8n cron trigger per client → invokes runner
3. Runner: pick latest restore point → Instant VM Recovery → health checks → auto-cleanup → log result
4. Health checks: boot state → VMware Tools → ICMP → IP assigned → disk visible (15 min timeout)
5. Results JSON consumed by n8n for alerting and Grafana for dashboards

## Components
- `automated-runner.ps1` — main runner (PowerShell 7+)
- `config/automated-creds.json` — credentials template
- `logs/` — per-run logs
- `results/results.json` — structured results for n8n/Grafana

## Related
- [[sentinel-guardian]] — similar automated testing pattern
- [[openclaw-fleet-management]] — scheduled automation via n8n
